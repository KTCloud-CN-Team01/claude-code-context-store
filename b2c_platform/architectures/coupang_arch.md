# Coupang (쿠팡) 推定アーキテクチャ（2024 年公開情報ベース）

> 参照日: 2026-06-30  
> 주의: 공개 기술 블로그 및 컨퍼런스 자료 기반 추정. 실제 구성과 다를 수 있음.

---

## 1. 전체 아키텍처 개요

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         Coupang 추정 아키텍처 전체                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  [클라이언트 층]                                                               │
│  iOS App / Android App / Web (React)                                         │
│         │                                                                    │
│         ▼                                                                    │
│  [CDN / Edge 층]                                                              │
│  CloudFront + Akamai                                                         │
│  └── Static Assets（이미지/JS/CSS）캐시                                        │
│         │                                                                    │
│         ▼                                                                    │
│  [API Gateway / BFF 층]                                                       │
│  ┌─────────────────────────────────────────┐                                 │
│  │ Internal Load Balancer（ALB）            │                                 │
│  │ + Spring Cloud Gateway（추정）           │                                 │
│  └─────────────────────────────────────────┘                                 │
│         │                                                                    │
│         ▼                                                                    │
│  [마이크로서비스 층（도메인별 분리）]                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ 주문     │ │ 결제     │ │ 상품     │ │ 회원     │ │ 배송     │          │
│  │ Service │ │ Service │ │ Service │ │ Service │ │ Service │          │
│  │(Kotlin) │ │(Kotlin) │ │(Java)   │ │(Java)   │ │(Java)   │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                                     │
│  │ 검색     │ │ 추천     │ │ 쿠팡페이  │                                     │
│  │ Service │ │ Service │ │ Service │                                     │
│  │(ES 기반) │ │(Python  │ │(자체 PG) │                                     │
│  │         │ │ ML)     │ │         │                                     │
│  └──────────┘ └──────────┘ └──────────┘                                     │
│         │                                                                    │
│         ▼                                                                    │
│  [이벤트 스트리밍 층]                                                           │
│  ┌─────────────────────────────────────────┐                                 │
│  │ Apache Kafka（주문/재고/배송 이벤트）        │                                 │
│  └─────────────────────────────────────────┘                                 │
│         │                                                                    │
│         ▼                                                                    │
│  [데이터 층]                                                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ Aurora   │ │DynamoDB  │ │  Redis   │ │ Elastic- │ │    S3    │          │
│  │ MySQL    │ │（카탈로그）│ │（세션/캐시）│ │  search  │ │（이미지/  │          │
│  │（주문/결제）│ │          │ │          │ │（검색）   │ │ 로그）   │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│                                                                              │
│  [컨테이너 기반]                                                               │
│  AWS EKS（Kubernetes on AWS）                                                 │
│  └── Karpenter（추정: 노드 자동 스케일링）                                       │
│  └── KEDA（Kafka 기반 Pod 자동 스케일링）                                        │
│                                                                              │
│  [CI/CD 기반]                                                                 │
│  GitHub → Jenkins（Build）→ ArgoCD（GitOps Deploy）→ EKS                     │
│  参照: https://medium.com/coupang-engineering/our-journey-to-continuous-delivery-at-coupang-105634185e28
│                                                                              │
│  [관측 기반]                                                                   │
│  Datadog（Metrics/APM）+ ELK Stack（Logs）                                    │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Rocket Delivery（로켓배송）물류 시스템 아키텍처

```
[로켓배송 디지털 파이프라인]

주문 완료（결제 성공）
    │
    ▼ Kafka 이벤트: OrderPaid
배송 오케스트레이터
    ├── 재고 확인（재고 서비스）
    ├── 풀필먼트 센터 할당（물류 서비스）
    └── 배송 기사 배정（TMS: Transportation Management System）
           │
           ▼
    실시간 위치 추적（GPS → IoT → Kinesis Data Streams → WebSocket）
           │
           ▼
    고객 앱（배달 현황 실시간 업데이트）

물류센터 자동화:
  - 컨베이어 벨트 + 로봇 ARM 제어: IoT Gateway → 내부 API
  - 온도관리 상품（신선식품）: 냉장/냉동 구역 실시간 모니터링
  - 분류기 제어: Kafka 이벤트 기반 실시간 라우팅

참조:
  https://medium.com/coupang-engineering/event-driven-architecture-at-coupang-3e8b2f6a6f07
  https://aws.amazon.com/solutions/case-studies/coupang/
```

---

## 3. 결제 아키텍처（쿠팡페이 통합）

```
[쿠팡 결제 플로우（추정）]

클라이언트
    │
    ▼
주문 서비스
    │ Saga Orchestrator（추정: Temporal or 자체 구현）
    ├──▶ 재고 차감（재고 서비스）
    ├──▶ 결제 요청（결제 서비스）
    │       ├── 쿠팡페이（자체 PG: 1순위）
    │       │     └── Circuit Breaker（Resilience4j 추정）
    │       ├── KakaoPay（대체 1）
    │       ├── NaverPay（대체 2）
    │       └── 카드사 직접 연동（대체 3）
    │
    ▼ 결제 성공 이벤트（Kafka: PaymentCompleted）
배송 서비스 → 로켓배송 파이프라인

결제 데이터:
  Aurora MySQL（주문/결제 기록）+ Redis（멱등성 키 / 세션）

쿠팡페이 분리 이점:
  - 자체 PG로 수수료 내재화
  - 결제 데이터 완전 통제（사기탐지 AI 자체 운용）
  - 간편결제 UX 최적화（원클릭 결제）
  참조: https://www.coupangpay.com/
```

---

## 4. MSA 전환 여정

```
[모놀리식 → MSA 전환 타임라인（공개 정보 기반）]

2010〜2015: 모놀리식 서비스（Java Monolith）
    │
    ▼
2016〜2018: 도메인별 서비스 분리 시작
            주문 / 상품 / 회원 / 결제 분리
    │
    ▼
2019〜2021: Kubernetes 도입（ECS → EKS 마이그레이션）
            Kafka 이벤트 드리븐 아키텍처 전환
            참조: https://medium.com/coupang-engineering/our-journey-to-continuous-delivery-at-coupang-105634185e28
    │
    ▼
2022〜現在: 세분화된 MSA + GitOps 기반 CD
            Datadog 전사 관측성 플랫폼
            쿠팡페이 별도 플랫폼으로 분리

전환 과정에서의 주요 과제:
  - 분산 트랜잭션（주문/결제 정합성 유지）
  - 서비스 간 API 버전 관리（하위 호환성）
  - 데이터 마이그레이션（모놀리식 DB 분리）
```

---

## 5. 기술적 강점・약점 요약

| 항목 | 강점 | 약점 |
|-----|------|------|
| 배송 | 로켓배송 디지털화 수준 최고 | — |
| 결제 | 자체 PG로 의존도 최소화 | — |
| MSA 전환 | 공개 사례 풍부（중견 기업 롤모델） | — |
| 서비스 메시 | — | Istio 전사 적용 여부 불분명 |
| 카오스 엔지니어링 | — | 공개 정보 없음（미실시 가능성）|

**참조 링크**:
- https://medium.com/coupang-engineering
- https://aws.amazon.com/solutions/case-studies/coupang/
- https://ir.coupang.com/
