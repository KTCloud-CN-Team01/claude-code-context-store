# 03 — クラウドネイティブ比較マトリクス（5 軸）

> 参照日: 2026-06-30  
> 凡例: ◎ 非常に優れている / ○ 優れている / △ 普通 / × 課題あり / 推定: 公開情報からの推察

---

## 1. サマリーマトリクス

```
                   Netflix  Spotify  Amazon  Coupang  M.Kurly  Inflearn
─────────────────────────────────────────────────────────────────────────
① アーキテクチャ     ◎       ◎        ◎       ○        ○        △
② CI/CD・デプロイ    ◎       ◎        ◎       ○        ○        △
③ オブザーバビリティ  ◎       ○        ◎       ○        ○        △
④ レジリエンス       ◎       ○        ◎       ○        △        △
⑤ サービスメッシュ   ○       ○        ○       △        △        ×
─────────────────────────────────────────────────────────────────────────
総合評価            ◎       ◎        ◎       ○        △〜○     △
```

---

## 2. 軸① アーキテクチャ

### 2-1. マイクロサービス化の程度

| サービス | MSA 段階 | 分割粒度 | 根拠・出典 |
|---------|---------|---------|---------|
| Netflix | 完成期（700+ マイクロサービス）| 機能単位（推定） | https://netflixtechblog.com/tagged/microservices |
| Spotify | 完成期（250+ サービス） | Squad 単位（公開）| https://engineering.atspotify.com/2022/03/society-of-systems/ |
| Amazon | 完成期（数千サービス）| Two-Pizza Team 単位 | https://aws.amazon.com/executive-insights/content/no-really-what-is-a-microservice/ |
| Coupang | 移行中〜完成期 | 推定: ドメイン単位 | https://medium.com/coupang-engineering/our-journey-to-continuous-delivery-at-coupang-105634185e28 |
| Market Kurly | 移行中 | 推定: 주문/상품/회원/배송 | https://helloworld.kurly.com/blog/introducing-event-storming/ |
| Inflearn | 移行中（モノリス残存）| 推定: 강의/결제/사용자 | https://tech.inflearn.com/post/276 |

### 2-2. API 設計比較

```
Netflix:
  外部: REST(JSON) via Zuul APIゲートウェイ
  内部: gRPC + 独自RPC
  BFF: Falcor(旧) → GraphQL 移行中
  参照: https://netflixtechblog.com/how-netflix-scales-its-api-with-graphql-federation-part-1-ae3557c187e2

Spotify:
  外部: REST + OAuth2
  内部: gRPC（Hermes プロトコル）
  BFF: 推定: Web/Mobile 別
  参照: https://engineering.atspotify.com/2023/08/how-we-handle-thousands-of-api-endpoints-at-spotify/

Amazon:
  外部: REST + GraphQL (Amplify)
  内部: 独自 IPC → SQS/SNS イベント駆動
  BFF: 推定: Device 別 BFF

Coupang:
  外部: REST
  内部: 推定: Kafka イベント駆動 + REST
  参照: https://medium.com/coupang-engineering/event-driven-architecture-at-coupang-3e8b2f6a6f07

Market Kurly:
  外部: REST
  内部: 推定: Kafka + REST
  参照: https://helloworld.kurly.com/blog/kurly-event-driven-architecture/

Inflearn:
  外부: REST
  내부: 추정: Spring 내부 호출 → Kafka 전환 중
```

### 2-3. データ設計比較

```
┌──────────────┬────────────────────────────────────────────────────┐
│ サービス      │ データ設計の特徴                                      │
├──────────────┼────────────────────────────────────────────────────┤
│ Netflix      │ Cassandra（ユーザー状態）+ EVCache（L2キャッシュ）     │
│              │ + DynamoDB（設定）+ MySQL（課金）                    │
├──────────────┼────────────────────────────────────────────────────┤
│ Spotify      │ Cassandra（再生履歴）+ BigQuery（分析）               │
│              │ + PostgreSQL（プレイリスト）                          │
├──────────────┼────────────────────────────────────────────────────┤
│ Amazon       │ DynamoDB（商品カタログ）+ Aurora（注文）              │
│              │ + ElastiCache（セッション）+ S3（大容量）             │
├──────────────┼────────────────────────────────────────────────────┤
│ Coupang      │ MySQL（注文/결제）+ DynamoDB（카탈로그）              │
│              │ + Redis（장바구니/세션）+ ES（검색）                  │
├──────────────┼────────────────────────────────────────────────────┤
│ Market Kurly │ Aurora MySQL + Redis + ES（추정）                   │
├──────────────┼────────────────────────────────────────────────────┤
│ Inflearn     │ Aurora MySQL + Redis + S3（강의 영상）（추정）        │
└──────────────┴────────────────────────────────────────────────────┘
```

---

## 3. 軸② CI/CD & デプロイ戦略

### 3-1. デプロイ頻度比較

