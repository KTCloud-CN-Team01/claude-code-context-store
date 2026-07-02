# 11번가 추정 아키텍처

> 참조일: 2026-06-30 | 출처: 11st-tech.github.io(공식 기술 블로그) + Spring Camp 2018 발표자료 기반 추정.
> 상세 근거: [02_service_selection.md](../report/02_service_selection.md) #4

---

## 1. 전체 아키텍처 개요

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         11번가 아키텍처 전체상                             │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. 초기 구조                                                            │
│  [Spring Cloud 기반 자체 플랫폼 "Vine" (2016 모놀리스→MSA 전환)]            │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐                             │
│  │ Zuul     │ │ Eureka   │ │ Config Server │   2018년 기준               │
│  │(API GW)  │ │(Discovery)│ │              │   ~600 인스턴스 / ~60 서비스 │
│  └──────────┘ └──────────┘ └──────────────┘                             │
│                                                                          │
│         ▼                                                                │
│  2. 도메인 분리 및 서비스 구조 — 하이브리드 인프라 과도기                   │
│  [하이브리드 인프라]                                                      │
│  ┌──────────────────────┐         ┌──────────────────────┐              │
│  │ IDC(온프레미스, 레거시)│ ◄─────► │ AWS EKS(신규)          │              │
│  └──────────────────────┘         └──────────────────────┘              │
│   완전 AWS 이전 공식 미확인 — 통념과 달리 장기 과도기                       │
│         │                                                                │
│         ▼                                                                │
│  [서비스 디스커버리 브릿지 — Eurekube Operator(자체 개발)]                  │
│  Spring Cloud Eureka ↔ EKS 서비스 디스커버리 연결(임시 브릿지 솔루션)        │
│                                                                          │
│  3. 플랫폼 운영 체계                                                     │
│  A) CD 계층                                                              │
│  Spring Batch 무중단 배포: 타임스탬프 디렉토리 + atomic switch + readlink  │
│  Feature Flag(OpenFeature) 기반 점진적 배포/실험                          │
│                                                                          │
│  B) 관찰성                                                               │
│  Prometheus + Micrometer(메트릭)                                        │
│  Splunk + ElasticSearch 이중 SIEM(GuardDuty/WAF/CloudTrail/VPC FlowLogs) │
│  ※ 분산 트레이싱·APM 도구 명시 사례는 공개 정보 부족                       │
│                                                                          │
│  C) 복원력 — OSS 컨트리뷰션까지 도달한 카오스 엔지니어링                     │
│  Chaos Mesh(EKS) + ToxiProxy(IDC)로 Eureka 카오스 테스트                  │
│  발견한 HttpClient 타임아웃 버그를 Spring Cloud Netflix OSS에 직접 기여     │
│  (Spring Cloud 2022.0.0에 반영됨 — 6개사 중 유일한 업스트림 기여 사례)       │
│  전시 서버 최적화: 110→70대 감축하며 TPS 33%↑(MongoDB Fast-Fail)          │
│                                                                          │
│  [서비스 메시 — 미도입]                                                   │
│  Spring Cloud Netflix 생태계(Eureka/Zuul/LoadBalancer)를                 │
│  애플리케이션 레벨에서 지속 확장 — 사이드카 메시 미채택 추정                │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

## 2. 5축 요약

| 축 | 내용 |
|---|---|
| 아키텍처 | MSA(Spring Cloud "Vine") + IDC/EKS 하이브리드 과도기 |
| CI/CD | 무중단 배포(심볼릭링크) + Feature Flag |
| 관찰성 | Prometheus+Micrometer, Splunk+ES SIEM(트레이싱은 비공개) |
| 복원력 | Chaos Mesh+ToxiProxy → Spring Cloud Netflix OSS 컨트리뷰션 |
| 서비스 메시 | 미도입(애플리케이션 레벨 대체 전략) |

## 3. 출처

- [Spring Camp 2018 발표자료](https://www.slideshare.net/balladofgale/spring-camp-2018-11-spring-cloud-msa-1)
- [Eurekube Operator](https://11st-tech.github.io/2022/07/20/eurekube-operator/)
- [AWS 고객 사례 - 11번가](https://aws.amazon.com/ko/solutions/case-studies/11st/)
- [심볼릭 링크 무중단 배포](https://11st-tech.github.io/2023/12/11/spring-batch-non-stop-deploy/)
- [Feature Flag](https://11st-tech.github.io/2023/11/07/openfeature/)
- [Service Discovery DR 2부 - Chaos Test](https://11st-tech.github.io/2022/12/30/eureka-disaster-recovery-2/)
- [전시 서버 최적화](https://11st-tech.github.io/2025/11/26/dpwas-improvement/)
