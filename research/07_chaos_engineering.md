# 0번 과제 (선택) — 카오스 엔지니어링 적용 수준 비교

> [01_company_profiles.md](./01_company_profiles.md), [02_comparison_matrix.md](./02_comparison_matrix.md)의 카오스 엔지니어링 항목을 심화. 2026-06-30 Walmart·카카오페이 재조사 결과를 반영해 기존 평가를 갱신.

---

## 1. 기업별 카오스 엔지니어링 상세

### Alibaba — 가장 적극적(OSS 생태계 직접 주도)
- **ChaosBlade**: 2019년 OSS 공개, **CNCF Sandbox 프로젝트**로 등재. Dubbo RPC 호출의 타임아웃·예외 주입까지 지원하는 등 자사 기술 스택(Dubbo)에 특화된 카오스 도구를 직접 개발·표준화.
- 全链路压测(전체 링크 스트레스 테스트)와 결합해 광군제(光棍节) 대비 실 트래픽 재생 테스트를 정례화, **그림자 테이블(Shadow Table)**로 운영 데이터 오염 방지.
- 출처: [ChaosBlade 소개](https://www.alibabacloud.com/blog/chaosblade---an-open-source-chaos-engineering-tool-by-alibaba_594850), [GitHub - chaosblade-io](https://github.com/chaosblade-io/chaosblade)

### 11번가 — OSS 컨트리뷰션까지 도달한 유일한 사례
- **Chaos Mesh(EKS) + ToxiProxy(IDC)**로 Eureka 서비스 디스커버리에 대한 카오스 테스트 수행, 발견한 HttpClient 타임아웃 버그를 **Spring Cloud Netflix OSS에 직접 기여**(Spring Cloud 2022.0.0에 반영) — 6개사 중 유일하게 "카오스 실험 → 실제 업스트림 OSS 개선"까지 이어진 사례.
- 출처: [Service Discovery DR 2부 - Chaos Test](https://11st-tech.github.io/2022/12/30/eureka-disaster-recovery-2/)

### Walmart — 재조사로 평가 대폭 상향(기존 "공개 정보 부족" → 풍부히 확인됨)
- 자체 CI/CD 플랫폼 **Concord + Gremlin** 연동으로 카오스 실험 자동화, Chaos Toolkit·Chaos Monkey·Toxiproxy·Pod Kill 실험 등을 다루는 연재 블로그 시리즈 운영.
- "카오스 엔지니어링"보다 **"리질리언스 엔지니어링"** 용어를 의도적으로 선호 — 단발성 실험이 아닌 **팀별 성숙도 레벨(1~3단계) 인증 체계**로 제도화(Chaos Conf 2018 발표 시점 50개+ 팀이 레벨2 통과).
- 자체 진단 도구 **"Resiliency Doctor"** 개발(애플리케이션 배포 전반의 취약점 점검).
- 출처: [Gremlin - Chaos Conf 2018 Walmart 발표](https://www.gremlin.com/blog/vilas-veeraraghaven-practicing-chaos-engineering-at-walmart-chaos-conf-2018), [Walmart Global Tech - Chaos Engineering 태그](https://medium.com/walmartglobaltech/tagged/chaos-engineering), [Concord Gremlin Plugin](https://concord.walmartlabs.com/docs/plugins-v2/gremlin.html)

### Shopify — 대규모 이벤트(BFCM) 연계형
- BFCM(블랙프라이데이/사이버먼데이) 대비 연 5회 대규모 스케일 테스트 + 리전 페일오버 훈련 + 카오스 엔지니어링을 정례화.
- Toxiproxy(장애 주입 OSS)를 자체 개발해 공개 — 단, Walmart/Alibaba처럼 "성숙도 레벨 체계"나 "전사 표준 도구화" 수준의 제도화 근거는 확인되지 않음.
- 출처: [BFCM Readiness 2025](https://shopify.engineering/bfcm-readiness-2025), [GitHub - Shopify/toxiproxy](https://github.com/Shopify/toxiproxy)

### 당근마켓 — 사후 대응형(사전 카오스 실험과는 결이 다름)
- 사전에 설계된 카오스 실험이 아니라, **실제 장애(Elastic Operator 롤링 재시작 중 Cold New Node로 인한 에러율 60% 급증)를 계기로 한 사후 개선**(search-coordinator 프록시 + Redis 분산락 워밍업 시스템)이 핵심 사례.
- 사전 예방적 카오스 엔지니어링 실시 여부는 공개 자료로 확인되지 않음 — "장애 대응력"은 검증되지만 "장애 예방형 실험 문화"와는 구분 필요.
- 출처: [Elasticsearch Warm-Up Part 2](https://medium.com/daangn/running-elasticsearch-on-kubernetes-the-easy-way-part-2-data-node-warm-up-0d81d433c5c1)

### 카카오페이 — 결제 도메인 중 유일하게 미확인(인접 DR 훈련만 확인)
- 순수 "카오스 엔지니어링" 키워드로는 tech.kakaopay.com을 포함한 공개 자료에서 확인되지 않음(시도한 검색: "카카오페이 카오스 엔지니어링", "site:tech.kakaopay.com 카오스" 등 — 모두 직접 결과 없음).
- **인접 영역인 DR(재해복구) 훈련**은 확인됨: 카카오페이증권이 2025-11-23 데이터센터 침수 가정 5시간 'IT 재난 대응 훈련' 실시(기술조직 약 40명 참여, Active-Active 이중화 검증, 향후 클라우드 포함 3중 안전망으로 확대 예정). 단 카카오페이증권(계열사) 명의이며 카카오페이 본체 단독 사례인지는 불명확.
- 카카오 그룹 차원에서는 2022년 판교 화재 이후 [Kakao Reliability Report](https://tech.kakao.com/2023/09/14/kakao-reliability-report/)에서 연 2~3회 비정기 장애 대응 훈련 체계화를 명시하나, 카카오페이가 이 훈련에 구체적으로 포함되는지는 확인되지 않음.
- 출처: [카카오페이증권 IT 재난훈련 보도](https://www.fnnews.com/news/202511251818005108), [tech.kakao.com 안정성 보고서](https://tech.kakao.com/2023/09/14/kakao-reliability-report/)

## 2. 갱신된 성숙도 순위

```
Alibaba(ChaosBlade, CNCF 등재 + OSS 생태계 주도)
  ≈ 11번가(Chaos Mesh+ToxiProxy, 실제 업스트림 OSS 기여까지 도달)
  ≈ Walmart(Concord+Gremlin 자동화 + 성숙도 레벨 인증 체계 + 전담 발표)
  >  Shopify(BFCM 연계 정례화 + 자체 OSS Toxiproxy, 제도화 근거는 약함)
  >  당근마켓(사후 대응형 — 사전 예방적 실험 문화는 미확인)
  >  카카오페이(순수 카오스 미확인, 인접 DR 훈련은 계열사 단위로만 확인)
```

기존 02_comparison_matrix.md의 순위([서비스 메시 채택 적극성] 등과 동일 형식)에서 Walmart를 "공개 정보 부족" 최하위 그룹으로 분류했던 것은 **2026-06-30 재조사로 정정됨**.

## 3. 도메인별 패턴 — 이커머스 vs 결제

| 구분 | 이커머스/오픈마켓(Alibaba·11번가·Shopify·당근마켓) | 결제(카카오페이) |
|---|---|---|
| 카오스 엔지니어링 공개 수준 | 4/4 기업 모두 확인(정도 차이는 있음) | 0/1 — 순수 카오스 미확인 |
| 대체 메커니즘 | 사전 예방적 실험(Chaos Mesh/ChaosBlade/Gremlin) | 사후적 DR 훈련(계열사 단위) |

표본이 결제 도메인 1개사뿐이라 일반화는 제한적이나, **04_pain_points.md PP-3(결제 도메인에서는 카오스 엔지니어링 실시 자체가 보수적)**의 근거로 활용 가능 — 결제 트랜잭션 무결성에 대한 리스크 회피 성향이 사전 장애 주입 실험을 꺼리게 만드는 요인으로 추정.

## 4. 다음 단계

- [ ] (3)단계(복원력·관찰성) 설계 시: 결제 도메인을 다루는 우리 프로젝트에서 카오스 실험을 1~2회 이상 실시하고 **결과를 공개**하는 것 자체가, 6개 비교 대상 중 결제 도메인(카카오페이)이 비워둔 공백을 메우는 명확한 차별화 지점.
