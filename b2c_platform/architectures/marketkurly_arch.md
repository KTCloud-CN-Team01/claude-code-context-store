# Market Kurly (마켓컬리) 推定アーキテクチャ（2024 年公開情報ベース）

> 참조일: 2026-06-30  
> 주의: 공개 기술 블로그（helloworld.kurly.com）기반 추정.

---

## 1. 전체 아키텍처 개요

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                       Market Kurly 추정 아키텍처                               │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  [클라이언트 층]                                                               │
│  iOS / Android / Web（React/TypeScript）                                     │
│         │                                                                    │
│         ▼                                                                    │
│  [CDN 층]                                                                    │
│  AWS CloudFront                                                              │
│  └── 상품 이미지（S3 원본 → CloudFront 캐시）                                    │
│         │                                                                    │
│         ▼                                                                    │
│  [API 층]                                                                    │
│  AWS ALB → Kubernetes Ingress（Nginx）                                        │
│         │                                                                    │
│         ▼                                                                    │
│  [마이크로서비스 층（DDD 기반 분리）]                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ 주문     │ │ 결제     │ │ 상품     │ │ 회원     │ │ 배송     │          │
│  │ Service │ │ Service │ │ Service │ │ Service │ │ Service │          │
│  │(Kotlin) │ │(Kotlin) │ │(Kotlin) │ │(Kotlin) │ │(Kotlin) │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│  ┌──────────┐ ┌──────────┐                                                   │
│  │ 검색     │ │ 정산     │                                                   │
│  │ Service │ │ Service │                                                   │
│  │(ES 기반) │ │(Kotlin) │                                                   │
│  └──────────┘ └──────────┘                                                   │
│         │                                                                    │
│         ▼                                                                    │
│  [이벤트 층]                                                                   │
│  Apache Kafka（주문/재고/배송 이벤트）                                           │
│         │                                                                    │
│         ▼                                                                    │
│  [데이터 층]                                                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                        │
│  │ Aurora   │ │  Redis   │ │ Elastic- │ │    S3    │                        │
│  │ MySQL    │ │（캐시）   │ │  search  │ │（이미지/  │                        │
│  │          │ │          │ │（검색）   │ │ 배송정보）│                        │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                        │
│                                                                              │
│  [컨테이너 기반: AWS EKS]                                                       │
│  EKS Managed Node Group + HPA                                                │
│                                                                              │
│  [CI/CD]                                                                     │
│  GitHub Actions → Docker Build → ECR → ArgoCD → EKS                         │
│  참조: https://helloworld.kurly.com/blog/kurly-devops-transformation/        │
│                                                                              │
│  [관측 기반]                                                                   │
│  Datadog（Metrics/APM）+ CloudWatch（Logs）                                    │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 새벽배송（早朝配送）スパイク対応アーキテクチャ

```
[새벽배송 주문 마감 스파이크 패턴]

시간대별 트래픽 패턴:
  23:00〜24:00: 주문 마감 전 급등（통상 트래픽의 5-10배）
  06:00〜08:00: 배송 시작（배송 추적 API 급등）
  그 외 시간: 저트래픽

스파이크 대응 구조:
  ┌─────────────────────────────────────────────┐
  │                                             │
  │  주문 마감 1시간 전（22:00）:               │
  │  Kubernetes HPA가 주문 서비스 파드 3 → 20개 │
  │  사전 스케일아웃（Scheduled Scaling）        │
  │                                             │
  │  주문 폭주 시（23:30〜24:00）:              │
  │  ① 주문 서비스: 큐（SQS）로 요청 버퍼링     │
  │  ② 결제 서비스: 비동기 처리                 │
  │  ③ 재고 서비스: Redis 원자적 감소（DECR）   │
  │                                             │
  │  새벽 배송 시작（06:00）:                   │
  │  배송 추적 WebSocket 서버 자동 스케일링      │
  └─────────────────────────────────────────────┘

참조:
  https://helloworld.kurly.com/blog/database-failover/
  https://helloworld.kurly.com/blog/kurly-event-driven-architecture/
```

---

## 3. 결제 아키텍처

```
[Market Kurly 결제 플로우（추정）]

클라이언트
    │
    ▼
주문 서비스
    │
    ├──▶ 재고 사전 확인（Redis DECR 원자적 처리）
    │
    ▼
결제 서비스
    ├── KakaoPay（메인）
    │     └── Circuit Breaker
    ├── NaverPay（대체 1）
    ├── 토스페이먼츠（대체 2）
    └── 카드사 직접 연동（대체 3）

결제 완료 이벤트（Kafka）
    ├──▶ 주문 서비스（주문 확정）
    ├──▶ 배송 서비스（배송 준비）
    └──▶ 정산 서비스（회계 처리）

정기구독（컬리패스）별도 플로우:
  매달 자동 결제 배치（Spring Batch）
  └── 실패 시 재시도 로직（exponential backoff）
  └── 실패 회원 알림 발송

결제 취소/환불（복잡한 보상 트랜잭션）:
  새벽배송 특성상 출고 전/후 환불 정책이 다름
  → 출고 전: 즉시 결제 취소（PG 취소 API 호출）
  → 출고 후: 회수 후 환불（배송 서비스와 협조）
  참조: https://helloworld.kurly.com/blog/payment-service-msa/
```

---

## 4. Event Storming 기반 DDD 설계

```
[Market Kurly의 Event Storming 적용（공개 자료 기반）]

주요 도메인 이벤트:
  주문 도메인:   OrderCreated / OrderConfirmed / OrderCancelled
  결제 도메인:   PaymentRequested / PaymentCompleted / PaymentFailed / RefundCompleted
  배송 도메인:   DeliveryScheduled / OutForDelivery / DeliveryCompleted
  재고 도메인:   StockReserved / StockReleased / StockDepleted

Bounded Context 분리:
  ┌─────────────────────────────────────────────┐
  │ 주문 BC ──이벤트──▶ 결제 BC                  │
  │           ──이벤트──▶ 배송 BC                │
  │           ──이벤트──▶ 재고 BC                │
  └─────────────────────────────────────────────┘

참조:
  https://helloworld.kurly.com/blog/introducing-event-storming/
```

---

## 5. 기술적 강점・약점 요약

| 항목 | 강점 | 약점 |
|-----|------|------|
| 배송 | 새벽배송 전용 스파이크 설계 | — |
| DDD | Event Storming 문화 정착 | — |
| Kotlin 전환 | 백엔드 현대화 | — |
| 서비스 메시 | — | Istio 미도입（추정） |
| 카오스 엔지니어링 | — | 공개 사례 없음 |
| 관측성 | — | Traces 미구현 가능성 높음 |

**참조 링크**:
- https://helloworld.kurly.com/
- https://helloworld.kurly.com/blog/kurly-devops-transformation/
- https://helloworld.kurly.com/blog/introducing-event-storming/
