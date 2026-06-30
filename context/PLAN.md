# PLAN.md — 과제(0) 시장·서비스 비교분석 & 클라우드 네이티브 차별화 도출

> 작성일: 2026-06-30 | 브랜치: jyupk | 담당: jyupk

---

## 1. 프로젝트 개요

본 과제는 "선정한 도메인에서 해외·국내 대표 서비스"를 클라우드 네이티브 관점에서 비교분석하고,
우리 프로젝트가 해결할 페인포인트와 차별화 포인트를 정의하는 것을 목적으로 한다.

---

## 2. 선정 도메인

**e-Commerce 결제·콘텐츠 플랫폼**

전자상거래(오픈마켓·C2C 마켓플레이스)와 간편결제를 핵심으로 하는 B2C 디지털 서비스군을 대상으로 한다.
글로벌 이커머스/유통·국내 오픈마켓/C2C 마켓플레이스·국내 간편결제 3개 서브도메인을 횡단해 비교한다.

선정 근거 상세는 `02_service_selection.md`를 참조.

| 구분 | 서비스 | 국가 | 서브도메인 | 시장 지위 |
|------|---------|----|-----------|-----------:|
| 해외① | Shopify | 🇺🇸 미국 | 이커머스 SaaS | 美 SW시장 ~29% 1위 |
| 해외② | Walmart | 🇺🇸 미국 | 유통/옴니채널 | 美 이커머스 2위 |
| 해외③ | Alibaba | 🇨🇳 중국 | 이커머스 | 中 GMV ~32% 1위(하락세) |
| 국내① | 11번가 | 🇰🇷 한국 | 오픈마켓 | ~12~13% 3~4위 |
| 국내② | 당근마켓 | 🇰🇷 한국 | C2C 마켓플레이스 | 중고거래 MAU 1위 |
| 국내③ | 카카오페이 | 🇰🇷 한국 | 간편결제 | 25.1% 2위 |

> 참고: Team01(black)은 Netflix/Spotify/Amazon(해외) + Coupang/Market Kurly/Inflearn(국내)를 선정 — 본 분석은 동일 도메인 카테고리 안에서 기업 구성을 겹치지 않게 구성.

---

## 3. 산출물 목록

```
ecommerce_payment_platform/
├── report/
│   ├── 00_executive_summary.md       # 에그제큐티브 서머리
│   ├── 01_domain_overview.md         # 도메인 시장 개관 + Porter 5 Forces
│   ├── 02_service_selection.md       # 서비스 선정 근거 + 기술스택 상세
│   ├── 03_comparison_matrix.md       # 클라우드 네이티브 비교 매트릭스
│   ├── 04_pain_points.md             # 페인포인트 6건 (MECE+피쉬본)
│   ├── 05_differentiation.md         # 차별화 전략 (VP Canvas)
│   ├── 06_slo_reliability.md         # SLO·신뢰성 공시 비교 (선택)
│   ├── 07_chaos_engineering.md       # 카오스 엔지니어링 비교 (선택)
│   └── 08_pain_point_validation.md   # 컨퍼런스 발표 교차검증 (선택)
├── architectures/
│   ├── shopify_arch.md / walmart_arch.md / alibaba_arch.md
│   ├── 11st_arch.md / daangn_arch.md / kakaopay_arch.md
│   └── our_target_arch.md            # 본 프로젝트 목표 아키텍처
└── diagrams/
    ├── comparison_radar.md           # 5축 비교 레이더 차트(텍스트 형식)
    ├── pain_point_map.md             # 페인포인트 피쉬본 다이어그램
    ├── deployment_strategies.md      # 배포 전략 비교
    └── differentiation_canvas.md     # Value Proposition Canvas 시각화
```

---

## 4. 작업 단계

| 단계 | 내용 | 주요 산출물 | 상태 |
|------|------|---------|------|
| Phase 0 | 도메인·서비스 선정 | 02_service_selection.md | ✅ 완료 |
| Phase 1 | 5축 비교 매트릭스 + 아키텍처 정리 | 03_, architectures/ | ✅ 완료 |
| Phase 2 | 페인포인트 도출 | 04_, pain_point_map.md | ✅ 완료 |
| Phase 3 | 차별화 전략 정의 | 05_, differentiation_canvas.md | ✅ 완료 |
| Phase 4 | 선택 과제(SLO·카오스·검증) | 06_, 07_, 08_ | ✅ 완료 |
| Phase 5 | 도메인 개관 & 서머리 통합 | 00_, 01_ | ✅ 완료 |
| Phase 6 | 구조 정리 (black 브랜치와 폴더 구조 통일) | 전체 재구성 | ✅ 완료 |

---

## 5. 적용 비즈니스 프레임워크

| 프레임워크 | 적용 위치 | 목적 |
|-------------|---------|------|
| MECE | 페인포인트 분류(04), 비교축 설계(03) | 누락·중복 없는 분류 |
| 피쉬본(이시카와) 다이어그램 | 페인포인트 근본원인 분석(04) | 인과관계 정리 |
| 비교 매트릭스 | 클라우드 네이티브 기술 비교(03) | 횡단 비교 시각화 |
| Value Proposition Canvas | 차별화 정의(05) | 고객 가치의 언어화 |
| Porter의 5 Forces | 시장 구조 분석(01) | 경쟁 환경 구조화 |

---

## 6. 토큰·세션 효율화 방침

- 문서를 **소단위 청크**(1파일 ≦ 400행 내외)로 분할
- 도표는 텍스트/ASCII 형식(외부 렌더링 불필요, 로컬 완결)
- 각 단계 완료 후 `git commit`으로 체크포인트 생성
- **모든 인용 URL은 실제 WebSearch/WebFetch로 검증된 것만 사용** — 가공·추정 URL 절대 금지(타 팀 레포에서 의심스러운 인용이 발견된 사례를 반면교사로 삼음). 상세 원칙은 `PROCESS.md` 참조.

---

## 7. 주요 참조 정보원

| 서비스/조직 | URL |
|--------------|-----|
| Shopify Engineering | https://shopify.engineering/ |
| Walmart Global Tech | https://medium.com/walmartglobaltech |
| Alibaba Cloud Blog | https://www.alibabacloud.com/blog |
| 11번가 Tech Blog | https://11st-tech.github.io/ |
| 당근마켓 Tech Blog | https://medium.com/daangn |
| 카카오페이 Tech Blog | https://tech.kakaopay.com/ |
| CNCF Landscape | https://landscape.cncf.io/ |
