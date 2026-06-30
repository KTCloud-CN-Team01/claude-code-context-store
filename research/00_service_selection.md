# 0번 과제 — 비교 대상 서비스 선정 및 근거

> 도메인: e-Commerce 결제·콘텐츠 플랫폼 (팀원 Team01의 `b2c_platform`과 동일 도메인, 비교 기업은 겹치지 않게 구성)
> 선정 기준 3축: ① 시장 점유율(해당 시장에서의 위치) ② 운영 신뢰성(규모·트랜잭션 처리량·공식 지표로 입증되는 안정성) ③ 기술 블로그 공유 정도(엔지니어링 콘텐츠의 양과 깊이)
> 모든 수치는 실제 WebSearch로 검증된 출처만 인용. 검색 시점 2026-06(데이터 자체는 출처별로 2024~2025년 최신값).

---

## 해외 3개사

### Shopify (이커머스 SaaS 플랫폼)
- **시장 점유율**: 글로벌 이커머스 플랫폼 시장 약 10.32%로 4위권, **미국 이커머스 소프트웨어 시장은 약 29%로 지배적 1위**. 2025년 GMV 3,780억 달러(전년比 +29.3%), 전 세계 480만 매장 운영.
- **운영 신뢰성**: 2024 블랙프라이데이 피크 시 초당 470만 요청·분당 2.84억 요청 처리, BFCM 기간 99.9% 가동률 달성.
- **기술 블로그 공유도**: shopify.engineering에 아키텍처(Pods)·CI/CD·관찰성·복원력(Semian/Toxiproxy) 전 영역에 걸쳐 매우 상세한 1인칭 엔지니어링 글 다수, OSS 다수 공개(Semian, Toxiproxy).
- 출처: [ECDB](https://ecdb.com/blog/shopify-s-influence-on-global-e-commerce-is-growing/5181), [Demandsage](https://www.demandsage.com/shopify-market-share/), [ByteByteGo](https://blog.bytebytego.com/p/how-shopify-prepares-for-black-friday)

### Walmart (글로벌 유통 1위, 옴니채널)
- **시장 점유율**: 미국 이커머스 시장에서 **Amazon에 이은 확고한 2위**(점유율 6~9%대, 출처별 편차 존재). 디지털 매출 FY2024 첫 1,000억 달러 돌파, 글로벌 이커머스 순매출 전년比 +23%(~300억 달러).
- **운영 신뢰성**: 멀티클라우드 Triplet Model + 약 1만 노드 엣지 인프라로 하루 110억 건 이벤트 처리(Kafka 기반). 공식 status page(walmart.statuspage.io) 운영.
- **기술 블로그 공유도**: Walmart Global Tech 블로그·Concord 공식 문서·Chaos Conf 발표 등으로 카오스 엔지니어링까지 포함한 폭넓은 자료 확인(단, 배포 빈도 등 일부 핵심 수치는 비공개).
- 출처: [Statista](https://www.statista.com/statistics/274255/market-share-of-the-leading-retailers-in-us-e-commerce/), [SEC 8-K FY2024](https://www.sec.gov/Archives/edgar/data/0000104169/000010416924000170/earningspresentationfy25.htm), [Walmart Status](https://walmart.statuspage.io/)

### Alibaba (아시아 최대 이커머스)
- **시장 점유율**: 중국 최대 이커머스 사업자 지위 유지하나, Douyin·Pinduoduo 경쟁 심화로 GMV 점유율이 2020년 약 50%에서 **약 32%까지 하락 추세**(출처에 따라 변동).
- **운영 신뢰성**: 광군제(光棍节/솽스이) 행사에서 결제 처리량 매년 기록 경신, 2020년 기준 초당 58.3만 건 결제(2009년 첫 해 대비 1,457배), PolarDB 초당 8,700만 TPS 사례.
- **기술 블로그 공유도**: Apache Dubbo·Nacos·Sentinel·Seata·ChaosBlade 등 다수 OSS를 직접 주도·기여, Alibaba Cloud 블로그에 아키텍처 사례 풍부.
- 출처: [Statista](https://www.statista.com/statistics/1625088/china-estimated-top-online-retailers-ecommerce-market-share/), [SCMP](https://www.scmp.com/tech/e-commerce/article/3038539/how-alibaba-powered-billions-transactions-singles-day-zero-downtime)

## 국내 3개사

### 11번가 (오픈마켓)
- **시장 점유율**: 한국 이커머스 시장에서 쿠팡(~30%)·네이버쇼핑(~25%)에 이어 **약 12~13%로 3~4위권**(G마켓과 경쟁). 단, 2024년 영업손실 754억 원(전년比 40% 개선), 매출은 35% 감소한 5,618억 원으로 시장 지위 약화 추세.
- **운영 신뢰성**: 공식 status page/uptime 수치는 확인되지 않으나, 라이브커머스 'LIVE11' 재구축(시청수 4배·거래액 7배), 전시 서버 최적화(서버 110→70대 감축하며 TPS 33% 증대) 등 정량적 운영 개선 사례 다수 공개.
- **기술 블로그 공유도**: 11st-tech.github.io에 무중단 배포·Feature Flag·카오스 테스트(Spring Cloud Netflix OSS 컨트리뷰션 사례 포함) 등 기술 깊이가 상당한 1인칭 글 다수.
- 출처: [디지털투데이](https://www.digitaltoday.co.kr/news/articleView.html?idxno=522436), [바이라인네트워크](https://byline.network/2025/02/25_11st/), [11번가 TechBlog](https://11st-tech.github.io/2025/11/26/dpwas-improvement/)

### 당근마켓 (C2C 중고거래 마켓플레이스)
- **시장 점유율**: 중고거래 앱 시장에서 **MAU 2,127만 명(2025-05)으로 압도적 1위**, 2위 번개장터(475만 명) 대비 약 4.5배 격차. 누적 가입자 4,300만 명(2025-03). 과거 자료 기준 중고거래 업종 점유율 93% 보도.
- **운영 신뢰성**: 2024년 매출 1,891억 원(전년比 +48%)·영업이익 376억 원으로 2년 연속 흑자. Elasticsearch 장애를 search-coordinator 워밍업 시스템으로 해결한 사례처럼 구체적 운영 개선 데이터 공개.
- **기술 블로그 공유도**: medium.com/daangn에 검색·SRE·보안(kube-apiserver 감사로그)까지 폭넓고 투명한 1인칭 기술 글 다수, 6개사 중 유일하게 상세 장애 포스트모템 공개.
- 출처: [플래텀](https://platum.kr/archives/264239), [전자신문](https://www.etnews.com/20250523000112), [플래텀](https://platum.kr/archives/255887)

### 카카오페이 (간편결제)
- **시장 점유율**: 국내 간편결제 시장에서 네이버페이(51.5%)에 이어 **약 25.1%로 2위**(토스페이 13.2%와 함께 "빅4" 구성). 2024년 TPV(거래액) 167.3조 원(+19%), 연결 매출 7,662억 원(+25%), 카카오페이머니 사용자 3,100만 명 돌파.
- **운영 신뢰성**: 카카오페이증권이 99.993%(2024)·99.99% 목표(2025) **SLO 수치를 명시적으로 공시**하는 6개사 중 유일한 국내 사례. AWS EKS+IDC+Kakao Cloud 3중 하이브리드로 "99.999%" 가용성 지향.
- **기술 블로그 공유도**: tech.kakaopay.com에 MSA 전용 태그를 운영하며 분산 트랜잭션·CI/CD(Wallga)·로그 플랫폼(Pallas v2) 등 결제 도메인 특화 기술 글 다수.
- 출처: [오픈서베이](https://blog.opensurvey.co.kr/article/ds-payment-2025-2/), [카카오 FY2024 Annual Report](https://t1.kakaocdn.net/pay_brand_admin/file/wyyRvaJE4aDiSlsfYVUGk/FY2024AnnualReport.pdf), [카카오페이 기술블로그 - 멀티클러스터](https://tech.kakaopay.com/post/multi-cluster/)

---

## 선정 근거 종합

- **시장 점유율**: 6개사 모두 각자 시장에서 1~4위 내 상위권(11번가만 하락 추세지만 여전히 Top4) — "대표성 있는 서비스 비교"라는 과제 취지에 부합.
- **운영 신뢰성**: 6개사 모두 정량적 운영 지표(가동률·처리량·거래액 등)를 공개 자료로 확인 가능 — 추정이 아닌 근거 기반 비교가 가능한 대상만 선정.
- **기술 블로그 공유도**: 6개사 모두 자체 기술 블로그 또는 컨퍼런스 발표를 운영 — 아키텍처 비교에 필요한 1차 자료 확보 가능. 다만 국내 3개사(특히 11번가·카카오페이)는 해외 대비 장애 포스트모템 공개에는 소극적(→ [04_pain_points.md](./04_pain_points.md) PP-4로 연결).
- **Team01과의 차별화**: Team01(Netflix/Spotify/Amazon, Coupang/MarketKurly/Inflearn)과 기업 구성을 겹치지 않게 구성하면서도 동일 도메인(e-Commerce 결제·콘텐츠) 안에서 비교 가능하도록 설계.

## 다음 단계

- [x] [01_company_profiles.md](./01_company_profiles.md): 5축 기술스택 상세 프로파일
- [x] [02_comparison_matrix.md](./02_comparison_matrix.md): 비교 매트릭스 + 강점/약점
- [x] [04_pain_points.md](./04_pain_points.md), [05_differentiation.md](./05_differentiation.md): 페인포인트 및 차별화 전략
