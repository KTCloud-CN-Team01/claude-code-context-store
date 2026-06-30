# 02 — サービス選定根拠（詳細）

> 参照日: 2026-06-30

---

## 1. 選定サービス一覧

| # | サービス | 区分 | 国 | サブドメイン | スコア |
|---|---------|------|----|-----------|---------:|
| 1 | Netflix | 海外① | 🇺🇸 | 動画サブスク | 19/20 |
| 2 | Spotify | 海外② | 🇸🇪 | 音楽サブスク | 18/20 |
| 3 | Amazon (Prime) | 海外③ | 🇺🇸 | EC + サブスク | 19/20 |
| 4 | Coupang (쿠팡) | 国内① | 🇰🇷 | クイックコマース | 20/20 |
| 5 | Market Kurly (마켓컬리) | 国内② | 🇰🇷 | 生鮮デリバリー | 16/20 |
| 6 | Inflearn (인프런) | 国内③ | 🇰🇷 | EdTech | 14/20 |

---

## 2. 海外サービス詳細

### 2-1. Netflix

**基本情報**

| 項目 | 内容 |
|-----|------|
| 設立 | 1997 年（ストリーミング開始: 2007 年）|
| 有料会員数 | 約 2 億 6,900 万人（2024 Q1）|
| 月次 ARR 換算 | 約 39 億 USD/月（平均単価 $15 × 会員数）|
| クラウド基盤 | AWS 100%（2016 年にデータセンター完全撤退）|
| 公式技術ブログ | https://netflixtechblog.com/ |
| StatusPage | https://status.netflix.com/ |

**選定根拠**

```
[決済]: 世界 190 か国でのマルチカレンシー・マルチ決済 PG 統合
        → Braintree / PayPal / ローカル PG との統合アーキテクチャが複雑
[CN]:   Chaos Monkey（カオスエンジニアリングツール）の発明者・OSS 公開
        Spinnaker（マルチクラウド CD）開発・OSS 公開
        Hystrix（サーキットブレーカー）開発・OSS 公開
[情報]: Tech Blog に 1,000+ 記事、AWS re:Invent で毎年詳細発表
```

参照:
- https://netflixtechblog.com/completing-the-netflix-cloud-migration-783e1ea8c3c0
- https://netflixtechblog.com/tagged/chaos-engineering

**推定技術スタック**

```
言語          : Java, Python, JavaScript (Node.js), Go
コンテナ基盤  : Titus（内製 ECS 互換）→ Kubernetes 移行中
API ゲートウェイ: Zuul（OSS）
サービスメッシュ: 独自実装 + Envoy 採用中
CI/CD         : Spinnaker（自社 OSS）
観測性        : Atlas（内製メトリクス）, Winston（ログ）, Zipkin 系トレーシング
データストア  : Cassandra, EVCache（Memcached 拡張）, MySQL, DynamoDB
CDN           : Open Connect（独自 CDN、ISP に BOX 設置）
決済          : Braintree + ローカル PG（190 か国）
```

参照: https://netflixtechblog.com/netflix-oss-and-spring-boot-coming-full-circle-4855947713a0

---

### 2-2. Spotify

**基本情報**

| 項目 | 内容 |
|-----|------|
| 設立 | 2006 年（スウェーデン）|
| 月間アクティブユーザー | 約 6 億 200 万人（2024 Q1）|
| 有料会員 | 約 2 億 3,900 万人（2024 Q1）|
| Freemium 比率 | 有料 40% / 無料 60%（2024 Q1）|
| クラウド基盤 | GCP 主体（2016 年移行完了）|
| 技術ブログ | https://engineering.atspotify.com/ |

**選定根拠**

```
[決済]: Freemium → 有料転換の決済フローが複雑（トライアル・割引・ファミリー）
        月次請求の大規模バッチ処理が技術的課題
[CN]:   Squad/Tribe/Chapter/Guild 組織モデル（CN チーム設計の教科書）
        Backstage（Internal Developer Platform）を OSS 公開
        参照: https://backstage.io/
[情報]: Engineering Blog に 200+ 記事
        QCon/KubeCon 等での積極発表
```

