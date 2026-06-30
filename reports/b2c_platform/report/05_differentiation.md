# 05 — 差別化戦略：インフラ・観測性・デプロイの差別化ポイント

> 参照日: 2026-06-30

---

## 1. バリュープロポジションキャンバス

```
┌──────────────────────────────────────────────────────────────────────┐
│               Value Proposition Canvas                               │
├────────────────────────────┬─────────────────────────────────────────┤
│    顧客プロフィール         │       バリュープロポジション              │
│   （B2C プラットフォーム    │      （本プロジェクトが提供する価値）      │
│    開発・運用チーム）       │                                         │
├────────────────────────────┼─────────────────────────────────────────┤
│ 【ジョブ（Job）】          │ 【ゲインクリエーター】                    │
│ - 決済を止めずにリリース    │ + 決済 SLO 99.99% の自動検証             │
│ - スパイクに自動対応        │ + カナリアデプロイで無停止リリース        │
│ - 障害を素早く検知・復旧    │ + OpenTelemetry で障害原因を 3 分以内特定 │
│ - MSA 間の整合性を保証     │ + Saga パターンで決済整合性を保証         │
├────────────────────────────┼─────────────────────────────────────────┤
│ 【ペイン（Pain）】         │ 【ペインリリーバー】                      │
│ - PG 障害で全決済停止       │ + PG マルチ化 + Circuit Breaker          │
│ - デプロイで 5xx が出る    │ + Argo Rollouts + Readiness Probe        │
│ - ログを手動で追う          │ + Traces + 構造化ログ + 相関 ID          │
│ - MSA 移行でデータ不整合   │ + Outbox + CDC (Debezium)               │
└────────────────────────────┴─────────────────────────────────────────┘
```

---

## 2. 差別化ポイント 3 本柱

### 差別化① — 決済特化のレジリエンス設計

**課題（PP-01, PP-06）**: 決済スパイク・PG 障害によるカスケード

```
本プロジェクトの解決策:

[アーキテクチャ]
  ┌─────────────────────────────────────────────────────┐
  │  주문 서비스           결제 서비스           PG 레이어  │
  │  OrderService  ──▶   PaymentService  ──▶  PG Adapter│
  │                         │                  /    \    │
  │                      Saga                토스   카카오│
  │                    Orchestrator         Pay   Pay   │
  │                         │                           │
  │                      Kafka（MQ）                    │
  └─────────────────────────────────────────────────────┘

実装要素:
  ① Saga Orchestrator（Temporal / Conductor）
     → 결제 플로우의 각 단계를 트랜잭션으로 관리
     → 실패 시 자동 보상 트랜잭션 실행

  ② PG Adapter + Circuit Breaker（Resilience4j）
     → 토스페이먼츠 장애 감지 → 자동으로 KakaoPay 전환
     → 전환 시간: < 1 초

  ③ KEDA（Kubernetes Event-Driven Autoscaling）
     → Kafka 결제 큐 길이 기반 자동 스케일링
     → 스파이크 시 결제 파드 0 → 50 까지 자동 확장

差別化指標:
  - 決済成功率: > 99.99%（PG 障害時もフェイルオーバーで維持）
  - PG 切替時間: < 1 秒（Circuit Breaker + Auto Failover）
  - スパイク時スケールアウト: < 30 秒（KEDA）
```

参照:
- https://temporal.io/
- https://keda.sh/
- https://resilience4j.readme.io/

---

### 差別化② — エンドツーエンドのオブザーバビリティ

**課題（PP-03）**: 障害検知遅延・Traces 欠如

```
本プロジェクトの観測基盤設計:

[LGTM スタック]
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │  App Pods ──▶ OpenTelemetry Collector            │
  │                    │         │         │         │
  │                  Loki    Prometheus   Tempo       │
  │                  (Logs)  (Metrics)   (Traces)    │
  │                    └─────────┴─────────┘         │
  │                              │                   │
  │                        Grafana Dashboard         │
  │                              │                   │
  │              SLO Alert（AlertManager）            │
  └──────────────────────────────────────────────────┘

実装要素:
  ① OpenTelemetry Auto-Instrumentation
     → Spring Boot / Kotlin アプリに Agent 注入のみで Traces 取得
     → コード変更ゼロでトレーシング完成

  ② 구조화 로그 + 상관 ID（Correlation ID）
     → TraceID を全ログに自動付与
     → Loki で Traces と Logs をクロスリンク

  ③ SLO ベースのアラート（エラーバジェット）
     → 結제 서비스 에러율 > 0.01% で 5 分以内に PagerDuty 通知
     → 誤検知率 < 1%（Composite Alert 条件）

差別化指標:
  - 障害検知時間（MTTD）: < 1 分（SLO アラート）
  - 障害原因特定時間: < 5 分（Traces + 構造化ログ相関）
  - MTTR（平均復旧時間）: < 15 分（Runbook 自動化）
```

参照:
- https://opentelemetry.io/
- https://grafana.com/oss/loki/
- https://grafana.com/oss/tempo/

---