| サービス | デプロイ頻度（推定） | デプロイ戦略 | ツール |
|---------|:-----------:|---------|------|
| Netflix | 数千回/日 | Canary → 段階ロールアウト | Spinnaker |
| Spotify | 数百回/日 | Feature Flag + Canary | 内製 Helios + GHA |
| Amazon | 11.6 秒に 1 回（2011 年公開値）| Blue/Green + Canary | 内製 Apollo |
| Coupang | 推定: 数十〜数百回/日 | 推定: Blue/Green | Jenkins + ArgoCD |
| Market Kurly | 推定: 数十回/日 | 推定: Rolling Update | GHA + ArgoCD |
| Inflearn | 推定: 数回/日 | 推定: Rolling Update | GHA |

参照（Amazon デプロイ頻度）: https://www.infoq.com/news/2013/10/web-tier-deployment-amazon/

### 3-2. デプロイパイプライン詳細

```
Netflix のデプロイパイプライン（Spinnaker）:
  コード → GitHub PR → CI（テスト）→ Artifact生成
  → Spinnaker: Bake → Deploy to Canary（1%）
  → 自動メトリクス検証（Kayenta: カナリア分析）
  → 段階的ロールアウト（10% → 50% → 100%）
  → 問題時: 自動ロールバック
  参照: https://netflixtechblog.com/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69

Spotify のデプロイパイプライン:
  コード → GitHub → GHA（ビルド/テスト）
  → Docker Image → GCR → Helios（CD）
  → Feature Flag で段階的有効化
  → Launchdarkly 等の Feature Flag サービス活用
  参照: https://engineering.atspotify.com/2022/11/seamless-cloud-native-experience-with-internal-developer-platform/

Coupang のデプロイパイプライン（推定）:
  コード → GitHub → Jenkins（ビルド）
  → ECR → ArgoCD → EKS（Canary or Rolling）
  → Datadog によるメトリクス検証
  参照: https://medium.com/coupang-engineering/our-journey-to-continuous-delivery-at-coupang-105634185e28
```

---

## 4. 軸③ オブザーバビリティ（可観測性）

### 4-1. 3本柱（Metrics / Logs / Traces）比較

```
┌──────────────┬──────────────┬──────────────┬──────────────────────────┐
│ サービス      │ Metrics      │ Logs         │ Traces                   │
├──────────────┼──────────────┼──────────────┼──────────────────────────┤
│ Netflix      │ Atlas（内製） │ Elasticsearch │ Zipkin 系（推定）         │
│              │ + Grafana    │ + Kibana     │ + 内製トレーシング           │
├──────────────┼──────────────┼──────────────┼──────────────────────────┤
│ Spotify      │ Prometheus   │ Stackdriver  │ OpenTelemetry（推定）      │
│              │ + Heroic     │ + BigQuery   │ + Jaeger（推定）           │
├──────────────┼──────────────┼──────────────┼──────────────────────────┤
│ Amazon       │ CloudWatch   │ CloudWatch   │ AWS X-Ray                │
│              │ + 内製        │ Logs         │ + 内製                    │
├──────────────┼──────────────┼──────────────┼──────────────────────────┤
│ Coupang      │ Datadog      │ ELK Stack    │ Datadog APM（推定）        │
├──────────────┼──────────────┼──────────────┼──────────────────────────┤
│ Market Kurly │ Datadog      │ CloudWatch   │ Datadog APM（推定）        │
│              │ （推定）      │ + ELK（推定）│                          │
├──────────────┼──────────────┼──────────────┼──────────────────────────┤
│ Inflearn     │ CloudWatch   │ CloudWatch   │ なし or 導入初期（推定）    │
│              │ + Sentry     │              │                          │
└──────────────┴──────────────┴──────────────┴──────────────────────────┘
```

### 4-2. 決済観測の特殊性

```
決済系メトリクスで必須の観測項目:
  ① 決済 成功率（Success Rate）             → SLO の核心
  ② 決済 レスポンスタイム P50/P95/P99        → ユーザー体験
  ③ PG（Payment Gateway）別エラー率         → 障害切り分け
  ④ 二重決済検知カウンター                   → データ整合性
  ⑤ 환불/취소 처리율                        → 異常検知

Netflix の決済観測（公開情報）:
  - 190 か国の PG 別成功率をリアルタイム監視
  - 異常検知: 決済成功率 < 閾値で自動アラート
  参照: https://netflixtechblog.com/keeping-movies-playable-while-improving-the-payment-stack-6b9de34cb044
```

---

## 5. 軸④ レジリエンス

### 5-1. カオスエンジニアリング成熟度

```
Level 0: カオスエンジニアリングなし
Level 1: 手動障害注入（テスト環境のみ）
Level 2: 自動化された障害注入（ステージング）
Level 3: 本番環境での自動カオス実行
Level 4: GameDay + 継続的カオス（フルオートメーション）

──────────────────────────────────────────────────────
Netflix  : Level 4 ◎（Chaos Monkey を本番で常時稼働）
Amazon   : Level 4 ◎（Game Day + 本番 FIS）
Spotify  : Level 3 ○（推定: ステージング + 一部本番）
Coupang  : Level 2 △（推定: CI パイプラインに組み込み）
M. Kurly : Level 1 △（推定: 手動テスト中心）
Inflearn : Level 0 ×（推定: 未実施）
──────────────────────────────────────────────────────
```

