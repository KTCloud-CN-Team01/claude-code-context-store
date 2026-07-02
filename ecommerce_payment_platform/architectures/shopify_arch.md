# Shopify 추정 아키텍처

> 참조일: 2026-06-30 | 출처: 공개 기술 블로그(shopify.engineering) 기반 추정. 실제 구성과 다를 수 있음.
> 상세 근거: [02_service_selection.md](../report/02_service_selection.md) #1

---

## 1. 전체 아키텍처 개요

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         Shopify 아키텍처 전체상                            │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. 초기 구조                                                            │
│  [클라이언트 계층] 스토어프론트(머천트 샵) / 체크아웃 / Admin                │
│         │                                                                │
│         ▼                                                                │
│  [모듈러 모놀리스 — Ruby on Rails + Packwerk]                              │
│  ┌────────────────────────────────────────────────────────────┐          │
│  │  Packwerk로 모듈 경계 강제 — "합리적 이유가 있을 때만"          │          │
│  │  분리(스토어프론트 렌더링·신용카드 보관 등 일부만 마이크로서비스)│          │
│  └────────────────────────────────────────────────────────────┘          │
│                                                                          │
│  2. 도메인 분리 전략 — MSA 대신 Pod 샤딩                                  │
│  [Pod 아키텍처 — shop_id 샤딩]                                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                    │
│  │ Pod 1    │ │ Pod 2    │ │ Pod 3    │ │ Pod N    │  (약 400개          │
│  │(DB 격리) │ │(DB 격리) │ │(DB 격리) │ │(DB 격리) │   K8s 클러스터)     │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                    │
│   테넌트 격리 + blast radius 제한 (도메인 세분화 대신 이 전략을 채택)       │
│                                                                          │
│  3. 플랫폼 운영 체계                                                     │
│  A) CD 계층                                                              │
│  PR → CI(병합큐) → Master → Canary(무작위 5%, 10분검증) → Production      │
│  하루 평균 ~150회 배포(트렁크 기반 개발)                                  │
│                                                                          │
│  B) 관찰성 — 벤더 종속 탈피 OSS 스택                                      │
│  Grafana(대시보드) + Cortex(메트릭) + Loki(로그) + Tempo(트레이싱)         │
│  비즈니스 지표(체크아웃 이탈률) ↔ 레이턴시 연계 대시보드                    │
│                                                                          │
│  C) 복원력                                                               │
│  Semian(서킷브레이커+bulkhead, SysV세마포어) + Toxiproxy(장애주입)         │
│  Leaky Bucket 쓰로틀링(REST 40/초, Plus 400/초)                          │
│                                                                          │
│  [서비스 메시 — 평가 후 보류]                                             │
│  Istio Resiliency Simulator 자체 개발해 평가했으나                       │
│  Envoy 사이드카 CPU 비용(1M RPS당 월 $5만+) 정량화 후 전사 도입은 보류    │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```
<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/928224cd-da8f-47b6-bb76-fe344f7cd4b1" />


## 2. 5축 요약

| 축 | 내용 |
|---|---|
| 아키텍처 | 모듈러 모놀리스 + Pod 샤딩(MSA 최소화 전략) |
| CI/CD | Canary(5%, 10분) 자동배포, 하루 ~150회 |
| 관찰성 | Grafana+Cortex+Loki+Tempo (OSS 풀스택) |
| 복원력 | Semian(CB+bulkhead) + Toxiproxy + Leaky Bucket |
| 서비스 메시 | 평가 후 비용 문제로 보류 |

## 3. 출처

- [Shopify Engineering - Pods Architecture](https://shopify.engineering/a-pods-architecture-to-allow-shopify-to-scale)
- [Deconstructing the Monolith](https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity)
- [Automatic Deployment at Shopify](https://shopify.engineering/automatic-deployment-at-shopify)
- [Shopify's Journey to Planet-Scale Observability](https://horovits.medium.com/shopifys-journey-to-planet-scale-observability-9c0b299a04dd)
- [GitHub - Shopify/semian](https://github.com/Shopify/semian)
- [Benchmarking Istio & Linkerd CPU](https://medium.com/@michael_87395/benchmarking-istio-linkerd-cpu-c36287e32781) — ⚠️ 블로그 출처(컨퍼런스 아님, [08_pain_point_validation.md](../report/08_pain_point_validation.md) 참조)
