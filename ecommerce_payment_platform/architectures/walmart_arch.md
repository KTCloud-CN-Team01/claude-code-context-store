# Walmart 추정 아키텍처

> 참조일: 2026-06-30 | 출처: 공개 기술 블로그(tech.walmart.com)·뉴스·Chaos Conf 발표 기반 추정.
> 상세 근거: [02_service_selection.md](../report/02_service_selection.md) #2

---

## 1. 전체 아키텍처 개요

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         Walmart 아키텍처 전체상                            │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. 초기 구조                                                            │
│  [마이크로서비스 계층 — Node.js(hapi 프레임워크), 2012년 모놀리스 재설계]    │
│                                                                          │
│         ▼                                                                │
│  2. 도메인 분리 및 플랫폼 확장 구조                                       │
│  [멀티클라우드 "Triplet Model" — 벤더 록인 회피 설계]                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │ Azure    │ │ GCP      │ │ 프라이빗 DC   │ │ 엣지(매장/물류)│            │
│  │          │ │(AWS 의도적│ │              │ │ 약 1만 노드   │            │
│  │          │ │ 배제)    │ │              │ │              │            │
│  └──────────┘ └──────────┘ └──────────────┘ └──────────────┘            │
│         비용 10~18% 절감 (퍼블릭 탄력성 ↔ 프라이빗 폴백 분산)               │
│         │                                                                │
│         ▼                                                                │
│  [WCNP — Walmart Cloud Native Platform, Kubernetes 기반]                  │
│  ┌────────────────────────────────────────────────────────────┐          │
│  │ 엣지~퍼블릭클라우드 전체에 서비스 메시·관찰성 통합 제공         │          │
│  │ (Istio + Linkerd 상황별 병행 — WCNP VP 공식 확인)              │          │
│  └────────────────────────────────────────────────────────────┘          │
│         │                                                                │
│         ▼                                                                │
│  [이벤트 계층 — Kafka, 약 8,500 노드]                                     │
│  ┌────────────────────────────────────────────────────────────┐          │
│  │ 실시간 재고관리: 하루 5억 이벤트                                │          │
│  │ 보충 시스템: 1억 SKU, 분당 85GB                                │          │
│  │ 전체: 하루 110억 건 이벤트 처리                                 │          │
│  └────────────────────────────────────────────────────────────┘          │
│                                                                          │
│  3. 플랫폼 운영 체계                                                     │
│  A) CD 계층                                                              │
│  OneOps(과거, OSS화)→ 현재 Concord(다양한 배포 타깃 지원)                 │
│  파이프라인에 Jaeger/Zipkin/OTel 트레이싱 통합 권장                       │
│                                                                          │
│  B) 관찰성 — 자체 OSS 네트워킹/관찰성                                     │
│  SONiC(네트워크 OS) + L3AF(eBPF 오케스트레이션)                          │
│                                                                          │
│  C) 복원력 — 카오스/리질리언스 엔지니어링 성숙도 체계                       │
│  Concord + Gremlin 연동 자동화, 팀별 레벨1~3 인증(50개+ 팀 레벨2 통과)     │
│  자체 진단 도구 "Resiliency Doctor"                                      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/ec1abb51-51e0-4831-b18e-18cb809c7895" />


## 2. 5축 요약

| 축 | 내용 |
|---|---|
| 아키텍처 | MSA(Node.js/hapi) + 멀티클라우드 Triplet Model |
| CI/CD | OneOps→Concord, 트레이싱 통합 권장(빈도 수치는 비공개) |
| 관찰성 | WCNP 통합 제공 + 자체 OSS(SONiC/L3AF) |
| 복원력 | Triplet 분산 설계 + Concord+Gremlin 카오스 자동화 + 성숙도 레벨1~3 |
| 서비스 메시 | Istio + Linkerd 병행(WCNP VP 공식 확인) |

## 3. 출처

- [TechTarget - Walmart's multi-cloud strategy](https://www.techtarget.com/searchcloudcomputing/news/252522631/Walmarts-multi-cloud-strategy-cuts-millions-in-IT-costs)
- [Confluent - Real-Time Inventory](https://www.confluent.io/blog/walmart-real-time-inventory-management-using-kafka/)
- [SiliconANGLE - Walmart's supercloud](https://siliconangle.com/2023/01/17/walmarts-supercloud-cloud-native-kubernetes-based-platform-supercloud2/)
- [TechCrunch - OneOps](https://techcrunch.com/2016/01/26/walmart-launches-oneops-an-open-source-cloud-and-application-lifecycle-management-platform/)
- [Gremlin - Chaos Conf 2018 Walmart](https://www.gremlin.com/blog/vilas-veeraraghaven-practicing-chaos-engineering-at-walmart-chaos-conf-2018)
- [Concord Gremlin Plugin](https://concord.walmartlabs.com/docs/plugins-v2/gremlin.html)
