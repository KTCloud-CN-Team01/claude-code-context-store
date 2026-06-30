# 06 — SLO・信頼性公示方式比較（海外 vs 国内）

> 참조일: 2026-06-30（選択課題）

---

## 1. SLO / SLA 公示文化の概要比較

```
┌──────────────────────────────────────────────────────────────────���─┐
│ SLO 公示レベルの分類                                                │
├────────────────────────────────��─────────────────────────────────┤
│ Level 3: 公開 SLA + エラーバジェット + ポストモーテム公開           │
│ Level 2: 公開 SLA + Status Page（履歴付き）                        │
│ Level 1: Status Page のみ（公開 SLA なし）                         │
│ Level 0: 非公開 / 実質的に何も公開していない                        │
└────────────────────────────────────────────────────────────────────┘
```

| サービス | SLO 公示レベル | 公開 SLA | Status Page | ポストモーテム公開 |
|---------|:---:|:---:|:---:|:---:|
| Netflix | Level 2 | 非公開 | https://status.netflix.com/ | 一部公開 |
| Spotify | Level 2 | 非公開 | https://status.spotify.com/ | 限定的 |
| Amazon AWS | Level 3 | 公開（99.99%〜99.9%）| https://status.aws.amazon.com/ | 一部公開 |
| Coupang | Level 1 | 非公開 | https://status.coupang.com/ | 未公開 |
| Market Kurly | Level 0〜1 | 非公開 | 不明 | 未公開 |
| Inflearn | Level 0 | 非公開 | なし | なし |

---

## 2. 海外企業の SLO・信頼性公示詳細

### 2-1. Netflix

```
公開情報:
  - Status Page: https://status.netflix.com/
    → 過去 90 日間のインシデント履歴を公開
    → サービス区分: Streaming, Website, Sign-up, Customer Service

  - 公開されていないもの:
    → 数値的 SLA（契約上の保証なし）
    → エラーバジェット消費率
    → 内部 SLO 数値

  - ポストモーテム公開事例:
    → 2012 年 AWS 障害ポストモーテム（Tech Blog 公開）
       https://netflixtechblog.com/post-mortem-of-october-22-2012-aws-degradation-efcee3ab40d5
    → Chaos Engineering 強化のきっかけとして詳細説明

Netflix の信頼性文化:
  - 内部では "Four Nines" (99.99%) を目標に運用（推定）
  - Chaos Engineering を通じた事前検証を重視
  - Error Budget: 消費率ベースでデプロイ判断
```

### 2-2. Amazon (AWS)

```
公開情報（最も透明度が高い）:
  - AWS Service Level Agreements（SLA）を全サービスで公開
    → S3: 99.9% / EC2: 99.99% / RDS Multi-AZ: 99.95%
    → SLA 違反時のサービスクレジット比率まで公開
    参照: https://aws.amazon.com/legal/service-level-agreements/

  - Status Page: https://health.aws.amazon.com/
    → リージョン・AZ・サービス別のリアルタイム状態
    → 過去インシデントの詳細説明（RCA 含む）

  - Postmortem 公開（一部）:
    → AWS の大規模障害は公式でポストモーテムに準ずる説明を発表
    → 例: 2021 年 12 月 US-EAST-1 障害
       https://aws.amazon.com/message/12721/

AWS の信頼性文化:
  - "Corrective Actions" を公式説明で必ず記載
  - Two-Pizza Team ごとに独立した SLO を設定（推定）
```

### 2-3. Spotify

```
公開情報:
  - Status Page: https://status.spotify.com/
    → 主要サービス（API, Web Player, Mobile, Desktop）別状態
    → 90 日間の可用率グラフを公開

  - ポストモーテム文化（Engineering Blog）:
    → "Blameless Culture" を技術記事で積極的に説明
       https://engineering.atspotify.com/2013/06/reliability-and-fallbacks-in-the-netflix-chaos-monkey-case/
    → 障害ではなく「学習の機会」として位置づけ

Spotify の信頼性文化（Squad モデルとの連携）:
  - 各 Squad が自分のサービスの SLO を定義・管理
  - SLO ダッシュボードを全社公開（社内）
  - Error Budget を使い切ったら機能開発より信頼性改善を優先
```

---

## 3. 国内（韓国）企業の SLO・信頼性公示詳細

### 3-1. Coupang

