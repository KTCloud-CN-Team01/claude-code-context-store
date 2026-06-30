# 差別化キャンバス（バリュープロポジションキャンバス 詳細版）

> 참조일: 2026-06-30

---

## 1. バリュープロポジションキャンバス全体

```
┌────────────────────────────────────────────────────────────────────────────┐
│                    Value Proposition Canvas                                │
├────────────────────────────────────┬───────────────────────────────────────┤
│        顧客プロフィール              │         バリュープロポジション           │
│    （B2C プラットフォーム運用チーム）  │       （本プロジェクトが提供する価値）    │
├────────────────────────────────────┼───────────────────────────────────────┤
│ 【Job-to-be-Done（やりたいこと）】  │ 【Gain Creators（嬉しくなること）】     │
│                                    │                                       │
│ ① 決済を 24/365 止めずに動かす     │ + 결제 성공률 99.99% を自動保証         │
│ ② スパイク時も自動スケール           │ + スパイク時 30 秒以内で自動スケール    │
│ ③ 新機能を安全に素早くリリース       │ + Canary デプロイで 0% ダウンタイム    │
│ ④ 障害を素早く検知・原因特定        │ + MTTD < 1 分、MTTR < 15 分          │
│ ⑤ MSA 間の決済整合性を保証         │ + Saga パターンで二重決済 0 件         │
│                                    │ + PG 障害時 1 秒以内に自動切替         │
├────────────────────────────────────┼───────────────────────────────────────┤
│ 【Pain（困っていること）】           │ 【Pain Relievers（課題を解決）】        │
│                                    │                                       │
│ - セール時に決済サーバーがダウン     │ + KEDA（Kafka ベース HPA）            │
│ - PG 障害で全決済が止まる          │ + PG Adapter + Circuit Breaker        │
│ - デプロイ中に 5xx が出る          │ + Argo Rollouts + PDB                 │
│ - ログを手動追跡（何時間も）        │ + OpenTelemetry Traces 自動計装        │
│ - 注文と決済情報が不一致            │ + Temporal Saga + Outbox CDC          │
│ - サービスメッシュがなく障害が伝播   │ + Istio + Envoy サイドカー             │
└────────────────────────────────────┴───────────────────────────────────────┘
```

---

## 2. 競合との差別化マトリクス

```
機能 / 特性                      本PJ   Coupang  M.Kurly  Inflearn
──────────────────────────────────────────────────────────────────
決済 Saga オーケストレーション     ◎       △        ×        ×
PG マルチ化 + 自動フェイルオーバー  ◎       ○        △        ×
OpenTelemetry 標準 Traces        ◎       △        ×        ×
SLO ベースアラート（エラーバジェット）◎      △        ×        ×
Argo Rollouts Canary デプロイ     ◎       ×        ×        ×
GitOps（ArgoCD）                  ◎       ○        ○        ×
Istio サービスメッシュ（mTLS）     ◎       △        ×        ×
Feature Flag 統合                 ◎       △        △        ×
カオスエンジニアリング（LitmusChaos）◎     △        ×        ×
IaC 徹底（Terraform）             ◎       ○        ○        △
──────────────────────────────────────────────────────────────────
実装数（/10）                     10       5        3        0
```

---

## 3. 差別化を支える技術スタック全体図

```
[本プロジェクト 技術スタック全体]

言語・FW:     Kotlin + Spring Boot / TypeScript + React
コンテナ:      AWS EKS + Karpenter（ノードコスト最適化）
サービスメッシュ: Istio + Envoy（mTLS + Circuit Breaker）
CI/CD:         GitHub Actions → ECR → ArgoCD → Argo Rollouts
Feature Flag:  Unleash（OSS）
Saga:          Temporal（Workflow Engine）
CDC:           Debezium → Kafka（MSK）
スケーリング:   HPA + KEDA（Kafka メッセージ数ベース）
オブザーバビリティ:
  Metrics → Prometheus + Grafana
  Logs    → Loki（構造化 JSON + TraceID）
  Traces  → Tempo + OpenTelemetry Collector
  Alert   → AlertManager（SLO エラーバジェットベース）
カオス:        LitmusChaos（Phase 2 から）
IaC:           Terraform（全リソース）
CDN:           CloudFront + Signed URL（コンテンツ保護）
決済 PG:       토스페이먼츠（Primary）+ KakaoPay + NaverPay（Fallback）
```

---

## 4. ROI 推定（定性的）

```
投資対効果の試算（定性的）:

Before（現状 = Inflearn レベル）:
  - 月次決済障害: 2〜4 回（推定）
  - 1 回の障害による損失: 数百万〜数千万ウォン（規模依存）
  - MTTR: 30〜120 分
  - デプロイ頻度: 数回/週（手動確認）

After（本プロジェクト適用後）:
  - 決済障害: 月 0〜1 回（PG 自動切替により最小化）
  - MTTR: < 15 分（Traces による迅速原因特定）
  - デプロイ頻度: 数回/日（自動カナリアで安全に増加）
  - エンジニア On-call 負荷: 70% 削減（自動検知・自動ロールバック）

投資コスト（月次推定）:
  - Istio: OSS（無料）+ 運用工数
  - Temporal: OSS（無料） or Cloud $200〜
  - LGTM スタック: OSS（無料）or Grafana Cloud $100〜
  - Argo Rollouts: OSS（無料）
  - 총 인프라 추가 비용: 月 $500〜$1,000 程度
```
