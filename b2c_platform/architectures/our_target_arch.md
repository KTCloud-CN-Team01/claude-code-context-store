# 本プロジェクト 目標アーキテクチャ

> 참조일: 2026-06-30  
> 既存サービスのペインポイント（PP-01〜PP-07）を解決する設計

---

## 1. 全体アーキテクチャ

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                 本プロジェクト 目標クラウドネイティブアーキテクチャ                  │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  [クライアント層]                                                              │
│  Web（React）/ Mobile（iOS/Android）                                          │
│         │                                                                    │
│         ▼                                                                    │
│  [CDN / Edge 層]                                                              │
│  AWS CloudFront + Lambda@Edge（認証・地域リダイレクト）                          │
│  └── CloudFront Signed URL（有料コンテンツ保護: PP-07 解決）                    │
│         │                                                                    │
│         ▼                                                                    │
│  [Ingress / API Gateway 層]                                                   │
│  ┌─────────────────────────────────────────────────────┐                    │
│  │ AWS ALB → Kubernetes Ingress（NGINX）               │                    │
│  │ + Kong Gateway（Rate Limiting / Auth）              │                    │
│  └─────────────────────────────────────────────────────┘                    │
│         │                                                                    │
│         ▼                                                                    │
│  [Service Mesh 層: Istio + Envoy]（PP-05 解決）                               │
│  ┌─────────────────────────────────────────────────────┐                    │
│  │ Istio Control Plane                                 │                    │
│  │ └── Envoy Sidecar（全 Pod 注入）                     │                    │
│  │     ├── mTLS 自動化（PCI DSS 準拠）                  │                    │
│  │     ├── Circuit Breaker（サービス間）                │                    │
│  │     ├── Retry / Timeout 設定                        │                    │
│  │     └── トラフィック観測（Metrics→Prometheus）        │                    │
│  └─────────────────────────────────────────────────────┘                    │
│         │                                                                    │
│         ▼                                                                    │
│  [マイクロサービス層（Kotlin / Spring Boot）]                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ 주문     │ │ 결제     │ │ 상품     │ │ 회원     │ │ 콘텐츠   │          │
│  │ Service │ │ Service │ │ Service │ │ Service │ │ Service │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│         │                                                                    │
│         ▼                                                                    │
│  [Saga Orchestrator 층]（PP-01, PP-02 解決）                                  │
│  ┌─────────────────────────────────────────────────────┐                    │
│  │ Temporal（Workflow Engine）                         │                    │
│  │ ├── 주문→결제→재고 Saga 관리                         │                    │
│  │ ├── 보상 트랜잭션 자동 실행                           │                    │
│  │ └── Outbox Pattern（Debezium CDC→Kafka）            │                    │
│  └─────────────────────────────────────────────────────┘                    │
│         │                                                                    │
│         ▼                                                                    │
│  [이벤트 스트리밍 층]                                                           │
│  Apache Kafka（MSK）+ Schema Registry（Confluent）                            │
│  └── KEDA（Kafka 기반 Pod 자동 스케일링: PP-01 解決）                           │
│         │                                                                    │
│         ▼                                                                    │
│  [데이터 층（Service per DB）]（PP-02 解決）                                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ Aurora   │ │DynamoDB  │ │  Redis   │ │ Elastic- │ │    S3    │          │
│  │（주문/결제）│ │（카탈로그）│ │（세션/캐시）│ │  search  │ │（콘텐츠） │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│                                                                              │
│  [컨테이너 기반: AWS EKS]                                                       │
│  EKS + Karpenter（비용 최적화）+ HPA + KEDA                                    │
│                                                                              │
│  [CI/CD 기반]（PP-04 解決）                                                    │
│  GitHub Actions → ECR → ArgoCD → Argo Rollouts（Canary）                    │
│  + Unleash Feature Flag                                                      │
│                                                                              │
│  [관측성 기반: LGTM 스택]（PP-03 解決）                                          │
│  OpenTelemetry Collector → Loki / Grafana / Tempo / Prometheus               │
│  + AlertManager（SLO 기반 알림）                                               │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 결제 플로우 상세（Saga Pattern）