```
公開情報:
  - Status Page: https://status.coupang.com/（存在確認：限定的）
  - 公開 SLA: なし（B2C であり契約形態が異なる）

  - 障害情報の公開方針:
    → SNS（Twitter/X）での非公式告知が中心
    → 大規模障害は公式アプリ内通知
    → 技術的詳細のポストモーテム公開: なし

문제점:
  → 2021 년 이후 복수의 결제/배송 장애가 발생했으나
     기술적 원인 공개 없이 "서비스 복구 완료" 공지만 게시
  → 사용자 신뢰 구축의 기회를 놓침
```

### 3-2. Market Kurly

```
공개 정보:
  - Status Page: 없음（확인 불가）
  - 공개 SLA: 없음

  - 장애 시 공개 방식:
    → 앱 내 공지사항 또는 SNS 공지
    → "서비스 이용이 일시적으로 불편할 수 있습니다" 수준

  - 기술 블로그（helloworld.kurly.com）에서의 간접 공개:
    → 장애 사례를 기술 학습 관점에서 공개（직접 인시던트 보고서는 아님）
    → DB 페일오버 사례를 기술 아티클로 공개
       https://helloworld.kurly.com/blog/database-failover/
    → 이 방식은 "Blameless 포스트모르템 문화의 초기 단계"로 평가
```

### 3-3. Inflearn

```
공개 정보:
  - Status Page: 없음
  - 공개 SLA: 없음

  - 장애 시 공개 방식:
    → 사용자 SNS 신고 후 수동 공지
    → 기술적 원인 미공개

현황 평가:
  → 스타트업 규모로 SLO 공식 정의 자체가 없을 가능성 높음
  → 관측성 기반 자체가 미흡하여 SLO 계측 불가
```

---

## 4. 海外 vs 国内 比較分析

```
比較ポイント                 海外（Netflix/Amazon/Spotify）  国内（Coupang/Kurly/Inflearn）
───────────────────────────────���────────────────────────────���─────────────────────
SLA 数値公開              △〜◎（Amazon は明示）              × ほぼ非公開
Status Page              ◎ 全サービスで運用                △〜× 一部のみ
ポストモーテム公開         ○ 技術ブログで積極公開              × ほぼ未公開
Blameless 文化            ◎ Spotify/Netflix が文化として定着   △ 一部企業で萌芽
Error Budget 運用          ◎ Google SRE 発祥、広く採用         × ほぼ未適用
SLO ダッシュボード          ◎ Grafana 等で全社可視化（推定）    × 社内も未整備の可能性
──────────────────────────────────────────────────────────────────────────────────
```

### 差異の根本原因分析

```
① ビジネス形態の差:
   海外（特に AWS）: SaaS/Cloud で B2B 契約 → SLA が法的義務
   国内 B2C: 消費者向け = 法的 SLA 義務なし → 非公開でも問題なし

② エンジニアリング文化の差:
   海外: "Blameless Postmortem" 文化（Google SRE 本が普及）
   国内: 障害は恥 → 隠す / 最小化する文化が残存

③ 情報共有インセンティブの差:
   海外: ポストモーテム公開 = 技術力のアピール（採用強化）
   国内: 障害情報公開 = リスク・信頼失墜の懸念

④ 組織成熟度:
   海外: SRE 組織が独立してSLO/Error Budget を管理
   국내: DevOps/SRE 역할이 불분명한 경우 많음
```

---

## 5. 本プロジェクトへの示唆

```
推奨アクション:

① SLO の定義と公開（内部から開始）
   → 결제 성공률 ≥ 99.99%、응답시간 P99 ≤ 3초 を内部 Grafana ダッシュボードで可視化
   → 将来的に Status Page（statuspage.io または Atlassian）を外部公開

② Blameless ポストモーテム文化の確立
   → 障害後 24 時間以内に内部 Postmortem 作成を標準化
   → 技術ブログでの公開を検討（採用・技術力アピール）

③ Error Budget の導入
   → 月次 Error Budget 会議（消費率 > 50% でデプロイフリーズ）
   → Grafana SLO ダッシュボードで自動可視化

参照:
  - Google SRE Workbook: https://sre.google/workbook/error-budget-policy/
  - Atlassian Statuspage: https://www.atlassian.com/software/statuspage
  - 토스페이먼츠 포스트모르템 사례: https://blog.toss.im/article/toss-service-incident-postmortem-20220531
```
