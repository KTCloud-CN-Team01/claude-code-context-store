# 02 — サービス選定根拠（詳細）

> 参照日: 2026-06-30

---

## 1. 選定サービス一覧

| # | サービス | 区分 | 母国 | 選定スコア |
|---|---------|------|------|-----------|
| 1 | Netflix | 海外① | 米国 | 20/20 |
| 2 | YouTube (Google) | 海外② | 米国 | 19/20 |
| 3 | Disney+ | 海外③ | 米国 | 15/20 |
| 4 | AbemaTV (CyberAgent) | 国内① | 日本 | 20/20 |
| 5 | U-NEXT | 国内② | 日本 | 14/20 |
| 6 | dアニメストア (NTTドコモ) | 国内③ | 日本 | 12/20 |

---

## 2. 海外サービス詳細選定根拠

### 2-1. Netflix

**基本情報**

| 項目 | 内容 |
|-----|------|
| 設立 | 1997年（ストリーミング開始: 2007年）|
| 本社 | カリフォルニア州ロスガトス |
| 有料会員数 | 約2億6,900万人（2024 Q1）|
| クラウド基盤 | AWS（2016年にデータセンター完全撤退）|
| 公式技術ブログ | https://netflixtechblog.com/ |

**選定根拠**

```
① クラウドネイティブの教科書的存在
   - 2008年にAWSへの移行を開始、2016年に完全クラウド化
   - Chaos Monkey（カオスエンジニアリングツール）を発明・OSSとして公開
   - Spinnaker（多クラウドCD）を開発・OSSとして公開
   参照: https://netflixtechblog.com/completing-the-netflix-cloud-migration-783e1ea8c3c0

② 技術情報の豊富さ
   - Tech Blogに1,000記事超の詳細な技術記事
   - AWS re:Invent等での毎年の詳細発表
   - OSS貢献: Hystrix(サーキットブレーカー), Eureka(サービスディスカバリ) 等

③ 障害報告の透明性
   - 公開ポストモーテムが複数存在
   - Status Page: https://status.netflix.com/ で過去履歴確認可能
```

**主要技術スタック（公開情報から）**

```
言語: Java, Python, JavaScript (Node.js)
コンテナ: Titus（内製 + Mesos → ECS互換）
サービスメッシュ: 独自実装 → Zuul(APIゲートウェイ)
CI/CD: Spinnaker（自社開発OSS）
観測性: Atlas（内製メトリクス）, Vizceral（トラフィック可視化）
データ: Cassandra, EVCache, MySQL
CDN: Open Connect（独自CDN）
```
参照: https://netflixtechblog.com/netflix-oss-and-spring-boot-coming-full-circle-4855947713a0

---

### 2-2. YouTube (Google)

**基本情報**

| 項目 | 内容 |
|-----|------|
| 設立 | 2005年（Google買収: 2006年）|
| 月間ユーザー | 25億人以上 |
| 動画アップロード | 1分間に500時間分 |
| クラウド基盤 | Google Cloud Platform（完全自社）|
| 技術情報源 | https://youtube-eng.googleblog.com/ |

**選定根拠**

```
① Google SREの実装例
   - 「SRE本」（Google Site Reliability Engineering）の実践の場
   - Error Budget・SLO概念の実装先進事例
   参照: https://sre.google/books/

② スケールの極致
   - 25億MAU、秒単位で数万件の同時アップロード処理
   - 地球規模でのレイテンシ最小化の実装
   
③ 技術革新の発信源
   - VP9/AV1コーデック（帯域幅削減技術）の開発・標準化
   - ABRアルゴリズム(WISH)の開発
   参照: https://research.google/pubs/pub40807/
```

**主要技術スタック（公開情報から）**

```
言語: C++, Python, Go, Java
インフラ: Borg（Kubernetes前身）→ Google Kubernetes Engine
データ: Spanner, Bigtable, Colossus
CDN: Google Global Cache（ISP設置）
ネットワーク: Jupiter（独自スイッチ）, B4（SDNバックボーン）
観測性: Monarch（内製メトリクス）, Dapper（分散トレーシング）
```
参照: https://research.google/pubs/pub43438/

---

### 2-3. Disney+

**基本情報**

| 項目 | 内容 |
|-----|------|
| ローンチ | 2019年11月 |
| 会員数 | 約1億3,000万人（2024）|
| クラウド基盤 | AWS + Azure |
| 開発元 | Disney Streaming（旧BAMTech）|
| 技術情報源 | https://medium.com/disney-streaming |

**選定根拠**

```
① モノリス→MSA移行の生きた事例
   - BAMTech（ESPN+等の元基盤）をMSAに再設計
   - ローンチ時の大規模障害（2019年11月）とその後の改善が公開
   参照: https://medium.com/disney-streaming/why-disney-chose-graphql-for-disney-37c23a6ae6a6

② GraphQL採用の先進事例
   - フロントエンド-バックエンド間にGraphQL連邦（Apollo Federation）を採用
   - 2021年のApolon Federation採用の技術ブログが詳細
   参照: https://medium.com/disney-streaming/open-source-contributions-from-disney-streaming-services-c1ceaaec26ec

③ 大規模スパイクへの対応
   - ローンチ時: 1,000万件以上の同時接続でサービス断
   - その後の改善プロセスが詳細に公開 → ペインポイント事例として最適
```