**推定技術スタック**

```
言語          : Python, Java, Scala（データ）, JavaScript（フロントエンド）
コンテナ基盤  : GKE（Google Kubernetes Engine）
API 設計      : gRPC（サービス間）+ REST（外部）
CI/CD         : GitHub Actions + 内製 Helios（旧 Luigi）
観測性        : Grafana + Prometheus + 内製 Heroic（メトリクス DB）
データストア  : BigQuery（分析）, Cassandra（ユーザーデータ）, GCS
CDN           : Google Cloud CDN + Akamai
決済          : Stripe + Braintree + ローカル PG
IDP           : Backstage（OSS 公開）
```

参照: https://engineering.atspotify.com/2020/04/spotifys-shift-to-a-tribe-model/

---

### 2-3. Amazon (Prime / AWS)

**基本情報**

| 項目 | 内容 |
|-----|------|
| Amazon Prime 会員数 | 約 2 億人以上（2024）|
| EC 売上 (2023) | 約 2,310 億 USD（北米）|
| クラウド基盤 | AWS（自社）100% + 自社 DC |
| 技術情報 | https://www.amazon.science/ |
| AWS アーキテクチャ | https://aws.amazon.com/blogs/architecture/ |

**選定根拠**

```
[決済]: Amazon Pay の処理規模（秒数十万件）がベンチマーク
        マルチカレンシー・ワンクリック決済の特許元
[CN]:   AWS 自体が CN インフラの源泉
        Two-Pizza Team による MSA 分割の元祖
        Event-Driven Architecture（SQS/SNS/Kinesis）の実装例
[情報]: Amazon Science Blog, AWS Architecture Blog に詳細記事多数
```

**推定技術スタック**

```
言語          : Java, C++, Python, Rust（新規開発）
コンテナ基盤  : ECS / EKS（Fargate も利用）
サービスメッシュ: AWS App Mesh
CI/CD         : 内製 Apollo + CodePipeline
観測性        : CloudWatch + AWS X-Ray（トレーシング）
データストア  : DynamoDB, Aurora, ElastiCache, S3
CDN           : CloudFront + Lambda@Edge
決済          : Amazon Pay（独自 PG）+ ローカル決済
```

---

## 3. 国内（韓国）サービス詳細

### 3-1. Coupang (쿠팡)

**基本情報**

| 項目 | 내용 |
|-----|------|
| 設立 | 2010 年（ソウル）|
| 売上 (2023) | 約 24.4 兆ウォン（≒ 1.9 兆円）|
| 会員数（ロケットワウ）| 約 1,400 万人（2024）|
| クラウド基盤 | AWS（主体）+ 自社オンプレ（物流センター）|
| 技術ブログ | https://medium.com/coupang-engineering |
| IR 資料 | https://ir.coupang.com/ |

**選定根拠**

```
[決済]: Coupang Pay（자체 PG）を保有、決済 수수료 내재화
        Rocket Delivery の COD・카드결제・간편결제 統合
[CN]:   Spring Boot モノリス → MSA 移行の韓国最大規模事例
        Kafka 大規模運用（주문/재고 이벤트 스트리밍）
        AWS re:Invent 2022 での詳細発表
        参照: https://aws.amazon.com/solutions/case-studies/coupang/
[情報]: Medium に 100+ 記事、AWS case study 詳細
        전사 MSA 전환 과정이 공개됨
```

**推定技術スタック**

```
言語          : Java（Spring Boot）, Kotlin, Python（ML）
コンテナ基盤  : EKS（Kubernetes on AWS）
CI/CD         : Jenkins + ArgoCD → GitHub Actions 移行中
観測性        : Datadog + ELK Stack
データストア  : MySQL, DynamoDB, Redis, Elasticsearch
メッセージング: Kafka（주문/재고 이벤트）
CDN           : CloudFront + Akamai
決済          : Coupang Pay（자체）+ KakaoPay + NaverPay + 카드사
물류 시스템   : 自社 TMS/WMS（Fulfillment Center 自動化）
```

