# Netflix 推定アーキテクチャ（2024 年公開情報ベース）

> 参照日: 2026-06-30  
> 注意: 公開技術ブログ・カンファレンス発表からの推定。実際の構成と異なる可能性あり。

---

## 1. 全体アーキテクチャ概要

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         Netflix アーキテクチャ全体像                           │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  [クライアント層]                                                              │
│  TV / Mobile / Web / Game Console                                            │
│         │                                                                    │
│         ▼                                                                    │
│  [CDN 層: Open Connect]                                                       │
│  ┌────────────────────────────────────────────────────────────┐              │
│  │ Netflix Open Connect Appliance (OCA)                       │              │
│  │ ISP 設置の専用サーバー（コンテンツキャッシュ）                  │              │
│  │ 190 か国 6,000+ ロケーション                                 │              │
│  └────────────────────────────────────────────────────────────┘              │
│         │（コンテンツ配信: 動画バイト列）                                        │
│         │（制御プレーン: API コール）                                            │
│         ▼                                                                    │
│  [API ゲートウェイ層: Zuul]                                                    │
│  ┌────────────────────────────────────────────────────────────┐              │
│  │ Zuul (OSS API Gateway)                                     │              │
│  │ - レート制限 / 認証 / ルーティング                             │              │
│  │ - Groovy フィルターで動的ルーティング                           │              │
│  └────────────────────────────────────────────────────────────┘              │
│         │                                                                    │
│         ▼                                                                    │
│  [マイクロサービス層（700+ サービス）]                                           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ユーザー   │ │コンテンツ  │ │ 推薦     │ │ 決済     │ │ 検索     │          │
│  │サービス  │ │サービス   │ │サービス  │ │サービス  │ │サービス  │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                                     │
│  │ストリーミ │ │ ABR制御  │ │ A/B テスト│                                     │
│  │ングサービス│ │サービス  │ │サービス  │                                     │
│  └──────────┘ └──────────┘ └──────────┘                                     │
│         │                                                                    │
│         ▼                                                                    │
│  [データ層]                                                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                        │
│  │Cassandra │ │ MySQL    │ │EVCache   │ │DynamoDB  │                        │
│  │（視聴履歴）│ │（決済/課金）│ │（L2 Cache）│ │（設定管理）│                        │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                        │
│                                                                              │
│  [コンテナ基盤: Titus]                                                         │
│  AWS EC2 Fleet ─── Titus（内製 ECS 互換 K8s） ─── 内製スケジューラ              │
│                                                                              │
│  [観測基盤]                                                                   │
│  Atlas（内製メトリクス） + Elasticsearch + Zipkin + Vizceral（可視化）           │
│                                                                              │
│  [CD 基盤: Spinnaker]                                                         │
│  GitHub → Jenkins（Build）→ Spinnaker → Titus（デプロイ）                     │
│  └── Kayenta（カナリア分析）→ 自動ロールバック                                   │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 決済アーキテクチャ詳細

```
[Netflix 決済フロー（推定）]

クライアント
    │ HTTPS（TLS 1.3）
    ▼
Zuul (API Gateway)
    │ 認証検証（JWT）
    ▼
決済サービス（Payment Service）
    ├── 決済 DB（MySQL: Aurora）
    ├── 冪等性チェック（Redis: Idempotency-Key）
    │
    ├──▶ Braintree（北米・欧州）
    ├──▶ PayPal（グローバル）
    ├──▶ ローカル PG（190 か国別）
    │       └── Circuit Breaker（Hystrix → Resilience4j）
    │
    └──▶ イベント発行（Apache Kafka）
             └── サブスクリプションサービス（権限付与）
             └── 通知サービス（メール/プッシュ）
             └── 分析サービス（BigQuery）

決済の冪等性保証:
  - Redis に OrderID ベースの Idempotency Key を保存
  - 24 時間 TTL で重複リクエストを検知・除外
  参照: https://netflixtechblog.com/keeping-movies-playable-while-improving-the-payment-stack-6b9de34cb044
```

---

## 3. Chaos Engineering アーキテクチャ

```
[Netflix Simian Army 構成]

Chaos Monkey      → ランダムに EC2 インスタンスを終了
Chaos Gorilla     → AWS Availability Zone を丸ごと落とす
Chaos Kong        → AWS Region を丸ごと落とす
Latency Monkey    → サービス間通信に人工遅延を注入
Conformity Monkey → ベストプラクティス違反のインスタンスを検出
Security Monkey   → セキュリティポリシー違反を検出

実行スケジュール:
  Chaos Monkey: 毎日 業務時間中（エンジニアが対応できる時間）に自動実行
  Chaos Kong:   定期的な Game Day（計画的な大規模障害訓練）

参照:
  https://netflixtechblog.com/the-netflix-simian-army-16e57fbab116
  https://github.com/Netflix/SimianArmy
```

---

## 4. サービスディスカバリ & ロードバランシング

```
[Eureka + Ribbon 構成（旧）→ Envoy 移行中]

旧構成:
  マイクロサービス → Eureka（サービスレジストリ）に自己登録
  呼び出し側: Ribbon（クライアントサイドロードバランシング）で取得・分散

新構成（推定）:
  Envoy サイドカー → xDS API → サービスディスカバリ
  Istio Control Plane → Envoy 設定を動的配布
  参照: https://netflixtechblog.com/open-sourcing-zuul-2-82ea476cb2b3
```

---

## 5. 技術的強み・弱みまとめ

| 項目 | 強み | 弱み |
|-----|------|------|
| スケール | 世界 190 か国の同時配信を維持 | — |
| カオス | 本番環境での常時カオス = 最高の耐障害性 | — |
| CD | Kayenta で自動カナリア分析 | — |
| 内製依存 | — | Titus/Atlas など内製多すぎて保守コスト高 |
| サービスメッシュ | — | Istio 標準化が遅く独自実装が多い |

**参照リンク**:
- https://netflixtechblog.com/
- https://github.com/Netflix (OSS 一覧)
- https://www.infoq.com/articles/netflix-migration-cloud/
