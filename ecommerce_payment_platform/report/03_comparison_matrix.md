# 03 — 클라우드 네이티브 5축 비교 매트릭스

> 상세 근거/출처는 [02_service_selection.md](./02_service_selection.md) 참조.
> 범례: ◎ 매우 적극적/공개 자료 풍부 · ○ 적용/일부 공개 · △ 부분적/간접 정황만 확인 · × 미확인/미적용 추정

## 1. 한눈에 보는 비교표

| 축 | Shopify | Walmart | Alibaba | 11번가 | 당근마켓 | 카카오페이 |
|---|---|---|---|---|---|---|
| **아키텍처 패턴** | 모듈러 모놀리스 + Pod 샤딩 | MSA(Node.js) + 멀티클라우드 Triplet | MSA + Dubbo RPC 생태계 | MSA(Spring Cloud "Vine") | MSA(폴리글랏) + 일부 모노레포(당근페이) | MSA + 3상태 트랜잭션 모델 |
| **인프라** | GCP, K8s ~400클러스터 | Azure+GCP+프라이빗+엣지(1만 노드) | 자체 ASI(K8s, 1만+ 노드) | IDC + AWS EKS 하이브리드 | AWS(EKS/ECS/Aurora/DynamoDB) | AWS EKS + IDC + Kakao Cloud 3중 |
| **CI/CD 도구** | 자체 Shipit + Buildkite | OneOps(OSS)→Concord | ACK 기반 자동화 | 자체(도구명 비공개) | GitHub Actions+Kustomize+ArgoCD | 자체 Wallga(Jenkins+ArgoCD) |
| **배포 전략** | Canary(5%, 10분) 자동배포 | 공개 정보 부족 | 全链路압测(전체링크 스트레스) | Feature Flag + 무중단(심볼릭링크) | GitOps Sync 자동화 | **Rolling/Canary/Blue-Green 모두 지원** |
| **배포 빈도/속도** | 하루 ~150회 | 공개 정보 부족 | 공개 정보 부족 | 공개 정보 부족 | 5시간→30분 단축 | 공개 정보 부족 |
| **관찰성 스택** | Grafana+Cortex+Loki+Tempo | WCNP 통합 + SONiC/L3AF(자체 OSS) | ARMS(OTel/Prometheus 표준) | Prometheus+Micrometer, Splunk+ES SIEM | Prometheus+Grafana, 감사로그+EventBridge+Slack | OTel+ClickHouse(ClickStack)+Grafana |
| **분산 트레이싱** | Tempo | Jaeger/Zipkin/OTel 언급 | OTel 지원 | 공개 정보 부족 | 공개 정보 부족 | OTel 도입(활용 사례는 추정) |
| **서킷 브레이커/복원력 라이브러리** | 자체 Semian + Toxiproxy | 공개 정보 부족 | Sentinel | Fast-Fail 자체구현(범용 프레임워크 X) | 공개 정보 부족 | 공개 정보 부족(코드레벨 직접구현) |
| **카오스 엔지니어링** | BFCM 연계 정례화 | **Concord+Gremlin, 성숙도 레벨1~3 체계** | **ChaosBlade(CNCF Sandbox)** | **Chaos Mesh+ToxiProxy, OSS 컨트리뷰션** | 사후 대응형(워밍업 시스템) | 순수 카오스 미확인, DR훈련만 확인(계열사) |
| **공개 장애 포스트모템** | 일부 공개(BFCM 회고) | 미확인 | 미확인 | 미확인 | **공개(Elasticsearch 장애 상세 회고)** | 미확인 |
| **서비스 메시** | 평가만(비용 문제로 보류) | **Istio+Linkerd 병행(VP 확인)** | **Dubbo Proxyless+ASM(3중 전략)** | 미도입 추정 | 일부 도메인 도입(채용공고 확인) | 미도입 추정(코드레벨 대체) |
| **트래픽 관리 방식** | Pod 샤딩(애플리케이션 레벨) | 서비스 메시 | Dubbo(애플리케이션) + ASM(메시) | Spring Cloud LoadBalancer(애플리케이션) | gRPC + 일부 메시 | GSLB+DNSCache(인프라 레벨) |

## 2. 축별 성숙도 순위 (근거 기반 상대 평가)

```
[서비스 메시 채택 적극성]
Alibaba(3중 전략) > Walmart(Istio+Linkerd) > 당근마켓(일부 도메인) > Shopify(평가 후 보류) > 11번가 ≈ 카카오페이(미도입 추정)

[카오스 엔지니어링 성숙도] (2026-06-30 재조사로 갱신 — Walmart 평가 상향)
Alibaba(ChaosBlade, CNCF) ≈ 11번가(Chaos Mesh+OSS 기여) ≈ Walmart(Concord+Gremlin, 성숙도 레벨체계, Chaos Conf 발표) > Shopify(BFCM 정례화) > 당근마켓(사후 대응형) > 카카오페이(순수 카오스 미확인, DR훈련만 계열사 단위 확인)

[관찰성 투명성/공개 수준]
Shopify(OSS 스택 풀공개) ≈ 당근마켓(장애 디테일 공개) > Alibaba(ARMS 상품화) > 11번가/카카오페이(SIEM·로그플랫폼 공개하나 트레이싱 활용 불명) > Walmart(통합됐다고만 언급, 세부 비공개)

[배포 파이프라인 정교함]
Shopify(Canary 자동화+10분검증) ≈ 카카오페이(Rolling/Canary/BlueGreen 선택형) > 당근마켓(GitOps+5시간→30분) > Alibaba(全链路압测) > 11번가(무중단 기법은 정교하나 도구체인 비공개) > Walmart(자체 PaaS 역사는 깊으나 최신 정보 부족)
```

