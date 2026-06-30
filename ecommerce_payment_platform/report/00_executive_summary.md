# 00 — 에그제큐티브 서머리

> 본 문서는 [01_domain_overview.md](./01_domain_overview.md) ~ [08_pain_point_validation.md](./08_pain_point_validation.md) 전체 리포트의 핵심을 통합한 요약이다.

---

## 1. 분석 개요

| 항목 | 내용 |
|---|---|
| 분석 도메인 | e-Commerce 결제·콘텐츠 플랫폼 |
| 대상 서비스 | 해외 3사(Shopify, Walmart, Alibaba) / 국내 3사(11번가, 당근마켓, 카카오페이) |
| 비교 축 | 아키텍처 / CI·CD / 관찰성 / 복원력 / 서비스 메시 (5축, MECE) |
| 페인포인트 | 6건 식별 (PP-1 ~ PP-6) |
| 컨퍼런스 발표 교차검증 | 1건 직접 검증(PP-5), 2건 부재 자체가 보강(PP-3·PP-4) |
| 차별화 포인트 | 메시 도입 ROI 정량화 + 재현 가능한 카오스 실험 + Blameless Postmortem 통합 파이프라인 |

---

## 2. 주요 발견사항

### 2-1. 축별 성숙도 순위 (정성 평가, 근거: [03_comparison_matrix.md](./03_comparison_matrix.md))

```
[서비스 메시 채택 적극성]
Alibaba(3중 전략) > Walmart(Istio+Linkerd) > 당근마켓(일부 도메인)
  > Shopify(평가 후 보류) > 11번가 ≈ 카카오페이(미도입 추정)

[카오스 엔지니어링 성숙도]
Alibaba(ChaosBlade, CNCF) ≈ 11번가(Chaos Mesh+OSS 기여) ≈ Walmart(Concord+Gremlin)
  > Shopify(BFCM 정례화) > 당근마켓(사후 대응형) > 카카오페이(미확인)

[관찰성 투명성/공개 수준]
Shopify(OSS 스택 풀공개) ≈ 당근마켓(장애 디테일 공개) > Alibaba(ARMS 상품화)
  > 11번가/카카오페이(SIEM·로그플랫폼은 공개, 트레이싱 활용 불명) > Walmart(세부 비공개)

[배포 파이프라인 정교함]
Shopify(Canary 자동화) ≈ 카카오페이(Rolling/Canary/BlueGreen 선택형)
  > 당근마켓(GitOps) > Alibaba(全链路压测) > 11번가(도구체인 비공개) > Walmart(최신정보 부족)
```

### 2-2. 해외 vs 국내 — 단일 축이 아닌 분산된 격차

블랙 브랜치(Team01) 분석과 달리, 본 분석 대상 6개사는 "해외가 전축에서 우위"인 균일한 패턴을 보이지 않는다.

| 비교 축 | 격차 양상 |
|---|---|
| 서비스 메시 | 해외 우위 뚜렷 (Alibaba·Walmart 적극, 11번가·카카오페이 미도입 추정) |
| 카오스 엔지니어링 | **국내 11번가가 해외 Shopify보다 앞섬**(OSS 컨트리뷰션까지 도달) — 단일 방향 격차 아님 |
| 관찰성 투명성 | **국내 당근마켓이 6개사 중 최상위권**(장애 디테일 공개) — 국내가 항상 뒤처지지 않음 |
| 신뢰성 공시(SLO) | 해외 3사는 공개 status page 보유, 국내는 부재 — 단 카카오페이(SLO 수치 공시)·당근마켓(회고 공개)은 각자 다른 방식으로 부분 보완 ([06_slo_reliability.md](./06_slo_reliability.md)) |

**시사점**: "국내가 전반적으로 뒤처진다"는 단순화 대신, **결제 도메인(카카오페이)이 보수적**이라는 더 정밀한 패턴이 식별됨([07_chaos_engineering.md](./07_chaos_engineering.md)).

---

## 3. 식별된 페인포인트 6건

| 카테고리 | PP | 내용 | 컨퍼런스 검증 |
|---|---|---|---|
| A. 기술/아키텍처 | PP-1 | 서비스 메시 도입 ROI 판단 근거 부재 | ⚠️ 출처가 블로그였음 재확인 |
| A. 기술/아키텍처 | PP-2 | 결제 분산 트랜잭션 Unknown 상태 수작업 의존 | 미확인 |
| B. 프로세스/거버넌스 | PP-3 | 결제 도메인은 카오스 엔지니어링 실시 자체가 보수적 | ✅ 발표 부재가 가설 강화 |
| C. 조직문화/공시 | PP-4 | 국내 기업 포스트모템 공개 문화 부재 | ✅ 발표 부재가 가설 강화 |
| D. 인프라/환경 | PP-5 | 하이브리드 인프라 과도기 기술부채 | ✅ Spring Camp 2018 직접 검증 |
| A. 기술/아키텍처 | PP-6 | "MSA = 정답"이라는 전제의 위험 | ⚠️ 발표자 소속 불명확 |

