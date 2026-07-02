# Alibaba 추정 아키텍처

> 참조일: 2026-06-30 | 출처: Alibaba Cloud 공식 블로그·CNCF 사례연구 기반 추정.
> 상세 근거: [02_service_selection.md](../report/02_service_selection.md) #3

---

## 1. 전체 아키텍처 개요

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         Alibaba 아키텍처 전체상                            │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. 초기 구조                                                            │
│  [RPC 계층 — Apache Dubbo, 2017 Apache 최상위 프로젝트(Alibaba 발원)]      │
│                                                                          │
│         ▼                                                                │
│  2. 도메인 분리 및 서비스 구조                                           │
│  [Spring Cloud Alibaba 생태계 — 자체 발원 OSS 다수]                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                    │
│  │ Nacos    │ │ RocketMQ │ │ Sentinel │ │ Seata    │                    │
│  │(디스커버리│ │(메시징)  │ │(플로우제어│ │(분산     │                    │
│  │ /설정)   │ │          │ │/CB)      │ │ 트랜잭션)│                    │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                    │
│         │                                                                │
│         ▼                                                                │
│  [서비스 메시 — 3갈래 전략]                                               │
│  ┌────────────────────────────────────────────────────────────┐          │
│  │ ① 전통 Dubbo RPC(사이드카 이전)                                │          │
│  │ ② Dubbo3 Sidecar/Proxyless Mesh(xDS로 Istiod 직접 통신)        │          │
│  │ ③ ASM(Alibaba Cloud Service Mesh, Istio 1.22 호환 완전관리형)   │          │
│  └────────────────────────────────────────────────────────────┘          │
│         │                                                                │
│         ▼                                                                │
│  [컨테이너 오케스트레이션]                                                │
│  Sigma → PouchContainer(2017 OSS화) → ASI(K8s 기반, Kube-on-Kube)        │
│  10,000-노드 규모 클러스터 + Cilium(CNCF 사례연구)                       │
│                                                                          │
│  3. 플랫폼 운영 체계                                                     │
│  A) CD 계층 — 光棍节(광군제) 대비 핵심 기법                                │
│  ACK 기반 자동화 + Virtual Kubelet(분당 100 Pod 탄력확장)                 │
│  全链路压测(전체링크 스트레스 테스트) + 그림자 테이블(운영데이터 오염방지)  │
│                                                                          │
│  B) 관찰성 — ARMS                                                        │
│  OpenTelemetry/Prometheus 표준 지원, eBPF 무계측 모니터링                 │
│  관리형 Grafana/Prometheus 제공                                          │
│  (※ Apache SkyWalking은 Alibaba 발명 아님 — 개인 개발자 발원, 채택만)      │
│                                                                          │
│  C) 복원력 — Sentinel + ChaosBlade                                       │
│  ChaosBlade(2019 OSS, CNCF Sandbox, Dubbo 타임아웃/예외 주입까지 지원)     │
│  Remote multi-active architecture(2013~, 다도시 트랜잭션 유닛 분산)        │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```
<img width="1133" height="1388" alt="image" src="https://github.com/user-attachments/assets/12519458-dbff-4d0f-a6cc-db73626c4dd3" />


## 2. 5축 요약

| 축 | 내용 |
|---|---|
| 아키텍처 | MSA + Dubbo RPC 생태계(Nacos/RocketMQ/Sentinel/Seata) |
| CI/CD | ACK 자동화 + 全链路압测 + 그림자 테이블 |
| 관찰성 | ARMS(OTel/Prometheus 표준, eBPF 무계측) |
| 복원력 | Sentinel + ChaosBlade(CNCF Sandbox) |
| 서비스 메시 | 3중 전략(전통 Dubbo + Proxyless Mesh + ASM) — 6개사 중 가장 적극적 |

## 3. 출처

- [Apache Dubbo](https://dubbo.apache.org/en/overview/what/)
- [CNCF Case Study - Alibaba](https://www.cncf.io/case-studies/alibaba/)
- [10 Years of Double 11](https://www.alibabacloud.com/blog/10-years-of-double-11-the-evolution-and-upgrade-of-alibabas-cloudification-architecture_594160)
- [Alibaba Cloud - CI/CD on Kubernetes](https://www.alibabacloud.com/blog/how-does-alibaba-implement-cicd-based-on-kubernetes_595086)
- [Full-Link Stress Testing](https://medium.com/@alibaba-cloud/unveiling-the-secrets-behind-alibabas-full-scale-stress-testing-for-double-11-d97f62f829b3)
- [Alibaba Cloud - What is ARMS](https://www.alibabacloud.com/help/en/arms/product-overview/what-is-arms)
- [ChaosBlade](https://www.alibabacloud.com/blog/chaosblade---an-open-source-chaos-engineering-tool-by-alibaba_594850)
- [Dubbo Proxyless Mesh](https://www.alibabacloud.com/blog/an-exploration-and-improvement-of-dubbo-in-proxyless-mesh-mode_600313)