参照: https://medium.com/coupang-engineering/our-journey-to-continuous-delivery-at-coupang-105634185e28

---

### 3-2. Market Kurly (마켓컬리)

**基本情報**

| 項목 | 내용 |
|-----|------|
| 設立 | 2015 年（ソウル）|
| 売上 (2023) | 約 2.1 兆ウォン（≒ 1,600 億円）|
| 会員数 | 約 1,000 万人以上（2024 推定）|
| クラウド基盤 | AWS 主体 |
| 技術ブログ | https://helloworld.kurly.com/ |

**選定根拠**

```
[決済]: 새벽배송（早朝配送）の予約決済 + 정기구독（定期購読）の混在
        환불/반품 처리의 복잡한 결제 취소 플로우
[CN]:   빠른 성장(스타트업→대기업) 과정에서의 스케일링 문제가 공개됨
        MSA 전환 과정, ECS→EKS 마이그레이션 사례
        참조: https://helloworld.kurly.com/blog/introducing-event-storming/
[情報]: Tech Blog にスケーリング・DB 설계・CI/CD 記事が 100 件以上
```

**推定技術スタック**

```
언어          : Kotlin（백엔드 주력）, TypeScript（프론트）
컨테이너 기반  : EKS（Kubernetes on AWS）
CI/CD         : GitHub Actions + ArgoCD
관측성         : Datadog + CloudWatch
데이터스토어   : Aurora MySQL, Redis, Elasticsearch
메시징         : Kafka（주문/재고）
CDN           : CloudFront
결제           : KakaoPay + NaverPay + 토스페이먼츠 + 카드사
물류           : 자체 물류센터（콜드체인 온도관리 시스템 연동）
```

参照: https://helloworld.kurly.com/blog/kurly-devops-transformation/

---

### 3-3. Inflearn (인프런)

**基本情報**

| 項目 | 내용 |
|-----|------|
| 設立 | 2016 年（ソウル）|
| 누적 학습자 | 약 100 만명（2024）|
| 코스 수 | 약 50,000 강의 이상（2024）|
| クラウド基盤 | AWS 주체 |
| 技術ブログ | https://tech.inflearn.com/ |

**選定根拠**

```
[決済]: 강의 개별 판매 + 구독(클럽) 혼재 결제 모델
        강사 정산 시스템（수익 배분）의 복잡한 회계 처리
[CN]:   한국 스타트업의 클라우드 네이티브 성장 사례
        모놀리식 → MSA 전환 중（공개 블로그에 상세 기술）
        참조: https://tech.inflearn.com/post/276
[情報]: Tech Blog に 50+ 記事（DB 설계, MSA 전환, 결제 시스템 등）
```

**推定技術スタック**

```
언어          : Kotlin（Spring Boot）, TypeScript（React）
컨테이너 기반  : ECS → EKS 마이그레이션 중
CI/CD         : GitHub Actions
관측성         : Sentry + CloudWatch + Datadog（도입 중）
데이터스토어   : Aurora MySQL, Redis, S3（동영상）
CDN           : CloudFront（동영상 배포）
결제           : 토스페이먼츠 + KakaoPay + NaverPay
동영상         : S3 + CloudFront + MediaConvert
```

参照: https://tech.inflearn.com/post/276

---

## 4. 選定サービスの総合比較ポジショニングマップ

```
技術成熟度とスケールによるポジショニング

[高スケール]
      │
      │   Netflix ●─────── 2億6900万会員・190か国
      │
      │   Amazon ●──────── EC 世界最大
      │
      │   Spotify ●─────── 6億MAU・Freemium
      │
      │   Coupang ●────── 韓国EC最大・Rocket Delivery
      │
      │   Market Kurly ●─ 韓国生鮮EC・急成長
      │
[低スケール] │   Inflearn ●── 韓国EdTech・スタートアップ規模
      │
      └──────────────────────────────────────────────
           [低CN成熟度]              [高CN成熟度]

（注: 2024年公開情報ベースの推定）
```

**次のドキュメント**: `03_comparison_matrix.md` → クラウドネイティブ比較マトリクス（5 軸）