**主要技術スタック（公開情報から）**

```
言語: Go, Kotlin, Python
API層: GraphQL + Apollo Federation
コンテナ: Kubernetes (EKS)
CI/CD: Spinnaker（Netflixから採用）
観測性: Grafana, Datadog
CDN: Akamai + CloudFront
認証: OAuth2 / PKCE
```

---

## 3. 国内サービス詳細選定根拠

### 3-1. AbemaTV (CyberAgent)

**基本情報**

| 項目 | 内容 |
|-----|------|
| サービス開始 | 2016年4月 |
| 月間ユーザー | 約2,000万人（2024年）|
| クラウド基盤 | GCP + AWS（マルチクラウド）|
| 親会社 | CyberAgent |
| 技術ブログ | https://developers.cyberagent.co.jp/blog/ |
| GitHub | https://github.com/CyberAgent |

**選定根拠**

```
① 国内最大の技術情報公開
   - CyberAgent Developer Blogに200記事以上のインフラ・SRE記事
   - CloudNative Days Japan等での積極的な発表

② Kubernetes大規模運用の国内先駆者
   - 2017年よりKubernetes本番利用
   - カスタムコントローラ・Operatorの開発・公開
   参照: https://developers.cyberagent.co.jp/blog/archives/14011/

③ ライブ配信という最難関の技術課題
   - スポーツライブ配信での超大規模同時接続
   - 2019 ラグビーW杯: 過去最大トラフィックへの対応事例公開
   参照: https://developers.cyberagent.co.jp/blog/archives/22141/
```

**主要技術スタック（公開情報から）**

```
言語: Go（バックエンド主力）, Swift/Kotlin（モバイル）
コンテナ: GKE（Google Kubernetes Engine）
CI/CD: ArgoCD + GitHub Actions
サービスメッシュ: Istio（2021年より本番適用）
観測性: Stackdriver + Datadog + 内製ダッシュボード
データ: Cloud Spanner, Cloud Bigtable, Redis
CDN: Akamai（グローバル）+ Fastly
```
参照: https://developers.cyberagent.co.jp/blog/archives/30291/

---

### 3-2. U-NEXT

**基本情報**

| 項目 | 内容 |
|-----|------|
| サービス開始 | 2007年（動画サービスとして）|
| 利用者数 | 約400万人（2024年）|
| クラウド基盤 | AWS（主要）+ オンプレミス（一部）|
| 親会社 | USEN-NEXT HOLDINGS |
| 技術情報 | https://tech.unext.co.jp/ |

**選定根拠**

```
① 国内VOD最大手としての安定運用事例
   - NTTドコモとの提携によるdTV統合の技術的対応
   - コンテンツ数31万本以上の大規模カタログ管理

② 国内企業の「中規模クラウドネイティブ」の代表例
   - 海外大手と比較したときの技術格差が明確化できる
   - AWS Well-Architectedフレームワークの適用事例

③ 技術ブログの存在（成長中）
   - 2022年よりtech blogを開始（内容は限定的）
   - Zennやconnpass等での勉強会開催
```

---

### 3-3. dアニメストア (NTTドコモ)

**基本情報**

| 項目 | 内容 |
|-----|------|
| サービス開始 | 2012年 |
| 登録者数 | 約650万人（2024年）|
| クラウド基盤 | NTTドコモ クラウド + AWS（一部）|
| 親会社 | NTTドコモ |
| 技術情報 | NTTドコモ技術ジャーナル |

**選定根拠**

```
① 通信キャリア系クラウドの代表例
   - NTTグループ内製クラウド（NTT Communications等）の利用
   - 従来型エンタープライズとクラウドネイティブの境界事例

② アニメ特化という差別化モデル
   - 特定ジャンルに特化した運用（コンテンツ量 vs. 多様性）
   - 季節スパイク（新クール開始期）の対応パターン

③ 国内大手キャリアのDX進捗の可視化
   - NTTドコモのデジタル変革のショーケース
   - 技術情報は限定的だが、サービス品質から推察可能
```

---

## 4. 選定サービスの総合比較マップ

```
技術成熟度とスケールによるポジショニング

高スケール
    │
    │  YouTube ●────────────────── 世界最大
    │
    │  Netflix ●─────────────── MSA教科書
    │
    │  Disney+ ●───────────── MSA移行中
    │
    │  AbemaTV ●────────── 国内最先端
    │
    │  U-NEXT  ●─────── 国内安定
    │
低スケール  │  dアニメ ●── キャリア系
    │
    └──────────────────────────────────────────────
       低技術成熟度                    高技術成熟度

（注: ポジションは公開情報からの推定。2024年時点）
```

**次のドキュメント**: `03_comparison_matrix.md` → クラウドネイティブ比較マトリクス
