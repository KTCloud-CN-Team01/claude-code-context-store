# デプロイ戦略比較

> 참조일: 2026-06-30

---

## 1. 各サービスのデプロイ戦略一覧

```
┌──────────────────────────────────────────────────────────────────────┐
│ デプロイ戦略の種類                                                      │
├──────────────┬────────────────────────────────────────────────────────┤
│ Rolling      │ Pod を少しずつ入れ替え。シンプルだが一時的に新旧混在      │
│ Blue/Green   │ 新旧 2 環境を並走。瞬時切替可能。コスト 2 倍              │
│ Canary       │ 新バージョンを一部（1-10%）トラフィックで検証してから展開  │
│ Feature Flag │ デプロイとリリースを分離。コード変更なしで機能 ON/OFF     │
└──────────────┴────────────────────────────────────────────────────────┘
```

| サービス | 主なデプロイ戦略 | 自動化レベル | ロールバック時間 |
|---------|:---:|:---:|:---:|
| Netflix | Canary（Spinnaker + Kayenta 自動分析）| ◎ フルオート | < 2 分 |
| Spotify | Feature Flag + Canary | ◎ フルオート | < 1 分 |
| Amazon | Canary + Blue/Green | ◎ フルオート | < 2 分 |
| Coupang | Rolling + 推定 Blue/Green | ○ 半自動 | 5〜10 分 |
| Market Kurly | Rolling Update | △ 一部手動 | 10〜30 分 |
| Inflearn | Rolling Update | × 手動確認 | 30 分以上 |
| **本 PJ** | **Canary（Argo Rollouts）+ Feature Flag** | **◎ フルオート** | **< 2 分** |

---

## 2. Netflix Canary デプロイ タイムライン

```
時刻   状態
──────────────────────────────────────────────────────────────
T+0    PR マージ → GitHub Actions CI 開始
T+5    ビルド完了 → Spinnaker パイプライン起動
T+7    Bake: Docker Image 作成
T+10   Canary 1% にデプロイ
T+15   Kayenta が自動メトリクス分析（エラー率/レイテンシ比較）
         ✓ 正常 → 次ステップへ
         ✗ 異常 → 自動ロールバック通知
T+20   10% 展開 → 分析継続
T+30   50% 展開
T+45   100% 展開 → デプロイ完了

参照: https://netflixtechblog.com/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69
```

---

## 3. 本プロジェクトの Argo Rollouts 設定概要

```yaml
# argo-rollout-config.yaml（概念例）
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5     # 5% トラフィックを Canary へ
        - pause: {duration: 5m}
        - analysis:        # 自動メトリクス検証
            templates:
              - templateName: payment-success-rate
        - setWeight: 25
        - pause: {duration: 5m}
        - setWeight: 50
        - pause: {duration: 5m}
        - setWeight: 100

---
# Analysis Template（結済成功率チェック）
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: payment-success-rate
spec:
  metrics:
    - name: payment-success-rate
      provider:
        prometheus:
          query: |
            sum(rate(payment_success_total[5m])) /
            sum(rate(payment_requests_total[5m]))
      successCondition: result >= 0.999
      failureLimit: 1
```

---

## 4. Feature Flag によるデプロイ/リリース分離

```
[デプロイとリリースの分離（Unleash Feature Flag）]

従来（デプロイ = リリース）:
  コード変更 → デプロイ → 全ユーザーに即時適用
  問題: 問題時に全ユーザーが影響を受ける

Feature Flag 導入後（デプロイ ≠ リリース）:
  コード変更 → デプロイ（Flag OFF = 誰にも見えない）
               ↓
               Flag を段階的に ON:
               1% → 5% → 10% → 50% → 100%
               ↓
               問題があれば Flag OFF で即時ロールバック
               （再デプロイ不要 = 数秒で戻せる）

활용 사례（本 PJ での想定）:
  - 새 결제 PG 통합: Flag OFF でコードをデプロイ後
                     1% ユーザーで動作確認 → 段階展開
  - 新機能の A/B テスト: 50% ずつ分割して比較
  - 緊急時の機能 Kill Switch: 障害時に問題機能を即 OFF
```