## 3. 기업별 강점·약점 분석

| 기업 | 강점 | 약점/리스크 |
|---|---|---|
| **Shopify** | 모듈러 모놀리스+Pod 샤딩으로 MSA 오버엔지니어링 회피, 메시 도입 비용을 정량 검증 후 합리적 보류, OSS 관찰성 스택 풀공개로 투명성 최상위 | 자체 메시 평가 이후 행보 비공개라 최신 판단 기준 확인 어려움, 일반 요금제는 SLA 미보장 |
| **Walmart** | 멀티클라우드 Triplet Model로 벤더 록인 회피, 자체 OSS(SONiC/L3AF) 개발력, 카오스 엔지니어링 성숙도 레벨 체계 보유(재조사로 확인) | 배포 전략·빈도·자체 SLA % 등 핵심 수치가 거의 비공개, 정보 파편화로 최신 아키텍처 전모 파악 어려움 |
| **Alibaba** | 서비스 메시 3중 전략(Dubbo Proxyless+ASM)으로 가장 정교한 마이그레이션 경로 보유, ChaosBlade로 카오스 엔지니어링 OSS 생태계 직접 주도, 全链路압测로 실트래픽 기반 검증 체계화 | 자국 시장 점유율 자체는 하락 추세(경쟁 심화), 전자상거래 본체(Taobao/Tmall) 자체 SLO 공시는 없고 클라우드 SLA로만 신뢰성 방증 |
| **11번가** | 무중단 배포(심볼릭링크), Eureka 카오스 테스트로 실제 Spring Cloud Netflix OSS에 컨트리뷰션한 유일한 사례 — 기술적 깊이 입증 | 시장점유율 3~4위로 하락 중(2024 매출 -35%), 서비스 메시 미도입, 공개 status page/SLA 전무, 하이브리드 인프라(IDC+EKS) 과도기 장기화 |
| **당근마켓** | 중고거래 시장 압도적 1위(MAU 4.5배 격차), 유일하게 투명한 장애 포스트모템 공개, kube-apiserver 감사로그 같은 독특한 보안 가시성 사례 | 서비스 메시는 일부 도메인(채용공고 확인)에 그침, 공개 SLO/SLA 수치 자체는 없음(장애 회고는 사후적) |
| **카카오페이** | 간편결제 빅4 중 2위(TPV 167조원, +19%), Rolling/Canary/Blue-Green 배포 전략 모두 지원하는 가장 유연한 CI/CD, **계열사 단위 SLO 수치(99.993%)를 명시적으로 공시**하는 유일한 국내 사례 | 순수 카오스 엔지니어링 실시 여부 미확인, 서비스 메시 미도입 추정(코드레벨 대체), 2022년 판교 화재로 그룹 전체 10시간+ 장애 이력 |

## 4. 1차 관찰 — 페인포인트 후보 (04_pain_points 단계 인풋)

1. **국내 기업의 신뢰성 공시 부재**: 11번가·카카오페이는 공개 포스트모템이 사실상 없음. 당근마켓만 예외적으로 투명(Elasticsearch 장애 회고). → 우리 프로젝트는 Blameless Postmortem 문화를 SLO 대시보드와 함께 설계 단계부터 내재화할 차별화 포인트로 삼을 수 있음.
2. **서비스 메시 도입의 비용 트레이드오프**: Shopify 사례(Envoy 사이드카 CPU 비용 정량화)는 "무조건 메시 도입"이 아니라 트래픽 규모·조직 성숙도에 따른 단계적 도입 판단 근거가 필요함을 시사.
3. **하이브리드 인프라 과도기의 복잡성**: 11번가(IDC+EKS), 카카오페이(EKS+IDC+KC 3중)처럼 국내 기업 다수가 완전 클라우드 네이티브 전환을 끝내지 못한 과도기 상태 — Service Discovery 이중화(Eurekube 사례) 같은 임시 솔루션이 기술 부채화될 위험.
4. **카오스 엔지니어링 격차**(2026-06-30 재조사로 수정: Walmart는 Concord+Gremlin 기반 카오스 실험과 팀별 성숙도 레벨 체계가 풍부히 확인됨 — "공개 정보 부족" 평가 철회): 6개 기업 중 **카카오페이만 유일하게** 순수 카오스 엔지니어링 실시 여부가 공개 자료로 확인되지 않음(인접한 DR 훈련은 계열사 단위로 확인) — 결제 도메인 특유의 장애 시뮬레이션 보수성을 시사.
5. **MSA 만능주의에 대한 반례**: Shopify(모듈러 모놀리스), 당근페이(Clean Architecture 모노레포)는 모두 의도적으로 "과도한 마이크로서비스 분리"를 피한 사례 — 우리 프로젝트의 Bounded Context 설계 시 서비스 개수를 최소화하는 근거로 활용 가능.

## 5. 다음 단계

- [x] 04_pain_points.md: MECE 분류 + 피쉬본 다이어그램으로 근본원인 분석, 6개로 구체화
- [x] 05_differentiation.md: Value Proposition Canvas로 우리 프로젝트의 차별화 전략 정의
- [x] 00_service_selection.md, 06_slo_reliability.md, 07_chaos_engineering.md 작성
