# Spotify 推定アーキテクチャ（2024 年公開情報ベース）

> 참조일: 2026-06-30  
> 注意: 公開技術ブログ・カンファレンス発表からの推定。

---

## 1. 全体アーキテクチャ概要

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         Spotify 推定アーキテクチャ                             │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  [クライアント層]                                                              │
│  Desktop（Electron）/ Mobile（iOS/Android）/ Web Player                      │
│         │                                                                    │
│         ▼                                                                    │
│  [CDN 層]                                                                    │
│  Google Cloud CDN + Akamai（楽曲ファイル配信）                                 │
│         │                                                                    │
│         ▼                                                                    │
│  [API 層]                                                                    │
│  REST API（外部）+ gRPC（内部サービス間）                                       │
│  └── OAuth2 / PKCE 認証（Spotify Accounts Service）                          │
│         │                                                                    │
│         ▼                                                                    │
│  [マイクロサービス層（Squad 単位で管理）]                                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ 再生     │ │ 検索     │ │ 推薦     │ │ 課金     │ │ ポッドキャ│          │
│  │ Service │ │ Service │ │ Service │ │ Service │ │ スト     │          │
│  │ (Go)    │ │ (Java)  │ │(Python  │ │(Kotlin) │ │ Service │          │
│  │         │ │         │ │ +ML)    │ │         │ │         │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│  ┌──────────┐ ┌──────────┐                                                   │
│  │ プレイリ │ │ ソーシャ │                                                   │
│  │ スト    │ │ ル       │                                                   │
│  │ Service │ │ Service │                                                   │
│  └──────────┘ └──────────┘                                                   │
│         │                                                                    │
│         ▼                                                                    │
│  [データ層]                                                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                        │
│  │Cassandra │ │PostgreSQL│ │ BigQuery │ │   GCS    │                        │
│  │（再生履歴）│ │（プレイリ │ │（分析・ML）│ │（楽曲ファ │                        │
│  │          │ │ スト）   │ │          │ │ イル）   │                        │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                        │
│                                                                              │
│  [コンテナ基盤: GKE]                                                           │
│  Google Kubernetes Engine（2016 年 GCP 移行完了）                              │
│  参照: https://engineering.atspotify.com/2023/10/declarative-kubernetes-cluster-management-at-spotify/
│                                                                              │
│  [IDP: Backstage]（OSS 公開）                                                 │
│  → 全 Squad が使う Internal Developer Portal                                  │
│  → サービスカタログ / CI/CD テンプレート / ドキュメント統合                       │
│  参照: https://backstage.io/                                                  │
│                                                                              │
│  [CI/CD]                                                                     │
│  GitHub Actions → GCR（Container Registry）→ Helios（CD）→ GKE              │
│  + Feature Flag（LaunchDarkly）                                               │
│                                                                              │
│  [観測基盤]                                                                   │
│  Prometheus + Heroic（内製 Metrics Backend）+ Grafana                         │
│  + Stackdriver Logging                                                       │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Squad/Tribe 組織モデルとアーキテクチャの対応

```
[Spotify Squad Model]

Company
└── Tribe（50-150 人）
    ├── Squad（6-12 人）— 独立したマイクロサービス 1〜数個を所有
    │   ├── Product Manager
    │   ├── Engineers（Backend/Frontend/Mobile）
    │   ├── Designer
    │   └── Data Scientist
    ├── Chapter（同職種の横断コミュニティ）
    └── Guild（興味関心で集まる任意コミュニティ）

アーキテクチャとの対応:
  各 Squad が：
    - 自サービスの SLO を定義・管理
    - 自サービスのデプロイを独立して実行
    - 自サービスの On-call を担当
    - Backstage でサービスを登録・管理

参照: https://engineering.atspotify.com/2020/04/spotifys-shift-to-a-tribe-model/
```

---

## 3. Freemium 課金アーキテクチャ

```
[Spotify Freemium 決済フロー]

無料ユーザー（Free Tier）
    │ 広告挿入再生（Ad Insertion Service）
    │ プレミアム誘導（Upsell Trigger）
    ▼
プレミアム登録フロー
    ├── Stripe（主要）+ ローカル PG（国別）
    ├── Apple Pay / Google Pay
    └── 家族プラン / 学割プランの価格計算エンジン

月次一括課金バッチ（Billing Service）:
    ├── 毎月 更新日に全会員の決済を処理
    ├── 失敗時: Exponential Backoff リトライ（3 回）
    ├── 最終失敗: Dunning（督促）メール → Free 降格
    └── 課金ジャーナル → BigQuery（会計・分析）

Freemium 特有の課題:
  - 試用期間終了後の自動課金 → ユーザークレームが多い
  - 国別価格差（PPP ベース）の実装
  - プロモーションコード管理の複雑性
  参照: https://engineering.atspotify.com/2022/01/how-we-use-golden-signals-to-define-and-monitor-slos/
```

---

## 4. 技術的強み・弱みまとめ

| 項目 | 強み | 弱み |
|-----|------|------|
| 組織 | Squad Model = CN 組織の教科書 | — |
| IDP | Backstage OSS = 開発者体験統一 | — |
| GCP 移行 | 完全クラウド化・コスト最適化 | — |
| Freemium | 複雑な課金モデルを大規模実装 | 月次バッチで障害リスク集中 |
| カオス | 推定 Level 3、本番一部実施 | Netflix ほど体系化されていない |

**参照リンク**:
- https://engineering.atspotify.com/
- https://backstage.io/
- https://github.com/spotify