参照（Netflix Chaos Engineering）:
- https://netflixtechblog.com/the-netflix-simian-army-16e57fbab116
- https://principlesofchaos.org/

### 5-2. サーキットブレーカー & フォールバック

```
Netflix Hystrix（旧）→ Resilience4j（現在推奨）:
  - マイクロサービス間呼び出しにサーキットブレーカーを標準装備
  - フォールバック: キャッシュからの応答、デフォルト値返却
  参照: https://github.com/Netflix/Hystrix/wiki

Amazon:
  - DynamoDB グローバルテーブル（マルチリージョン自動フェイルオーバー）
  - Route 53 ヘルスチェック + フェイルオーバールーティング

Coupang（推定）:
  - Spring Cloud Circuit Breaker（Resilience4j）採用
  - 결제 PG 다중화（KakaoPay / NaverPay / 토스 を並列保持、片方落ちでも続行）
```

---

## 6. 軸⑤ サービスメッシュ / 通信設計

### 6-1. サービスメッシュ採用比較

| サービス | メッシュ技術 | 採用範囲 | 主目的 |
|---------|-----------|---------|------|
| Netflix | Envoy + 独自実装 | 全サービス（推定） | トラフィック管理・観測性 |
| Spotify | Envoy + Istio（推定） | 新規サービスから | mTLS + トラフィック制御 |
| Amazon | AWS App Mesh（Envoy）| AWS 上サービス | サービス間通信標準化 |
| Coupang | 推定: Istio 一部採用 | 推定: 主要サービス | トラフィック管理 |
| Market Kurly | 推定: 未導入〜検討 | — | — |
| Inflearn | 推定: 未導入 | — | — |

### 6-2. 決済サービスの通信セキュリティ

```
決済マイクロサービス間の通信セキュリティ要件:

最低要件:
  [✓] HTTPS（TLS 1.2 以上）
  [✓] OAuth2 / JWT 認証

推奨（CN レベル）:
  [✓] mTLS（mutual TLS）= サービスメッシュによる自動管理
  [✓] ゼロトラストネットワーク（VPC 内部でも認証必須）
  [✓] Secrets Manager（認証情報の動的ローテーション）

Netflix の実装:
  - Metatron（内製 PKI）でマイクロサービス間 mTLS を自動化
  参照: https://netflixtechblog.com/introducing-netflix-metatron-2e4c1f5a2290
```

---

## 7. 強み・弱み分析サマリー

### Netflix

```
強み:
  + クラウドネイティブの全技術を内製化・先駆け
  + カオスエンジニアリング本番稼働で最高レベルの耐障害性
  + デプロイ自動化（Kayenta）で人手のリスク排除

弱み:
  - 内製技術が多すぎてベンダーロックインに近い状態
  - Titus（内製 K8s 相当）の保守コストが高い
  参照: https://netflixtechblog.com/titus-the-netflix-container-management-platform-on-the-move-d9879891b39e
```

### Spotify

```
強み:
  + Backstage（IDP）で開発者体験を統一し生産性が高い
  + Squad/Tribe モデルで CN 組織文化が浸透
  + GCP への完全移行で運用コスト最適化

弱み:
  - 推定: サービスメッシュの全社標準化が遅い
  - Freemium 故の決済障害が直接的な収益影響を受けにくい（悪い意味で障害優先度が下がりやすい）
```

### Coupang

```
강점:
  + 모놀리식 → MSA 전환 사례를 상세히 공개（韓国企業では稀）
  + Coupang Pay 자체 PG로 결제 신뢰성을 내재화
  + 물류 자동화（로켓배송）와 클라우드 통합 수준이 높음

약점:
  - 서비스 메시 도입이 해외 선진 기업 대비 지연（추정）
  - 카오스 엔지니어링 실천 수준 공개 정보 부족
```

### Market Kurly

```
강점:
  + 빠른 성장에 따른 스케일링 해결 사례가 풍부
  + Kotlin 도입으로 백엔드 현대화 진행 중
  + Event Storming 기반 DDD 설계를 팀 문화로 도입

약점:
  - 레질리언스 설계 공개 정보 부족
  - 관측성 도구 도입이 Datadog 의존으로 집중（단일 벤더 리스크）
```

### Inflearn

```
강점:
  + 스타트업 규모에서 실용적인 기술 선택（오버 엔지니어링 회피）
  + 결제 시스템 설계 공개 블로그가 상세함（토스페이먼츠 연동 사례）

약점:
  - 서비스 메시·카오스 엔지니어링 미도입
  - 모놀리식 잔존으로 MSA 전환 과도기적 상태
  - 관측성 3 기둥 중 Traces 부재（추정）
```

**次のドキュメント**: `04_pain_points.md` → ペインポイント抽出（5 件以上）
