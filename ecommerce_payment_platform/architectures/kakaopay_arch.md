# 카카오페이 추정 아키텍처

> 참조일: 2026-06-30 | 출처: tech.kakaopay.com(공식 기술 블로그) + 카카오 FY2024 Annual Report 기반 추정.
> 상세 근거: [02_service_selection.md](../report/02_service_selection.md) #6

---

## 1. 전체 아키텍처 개요

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        카카오페이 아키텍처 전체상                          │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. 초기 구조                                                            │
│  [인프라 — 3중 하이브리드, 99.999% 가용성 지향]                            │
│  ┌──────────┐ ┌──────────────┐ ┌──────────────────┐                     │
│  │ AWS EKS  │ │ IDC          │ │ Kakao Cloud(KC)   │                     │
│  │          │ │ Kubernetes   │ │ Kubernetes        │                     │
│  └──────────┘ └──────────────┘ └──────────────────┘                     │
│         GSLB + NodeLocal DNSCache로 트래픽 관리(메시 대체재로 추정)        │
│                                                                          │
│         ▼                                                                │
│  2. 도메인 분리 및 서비스 구조 — 결제 도메인 특화                          │
│  [MSA — 분산 트랜잭션 처리]                                               │
│  ┌────────────────────────────────────────────────────────────┐          │
│  │ 3상태 모델(성공/실패/Unknown) + tx_key 멱등성                  │          │
│  │ + Saga적 보상 트랜잭션(결제 취소)                              │          │
│  │ Kotlin Result 패턴 기반 HTTP 클라이언트                        │          │
│  └────────────────────────────────────────────────────────────┘          │
│  카카오페이증권: 99.993%(2024 실적)·99.99%(2025 목표) SLO **명시 공시**     │
│  (6개사 중 유일한 국내 SLO 공시 사례)                                      │
│                                                                          │
│  3. 플랫폼 운영 체계                                                     │
│  A) CD 계층 — 자체 플랫폼 "Wallga"                                       │
│  Jenkins + ArgoCD 기반, Rolling/Canary/Blue-Green **3가지 모두 지원**      │
│  (개발자가 Job에서 선택) — "You build it, you run it" 철학                │
│  Feature Flag: Redis Pub/Sub + 로컬 캐시 실시간 동기화                    │
│                                                                          │
│  B) 관찰성 — Pallas v2 로그 플랫폼(2026.02, "호그와트 도서관 프로젝트")     │
│  Filebeat→OpenTelemetry / Fluentd→OTel Collector                        │
│  OpenSearch→ClickHouse(ClickStack) / 조회 HyperDX / 모니터링 Grafana      │
│  하루 41TB·200억 건, 지연 수시간→20초 이내, 비용 85.6%↓, 처리량 26배↑      │
│                                                                          │
│  C) 복원력 — 애플리케이션 레벨 직접 구현                                   │
│  네트워크 예외 3상태 구분 + 재시도/보상 트랜잭션/수기 처리                  │
│  PG사 다중화 자동 전환 / 명시적 CB(Resilience4j 등) 도입 여부 비공개       │
│  순수 "카오스 엔지니어링" 공개 자료 없음                                   │
│  (인접: 카카오페이증권 명의 DR훈련 2025-11-23, 본체 단독 사례 불명확)       │
│                                                                          │
│  [서비스 메시 — 미도입 추정]                                              │
│  Istio/Envoy/Linkerd 명시적 언급 전무 — 복원력을 사이드카가 아닌            │
│  애플리케이션 코드 레벨에서 직접 구현(간접 정황)                           │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```
<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/d8941059-dea4-4439-bbda-8b8ad1e836a6" />


## 2. 5축 요약

| 축 | 내용 |
|---|---|
| 아키텍처 | MSA + 결제 특화 분산 트랜잭션(3상태 모델+tx_key+Saga) |
| CI/CD | Wallga(Jenkins+ArgoCD), Rolling/Canary/Blue-Green 모두 지원 |
| 관찰성 | Pallas v2(OTel+ClickHouse+HyperDX+Grafana), 26배 처리량 개선 |
| 복원력 | 애플리케이션 레벨 직접 구현, 순수 카오스 엔지니어링 미확인 |
| 서비스 메시 | 미도입 추정(GSLB/DNSCache로 대체) |

## 3. 출처

- [MSA 환경 네트워크 예외 처리](https://tech.kakaopay.com/post/msa-transaction/)
- [99.999%를 향한 집착 - 멀티/하이브리드 클러스터](https://tech.kakaopay.com/post/multi-cluster/)
- [DevOps문화와 Platform Engineering](https://tech.kakaopay.com/post/kakaopaysec-devops-platform/)
- [피처 플래그 개발기](https://tech.kakaopay.com/post/feature-flag/)
- [일 41TB 로그 ClickStack 처리 - Pallas v2](https://tech.kakaopay.com/post/pallas-v2-log-platform/)
- [카카오페이증권 IT 재난훈련](https://www.fnnews.com/news/202511251818005108)
- [카카오 FY2024 Annual Report](https://t1.kakaocdn.net/pay_brand_admin/file/wyyRvaJE4aDiSlsfYVUGk/FY2024AnnualReport.pdf)