상세 근거: [04_pain_points.md](./04_pain_points.md) (MECE+피쉬본), [08_pain_point_validation.md](./08_pain_point_validation.md) (컨퍼런스 교차검증)

---

## 4. 본 프로젝트의 차별화 전략 — 단계별 매핑

```
┌──────────────────────────────────────────────────────────────────────┐
│         과제 단계별로 6개 페인포인트를 직접 해소하는 산출물 설계            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  (1) 도메인·서비스 분리                                                │
│      PP-6 해소: Bounded Context를 "잘게 쪼개기"가 아닌                  │
│      결제 일관성 경계 단위로 최소화(Shopify·당근페이 반례를 설계 근거화) │
│                                                                      │
│  (2) 서비스 메시 통합                                                  │
│      PP-1 해소: 메시 도입 전/후 CPU·레이턴시·에러율 직접 측정           │
│      (Shopify의 정량 검증 문화를 우리 환경에 재현)                      │
│                                                                      │
│  (3) 복원력·관찰성·케이스 스터디                                        │
│      PP-2 해소: Saga + 명시적 서킷 브레이커로 Unknown 상태 자동 보상율 측정│
│      PP-3 해소: 카오스 실험을 사전설계→실시→결과공개까지 의무화          │
│      PP-4 해소: Blameless Postmortem 템플릿 표준 채택                  │
│      PP-5 해소: 단일 클러스터·단일 오케스트레이터로 과도기 부채 원천 차단 │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

상세: [05_differentiation.md](./05_differentiation.md) (Value Proposition Canvas)

---

## 5. 경쟁 비교에서 본 본 프로젝트의 포지션

```
6개 선도 기업 중 어느 한 곳도
"메시 도입 ROI 정량화" + "재현 가능한 카오스 실험" + "블레임리스 포스트모템"
을 동시에 갖추지 못함:

  Shopify   ✅ 메시 ROI 정량화(블로그)     ❌ 카오스 결과 비공개      ❌ 포스트모템 비공개
  11번가    ❌ 메시 미도입                ✅ 카오스 OSS 컨트리뷰션   ❌ 포스트모템 비공개
  당근마켓  △ 일부 도메인만               △ 사후 대응형             ✅ 장애 회고 공개
  ─────────────────────────────────────────────────────────────────
  본 프로젝트 ✅ 3개 모두 통합 시연 목표 (단계 (2)+(3)에서 산출물화)
```

세 회사 각각이 부분적으로만 잘하는 것을 **하나의 프로젝트 안에서 통합 시연**하는 것이 핵심 차별화 포인트.

---

## 6. 문서 구성

```
ecommerce_payment_platform/
├── report/
│   ├── 00_executive_summary.md   ← 본 문서
│   ├── 01_domain_overview.md     ← 도메인 시장 개관 + Porter 5 Forces
│   ├── 02_service_selection.md   ← 선정 근거 + 6개사 기술스택 상세
│   ├── 03_comparison_matrix.md   ← 5축 비교 매트릭스 + 강점/약점
│   ├── 04_pain_points.md         ← MECE + 피쉬본 페인포인트 6건
│   ├── 05_differentiation.md     ← Value Proposition Canvas 차별화 전략
│   ├── 06_slo_reliability.md     ← SLO·신뢰성 공시 비교 (선택)
│   ├── 07_chaos_engineering.md   ← 카오스 엔지니어링 비교 (선택)
│   └── 08_pain_point_validation.md ← 컨퍼런스 발표 교차검증 (선택)
├── architectures/
│   ├── shopify_arch.md / walmart_arch.md / alibaba_arch.md
│   ├── 11st_arch.md / daangn_arch.md / kakaopay_arch.md
│   └── our_target_arch.md        ← 본 프로젝트 목표 아키텍처
└── diagrams/
    ├── comparison_radar.md       ← 5축 비교 레이더 차트
    ├── pain_point_map.md         ← 페인포인트 피쉬본 다이어그램
    ├── deployment_strategies.md  ← 배포 전략 비교
    └── differentiation_canvas.md ← Value Proposition Canvas 시각화
```

---

## 7. 결론

```
e-Commerce 결제·콘텐츠 플랫폼 도메인에서, 6개 선도 기업을 비교한 결과
"해외가 전반적으로 앞선다"는 단순한 도식은 성립하지 않는다.

  - 메시 채택은 해외(Alibaba·Walmart)가 앞서지만
  - 카오스 엔지니어링은 국내 11번가가 Shopify보다 앞서고
  - 관찰성 투명성은 국내 당근마켓이 최상위권이다

다만 결제 도메인 본체(카카오페이)는 일관되게 보수적인 패턴을 보이며,
이는 "해외 vs 국내"가 아니라 "결제 vs 비결제 도메인"의 리스크 회피
성향 차이로 더 정밀하게 설명된다.

본 프로젝트는 이 분산된 강점들 — Shopify의 정량적 비용 검증 문화,
11번가의 OSS 기여형 카오스 엔지니어링, 당근마켓의 투명한 장애 회고 —
을 하나의 파이프라인으로 통합 시연하는 것을 목표로 한다.
```
