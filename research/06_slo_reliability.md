# 0번 과제 (선택) — SLO/신뢰성 공시 방식 비교

> [04_pain_points.md](./04_pain_points.md) PP-4("국내 기업 포스트모템 공개 문화 부재")를 검증하기 위한 보강 리서치.
> 각 기업이 ① 공개 status page 운영 여부 ② 공식 SLA/SLO 수치 공시 여부 ③ 장애 시 투명성 공개 사례를 기준으로 비교.

---

## 1. 기업별 SLO/신뢰성 공시 현황

### Shopify
- **공개 status page**: [shopifystatus.com](https://www.shopifystatus.com/) — 컴포넌트별 상태 + [수개월치 장애 이력](https://www.shopifystatus.com/history) 공개. 최근 90일 기준 가동률 99.99% 표기.
- **공식 SLA**: Shopify Plus(엔터프라이즈) 등급은 Checkout/Storefront API 가용성 **99.9%**를 SLA로 보장(위반 시 크레딧 지급), 상위 등급은 **99.99%**(연간 다운타임 53분 이하) 명시. 단, 일반(Basic/Grow/Advanced) 요금제는 SLA 미제공.
- 출처: [Shopify 공식 SLA 블로그](https://www.shopify.com/blog/service-level-agreement), [DevHelm SLA Report Card](https://devhelm.io/sla/shopify)

### Walmart
- **공개 status page**: [walmart.statuspage.io](https://walmart.statuspage.io/) (Statuspage.io 기반) 운영, 최근 30일 가동률(예: 99.6~100%) 표기.
- **공식 SLA**: 자체 소비자 대상 SLA % 공개는 확인 안 됨. 인프라(Google Cloud Spanner 등) 사용 시 99.999% 가용성을 언급하나 이는 **클라우드 벤더의 SLA**이지 Walmart 자체 공표 SLO가 아님.
- 출처: [UptimeRobot 모니터링](https://uptimerobot.com/status/walmart/), [Enginuity 뉴스레터](https://newsletter.enginuity.software/p/walmart-data-platform-with-cloud-spanner)

### Alibaba — 클라우드 플랫폼과 전자상거래 본체 구분 필요
- **Alibaba Cloud(클라우드 플랫폼)**: 공식 SLA 다수 공개 — ECS 단일 인스턴스 99.975%, 멀티존 99.995%, 네트워크 서비스 99.95% 등 서비스별 세분화. 공개 status page도 운영([status.alibabacloud.com](https://status.alibabacloud.com/)).
- **Taobao/Tmall(전자상거래 본체)**: 자체 가동률/SLO 공개 status page 없음. 더블11 행사 시 자사 클라우드 인프라(Apsara 등) 활용 사례 소개는 있으나 **정량적 안정성 지표 비공개**.
- 출처: [Alibaba Cloud SLA 발표](https://www.alibabacloud.com/blog/alibaba-clouds-latest-sla-has-a-multi-instance-availability-of-99-995%25_595695), [Double 11 Customer Story](https://www.alibabacloud.com/en/customers/double-11?_p_lc=1)

### 11번가
- **공개 status page**: 확인 안 됨.
- **공식 SLA/SLO**: 확인 안 됨. 이용약관상 "불가항력적 장애 시 책임 면제" 조항만 존재, 장애 시 개별 공지사항 게시만 확인.
- 출처: [11번가 이용약관](https://www.11st.co.kr/annc/AnncMainPreview.tmall?method=getProvision&anncCd=01)

### 당근마켓
- **공개 status page**: 확인 안 됨.
- **공식 SLA/SLO**: 공시 확인 안 됨. 다만 기술 블로그에서 SRE 조직·장애 대응 체계의 존재는 시사되며, **유일하게 장애 자체의 상세 회고(Elasticsearch Cold Node 이슈)를 1인칭으로 공개** — 수치 공시는 없지만 사후 투명성은 6개사 중 최고 수준.
- 출처: [Elasticsearch Warm-Up Part 2](https://medium.com/daangn/running-elasticsearch-on-kubernetes-the-easy-way-part-2-data-node-warm-up-0d81d433c5c1)

### 카카오페이
- **공식 SLO 수치 공개 — 6개사 중 유일한 명시적 사례**: 카카오페이증권이 기술 블로그에서 **2024년 SLO 99.993%, 2025년 목표 99.99%**를 직접 공시. 증권업 특성상 99.999%를 지향하며 멀티/하이브리드 클러스터 전략을 그 근거로 설명.
- **공개 status page**: 자체 운영 없음(개발자용 [카카오 오픈 API 상태](https://developers.kakao.com/status)는 플랫폼 API 한정).
- **그룹 차원 투명성**: 2022년 SK C&C 판교 데이터센터 화재로 카카오 전 서비스(카카오페이 포함) 10시간+ 마비 — 이후 카카오는 [Kakao Reliability Report](https://t1.kakaocdn.net/kakaocorp/kakaocorp/admin/promise/report/KakaoReliabilityReport_online.pdf)와 [공식 사과문](https://www.kakaocorp.com/page/detail/9814)을 발간한 **사후 투명성 공개 사례** 보유.
- 출처: [카카오페이 기술블로그 - 멀티/하이브리드 클러스터](https://tech.kakaopay.com/post/multi-cluster/)

## 2. 비교 요약표

| 기업 | 공개 status page | 공식 SLA/SLO 수치 | 장애 투명성 공개 |
|---|---|---|---|
| Shopify | ◎ (shopifystatus.com) | ◎ (Plus 99.9~99.99%) | ○ (이력 페이지) |
| Walmart | ○ (statuspage.io) | × (벤더 SLA만 언급) | △ |
| Alibaba | ◎ (클라우드만), × (이커머스 본체) | ◎ (클라우드만) | × (이커머스 본체) |
| 11번가 | × | × | × |
| 당근마켓 | × | × | ◎ (회고는 최고 수준이나 수치 공시는 없음) |
| 카카오페이 | × | ◎ (카카오페이증권 SLO 명시) | ○ (그룹 차원 사후 보고서만) |

## 3. 해석 — 해외 vs 국내 패턴 차이

- **해외 3개사(Shopify·Walmart·Alibaba)**: 셋 다 최소 1개 이상의 공개 status page를 운영하며, Shopify·Alibaba(클라우드)는 정량적 SLA까지 명시 — **"가동률을 숫자로 약속하는" 문화**가 정착.
- **국내 3개사(11번가·당근마켓·카카오페이)**: 자체 운영 status page는 전무. 다만 패턴이 균일하지 않음 — 당근마켓은 "사후 회고"로, 카카오페이는 "사전 SLO 수치 공시"(카카오페이증권 한정)로 서로 다른 방식의 투명성을 보임. 11번가는 두 방식 모두 확인되지 않아 **국내 3개사 중 신뢰성 공시가 가장 취약**.
- **PP-4("국내 기업 포스트모템 공개 문화 부재") 재검증 결과**: 완전히 균일한 부재는 아니며, "공개 status page" 기준으로는 국내 3개사 전부 미흡하지만 "수치 공시"(카카오페이)와 "회고 공개"(당근마켓)는 각각 부분적으로 존재 — 04_pain_points.md의 PP-4는 **"공개 status page 부재"로 더 정밀하게 재정의**할 수 있음.

## 4. 다음 단계

- [ ] (1)단계 설계 시 우리 프로젝트는 처음부터 공개 status page + 명시적 SLO 수치 + 장애 발생 시 Blameless Postmortem을 함께 묶어 제공 — 해외 3개사와 당근마켓/카카오페이의 장점을 통합한 신뢰성 공시 모델을 목표로 설정.
