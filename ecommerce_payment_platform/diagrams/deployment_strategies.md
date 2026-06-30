# 배포 전략 비교

> 인풋: [03_comparison_matrix.md](../report/03_comparison_matrix.md), [02_service_selection.md](../report/02_service_selection.md)
> 모든 수치는 02/03 단계에서 이미 검증된 출처 기반. 본 프로젝트(목표) 행은 **아직 측정되지 않은 설계 목표치**임을 명시.

---

## 1. 배포 전략 종류

```
┌──────────────────────────────────────────────────────────────────────┐
│ 배포 전략 정의                                                         │
├──────────────┬────────────────────────────────────────────────────────┤
│ Rolling      │ Pod를 점진적으로 교체. 단순하지만 일시적으로 신구 버전 혼재│
│ Blue/Green   │ 신구 2개 환경 병행 운영. 즉시 전환 가능, 비용 2배          │
│ Canary       │ 신버전을 일부(1~10%) 트래픽으로 먼저 검증 후 점진 확대    │
│ Feature Flag │ 배포와 릴리즈를 분리. 코드 재배포 없이 기능 ON/OFF 전환   │
└──────────────┴────────────────────────────────────────────────────────┘
```

## 2. 6개사 + 본 프로젝트(목표) 배포 전략 비교

| 기업 | 주요 배포 전략 | 자동화 수준 | 비고 |
|---|---|---|---|
| Shopify | Canary(무작위 5%, 10분 검증) | ◎ 자동 | 하루 ~150회 배포, 트렁크 기반 개발 |
| Walmart | 공개 정보 부족 | △ 불명 | Concord 파이프라인 도구만 확인, 전략 비공개 |
| Alibaba | 全链路압测(전체링크 스트레스) + 그림자 테이블 | ◎ 자동(ACK) | 光棍节 대비 검증 특화, 전형적 Canary와는 결이 다름 |
| 11번가 | 심볼릭링크 기반 무중단 배포 + Feature Flag | ○ 부분자동 | 도구체인 명칭 비공개, 기법 자체는 정교함(Spring Batch) |
| 당근마켓 | GitOps Sync(ArgoCD) | ◎ 자동 | 배포시간 5시간→30분 단축(실측) |
| 카카오페이 | **Rolling/Canary/Blue-Green 모두 지원**(Job 선택형) | ◎ 자동(Wallga) | 6개사 중 유일하게 3전략 모두 지원 확인 |
| **본 프로젝트(목표)** | **Canary(Istio 카나리 라우팅 + Argo Rollouts)** | **◎ 자동(목표)** | **PP-1 해소: 메시 도입 전/후 비교 데이터를 Canary 분석 단계에 직접 연동** |

## 3. Shopify Canary 배포 타임라인 (확인된 사실만)

```
시점        상태
──────────────────────────────────────────────────────────
PR Merge    CI(병합 큐) 통과 → Master 반영
   │
   ▼
Canary      무작위 트래픽 5%로 신버전 노출
   │        (구체적 분 단위 메트릭 분석 도구명은 비공개)
   ▼
10분 검증   이상 없으면 다음 단계
   │
   ▼
Production  전체 트래픽 전환

참고: 트렁크 기반 개발로 하루 평균 ~150회 배포 — Shopify Engineering
출처: shopify.engineering/automatic-deployment-at-shopify
※ Netflix Kayenta처럼 분 단위 세부 분석 사이클을 공개한 사례는
  6개사 중 확인되지 않음(Shopify도 "5%, 10분"만 공개) — 추가 단계는 추정하지 않음.
```

## 4. 본 프로젝트(목표) — Argo Rollouts Canary 설계 개념

> [our_target_arch.md](../architectures/our_target_arch.md) 2장(서비스 메시 단계)의 카나리 라우팅을 실제 배포 파이프라인 단계로 구체화한 설계안. 아직 실측되지 않은 목표 설계임.

```yaml
# argo-rollout-config.yaml (개념 예시 — 미실측 설계안)
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5      # PP-1: 메시 적용 전/후 비교용 트래픽 분할
        - pause: {duration: 5m}
        - analysis:         # 자동 메트릭 검증
            templates:
              - templateName: payment-success-rate
        - setWeight: 25
        - pause: {duration: 5m}
        - setWeight: 50
        - pause: {duration: 5m}
        - setWeight: 100

---
# Analysis Template (결제 성공률 검증)
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: payment-success-rate
spec:
  metrics:
    - name: payment-success-rate
      provider:
        prometheus:
          query: |
            sum(rate(payment_success_total[5m])) /
            sum(rate(payment_requests_total[5m]))
      successCondition: result >= 0.999
      failureLimit: 1
```

## 5. Feature Flag로 배포/릴리즈 분리 (11번가 사례 벤치마크)

```
[배포와 릴리즈의 분리]

기존 방식(배포 = 릴리즈):
  코드 변경 → 배포 → 전체 사용자에게 즉시 적용
  문제: 이상 발생 시 전체 사용자가 영향받음

Feature Flag 도입 후(배포 ≠ 릴리즈):
  코드 변경 → 배포(Flag OFF = 아무도 못 봄)
              ↓
              Flag를 단계적으로 ON: 1% → 5% → 10% → 50% → 100%
              ↓
              문제 발생 시 Flag OFF로 즉시 롤백(재배포 불필요)

본 프로젝트 적용 시나리오(PP-2 연계):
  - 새 PG(결제대행사) 연동: Flag OFF로 배포 후 1% 사용자로 동작 확인 → 단계 확대
  - Saga 보상 트랜잭션 로직 변경: Flag로 신규/기존 로직 A/B 비교 후 전환
  - 긴급 Kill Switch: 장애 시 문제 기능 즉시 OFF

참고: 11번가는 Feature Flag(OpenFeature)를 무중단 배포와 결합해 운영 — 11st-tech.github.io/2023/11/07/openfeature/
```