### 差別化③ — ゼロダウンタイム継続デリバリー

**課題（PP-04）**: デプロイ時の 5xx 発生・手動リリース

```
本プロジェクトのデプロイパイプライン:

[GitOps + Progressive Delivery]
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  PR Merge ──▶ GitHub Actions（Build/Test）               │
  │                    │                                     │
  │               Docker Image Push                          │
  │                    │                                     │
  │              ArgoCD（GitOps Sync）                       │
  │                    │                                     │
  │           Argo Rollouts（Progressive Delivery）          │
  │           ┌──────────────────────────────┐              │
  │           │ Step 1: Canary 5%            │              │
  │           │ → 결제 에러율 < 0.1% チェック  │              │
  │           │ Step 2: 25% → 50% → 100%    │              │
  │           │ → 自動メトリクス検証（Kayenta）│              │
  │           │ Step 3: 異常検知 → 自動ロールバック          │
  │           └──────────────────────────────┘              │
  └──────────────────────────────────────────────────────────┘

実装要素:
  ① Argo Rollouts + Prometheus Analysis
     → カナリア段階ごとにエラー率・レイテンシを自動評価
     → 기준 이상 시 자동 롤백（人手不要）

  ② Feature Flag（OpenFeature / Unleash）
     → デプロイとリリースを分離
     → 決済 새 PG 통합은 Feature Flag で段階的 ON

  ③ PodDisruptionBudget + preStop Hook
     → ローリング更新中の処理中決済を graceful に完了させてから Pod 終了

差別化指標:
  - デプロイ時エラー率増加: 0%（ゼロダウンタイム）
  - デプロイ所要時間: < 10 分（Canary 100%到達まで）
  - 自動ロールバック発動時間: < 2 分（異常検知から）
```

参照:
- https://argoproj.github.io/rollouts/
- https://openfeature.dev/
- https://www.getunleash.io/

---

## 3. 差別化ポイントの競合比較

```
差別化ポイント vs. 競合サービス比較

                   Netflix  Spotify  Amazon  Coupang  M.Kurly  Inflearn  【本PJ】
───────────────────────────────────────────────────────────────────────────────────
決済 Saga + Circuit    ◎       ○        ◎       △        ×        ×        ◎
PG マルチ化+自動切替   ◎       ○        ◎       ○        △        ×        ◎
OpenTelemetry標準化   ○       △        △       △        ×        ×        ◎
SLOベースアラート      ◎       ○        ◎       △        ×        ×        ◎
Argo Rollouts Canary  △       △        △       ×        ×        ×        ◎
GitOps (ArgoCD)       ○       ○        △       ○        ○        ×        ◎
Feature Flag統合      ◎       ◎        ◎       △        △        ×        ◎
───────────────────────────────────────────────────────────────────────────────────
（◎=実施中確認, ○=推定実施, △=一部実施, ×=未実施/不明）
```

---

## 4. 実装ロードマップ

```
Phase 1（基盤整備: 1〜2 か月）:
  [✓] EKS クラスタ構築（IaC: Terraform）
  [✓] ArgoCD + GitHub Actions（GitOps パイプライン）
  [✓] OpenTelemetry Collector + LGTM スタック

Phase 2（決済強化: 2〜3 か月）:
  [ ] Saga Orchestrator 導入（Temporal）
  [ ] PG Adapter + Circuit Breaker 実装
  [ ] 결제 SLO 定義 + アラート設定

Phase 3（Progressive Delivery: 3〜4 か月）:
  [ ] Argo Rollouts 導入（Canary デプロイ）
  [ ] Feature Flag 統合（Unleash）
  [ ] カオス実験（LitmusChaos / k6）

Phase 4（成熟化: 4〜6 か月）:
  [ ] Istio サービスメッシュ全展開
  [ ] Chaos Engineering 自動化
  [ ] Cost Optimization（Karpenter）
```

---

## 5. 定量的 SLO 定義（本プロジェクト）

```
Service Level Objectives（目標値）:

┌────────────────────────────────────────────────────────┐
│ SLO ID  │ 対象               │ 目標        │ 計測方法  │
├────────────────────────────────────────────────────────┤
│ SLO-01  │ 결제 성공률          │ ≥ 99.99%    │ Prometheus│
│ SLO-02  │ 결제 응답시간 P99    │ ≤ 3 초      │ Tempo     │
│ SLO-03  │ 주문 서비스 가용성    │ ≥ 99.95%    │ Blackbox  │
│ SLO-04  │ 배포 중 에러율 증가  │ ≤ 0.01%     │ Prometheus│
│ SLO-05  │ MTTD（장애 감지）    │ ≤ 1 분      │ AlertMgr  │
└────────────────────────────────────────────────────────┘

エラーバジェット（月次）:
  SLO-01（99.99%）: 허용 다운타임 = 4.38 분/월
  SLO-03（99.95%）: 허용 다운타임 = 21.9 분/월
```

**次のドキュメント**: `06_slo_reliability.md` → SLO・信頼性公示方式の海外-国内比較