```
[결제 Saga 플로우（Temporal 기반）]

클라이언트 → 주문 서비스
                │
                ▼
         Temporal Workflow 시작
                │
                ├── Activity 1: 주문 생성（주문 DB 저장）
                │
                ├── Activity 2: 재고 예약（재고 서비스 API）
                │   실패 → 보상: 주문 취소
                │
                ├── Activity 3: 결제 요청
                │   ├── PG Adapter → 토스페이먼츠（1순위）
                │   │   Circuit Breaker: OPEN → KakaoPay 전환（PP-06 解決）
                │   ├── 성공: 결제 완료 이벤트 발행
                │   └── 실패 → 보상: 재고 예약 해제 + 주문 취소
                │
                ├── Activity 4: 수강/배송 권한 부여（콘텐츠 서비스）
                │   실패 → 보상: 결제 취소（PG 환불 API）
                │
                └── Workflow 완료: 성공 이벤트 발행

멱등성 보장:
  - 각 Activity에 Idempotency-Key（OrderID_StepID）
  - Redis에 24h TTL로 중복 실행 방지
```

---

## 3. Progressive Delivery パイプライン

```
[배포 파이프라인: Argo Rollouts Canary]

PR Merge to main
    │
    ▼
GitHub Actions（빌드 + 단위테스트）
    │
    ▼
Docker Build → ECR Push
    │
    ▼
ArgoCD（GitOps Sync）
    │
    ▼
Argo Rollouts（Canary Strategy）
    ├── Step 1: Canary 5%（5분 대기）
    │     └── Prometheus Analysis: 결제 에러율 < 0.1%?
    │     └── 실패 시 → 자동 롤백
    ├── Step 2: 25%（5분 대기）
    ├── Step 3: 50%（5분 대기）
    └── Step 4: 100%（Full Rollout）

SLO 메트릭 기준（Canary 실패 판단）:
  - 결제 에러율 > 0.1% → 롤백
  - P99 응답시간 > 3초 → 롤백
  - HTTP 5xx 비율 > 0.5% → 롤백
```

---

## 4. オブザーバビリティ設計

```
[3 기둥 + 결제 특화 관측]

Metrics（Prometheus）:
  - 결제 성공률（PG별 분리）
  - 결제 P50/P95/P99 응답시간
  - Saga Workflow 완료율/실패율
  - Circuit Breaker 상태（OPEN/CLOSED）

Logs（Loki）:
  - 구조화 JSON 로그（모든 서비스）
  - TraceID 포함（Traces와 연동）
  - 결제 이벤트 감사 로그（별도 보존）

Traces（Tempo）:
  - OpenTelemetry Auto-Instrumentation
  - 전체 주문-결제 플로우 트레이싱
  - Slow Query 자동 태그

Dashboard（Grafana）:
  - 결제 SLO 대시보드（에러 버짓 소비율）
  - Saga 워크플로우 진행 상황
  - PG별 성공률 비교
```

---

## 5. ペインポイント解決マッピング

```
┌──────────┬────────────────────────────────────────────┬──────────────┐
│ PP ID   │ 해결 기술                                   │ 목표 지표     │
├──────────┼────────────────────────────────────────────┼──────────────┤
│ PP-01   │ KEDA + Saga + Circuit Breaker               │ 결제 성공률   │
│ 결제 스파이크│                                           │ ≥ 99.99%     │
├──────────┼────────────────────────────────────────────┼──────────────┤
│ PP-02   │ Temporal Saga + Outbox + Debezium CDC       │ 데이터 불일치 │
│ 데이터 정합성│                                          │ ≈ 0건/일      │
├──────────┼────────────────────────────────────────────┼──────────────┤
│ PP-03   │ OpenTelemetry + LGTM + SLO Alert            │ MTTD < 1분   │
│ 감지 지연 │                                            │ MTTR < 15분  │
├──────────┼────────────────────────────────────────────┼──────────────┤
│ PP-04   │ Argo Rollouts Canary + PDB + Readiness Probe│ 배포 중 5xx  │
│ 배포 다운타임│                                          │ = 0%          │
├──────────┼────────────────────────────────────────────┼──────────────┤
│ PP-05   │ Istio + Envoy Circuit Breaker               │ 장애 전파     │
│ 장애 전파 │                                            │ 차단 99%      │
├──────────┼────────────────────────────────────────────┼──────────────┤
│ PP-06   │ PG Adapter + Circuit Breaker + Failover     │ PG 장애 시   │
│ PG 단일장애점│                                          │ 전환 < 1초    │
├──────────┼────────────────────────────────────────────┼──────────────┤
│ PP-07   │ CloudFront Signed URL + IaC(Terraform)      │ 무단 접근     │
│ CDN 설정 오류│                                          │ 0건           │
└──────────┴────────────────────────────────────────────┴──────────────┘
```
