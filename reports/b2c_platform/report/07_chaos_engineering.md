# 07 — カオスエンジニアリング適用水準比較（選択課題）

> 참조일: 2026-06-30

---

## 1. カオスエンジニアリング成熟度モデル

```
[Chaos Engineering Maturity Model（5 レベル）]

Level 0: 未実施
  → 障害を恐れて意図的な障害注入を行わない
  → 本番障害で初めて脆弱性に気づく

Level 1: 手動・計画的（テスト環境）
  → ステージング環境で月次 GameDay
  → 手動でサーバーを停止してみる
  → 結果はメモ、体系的な計測なし

Level 2: 自動化（ステージング環境）
  → CI/CD パイプラインに障害注入テストを統合
  → 自動化ツール使用（LitmusChaos, Gremlin 等）
  → 計測と合否判定を自動化

Level 3: 本番環境での限定的実施
  → 本番環境の一部（1%〜）で障害注入
  → SLO を監視しながら実施
  → Rollback 計画あり

Level 4: 本番環境での継続的カオス（フルオートメーション）
  → Chaos Monkey が毎日自動実行
  → GameDay を定期開催（ChaosKong レベル）
  → 障害注入がエンジニア文化として定着
```

---

## 2. 各サービスのカオスエンジニアリング現状

### 2-1. Netflix — Level 4（最高水準）

```
実装内容（公開情報）:

[Simian Army（シミアン軍団）]
  ツール名           | 役割
  ──────────────────────────────────────────────────────
  Chaos Monkey      | ランダムに EC2 インスタンスを終了
  Chaos Gorilla     | 1 AZ を丸ごと無効化
  Chaos Kong        | 1 AWS Region を丸ごき無効化
  Latency Monkey    | サービス間通信に人工遅延を注入（P99 悪化シミュレーション）
  Conformity Monkey | ベストプラクティス違反のインスタンスを検出・修正
  Security Monkey  | セキュリティ違反を検出・アラート
  Doctor Monkey     | CPU/メモリ異常の検出
  Janitor Monkey    | 未使用リソースの自動クリーンアップ

実行スケジュール:
  Chaos Monkey: 平日 業務時間中 に自動実行（エンジニアが在席中）
  Chaos Kong: 四半期 GameDay（事前告知なし）

哲学:
  "障害は避けられない。いつ起こるかではなく、起きたときに
   どう対応できるかを訓練する"
  参照: https://netflixtechblog.com/the-netflix-simian-army-16e57fbab116

結果:
  - AWS 大規模障害時に Netflix のみ大部分が継続稼働（2012 年事例）
  - 単一障害点（SPOF）がほぼ解消
```

参照:
- https://github.com/Netflix/SimianArmy
- https://principlesofchaos.org/（Netflix エンジニアが作成）

---

### 2-2. Amazon (AWS) — Level 4

```
実装内容（公開情報）:

[AWS Fault Injection Service (FIS)]
  - 2021 年 GA の AWS 公式カオスエンジニアリングサービス
  - AWS インフラへの障害注入を マネージドに提供
  参照: https://aws.amazon.com/fis/

[内部での実施（推定）]
  - Amazon 内部では "GameDay" を定期実施
  - EC2 AZ 障害、DynamoDB 遅延、S3 接続不可 等をシミュレーション
  - Two-Pizza Team 単位で独立して実施

結果:
  - AWS 自体が 99.99%+ の SLA を維持できているのは
    継続的なカオス実験による事前検証の賜物
```

---

### 2-3. Spotify — Level 2〜3（推定）

```
公開情報:
  - "Blameless Culture" は確立、カオスエンジニアリングも実施（推定）
  - Engineering Blog での言及:
    "We run experiments in staging and sometimes in production"
    参照: https://engineering.atspotify.com/2021/02/engineering-for-resiliency/

  - ツール: LitmusChaos または Gremlin（推定）

推定実施内容:
  - ステージング: 毎週自動実行（CI パイプライン統合）
  - 本番: 四半期 GameDay（特定 Squad）
  - Kubernetes ノード障害 / Pod 強制削除 / ネットワーク遅延注入

Level 3 評価根拠:
  - 本番での実施を示唆する記述あり
  - ただし Netflix レベルの常時実施は未確認
```

---

### 2-4. Coupang — Level 1〜2（推定）

```
公개 정보:
  - 카오스 엔지니어링 공개 기사: 없음（간접 언급 수준）
  - AWS re:Invent 발표에서 복원력 설계 언급 있음
    참조: https://aws.amazon.com/solutions/case-studies/coupang/

추정 내용:
  - CI/CD 파이프라인에 기본 헬스체크 통합
  - 스테이징 환경에서 일부 장애 시뮬레이션 실시（추정）
  - 본번 카오스 실험: 미실시 가능성 높음

Level 1-2 평가 근거:
  - Kafka / MSA 아키텍처 수준으로 보아 최소한 Level 1은 실시 중
  - 공개 사례 부재 → Level 4는 아님
  - 쿠팡의 규모（24.4조원）에서 Level 0은 비현실적
```

---

### 2-5. Market Kurly — Level 1（推定）

```
공개 정보:
  - DB 페일오버 사례 공개
    참조: https://helloworld.kurly.com/blog/database-failover/
    → RDS 페일오버를 수동으로 테스트한 사례
    → Level 1（수동, 계획적）에 해당

미실시 항목（추정）:
  - Kubernetes Pod 무작위 삭제
  - 네트워크 지연 주입
  - AZ 장애 시뮬레이션

Level 1 평가 근거:
  - DB 페일오버 블로그 = "수동 장애 주입"의 증거
  - CI 파이프라인 통합 카오스는 미언급
```

---

### 2-6. Inflearn — Level 0（推定）

```
공개 정보:
  - 카오스 엔지니어링 관련 언급: 없음

Level 0 평가 근거:
  - 스타트업 규모（50인 미만 추정）
  - 관측성 기반 자체가 미성숙 → 카오스 실험 결과 측정 불가
  - 블로그 내용이 MSA 전환 / 결제 시스템 기본 구성에 집중
  - Level 0이 현 단계에서는 합리적 선택（오버 엔지니어링 회피）
```

---

## 3. 比較マトリクス

```
                        Netflix  Spotify  Amazon  Coupang  M.Kurly  Inflearn
──────────────────────────────────────────────────────────────────────────────
成熟度レベル              4        3        4       2        1        0
自動化ツール              ◎        ○        ◎      △        △        ×
本番環境での実施          ◎        △        ◎      ×        ×        ×
Kubernetes Pod 削除       ◎        ○        ◎      △        ×        ×
ネットワーク障害注入       ◎        ○        ◎      △        ×        ×
Region 障害（ChaosKong）  ◎        ×        ◎      ×        ×        ×
定期 GameDay              ◎        ○        ◎      △        △        ×
文化としての定着           ◎        ◎        ◎      △        ×        ×
──────────────────────────────────────────────────────────────────────────────
```

---

## 4. 本プロジェクトのカオスエンジニアリング導入計画

```
推奨ロードマップ:

Phase 1（Month 1-2）: Level 1 確立
  [✓] LitmusChaos インストール（Kubernetes 上）
  [✓] ステージング環境での手動 Pod 削除テスト
  [✓] 結済サービスの Circuit Breaker 動作確認

Phase 2（Month 2-4）: Level 2 確立
  [ ] GitHub Actions への LitmusChaos 実験統合
      → PR マージ前に自動カオステスト実行
  [ ] 実験テンプレート作成:
      ① Pod 削除（結済サービス, 주문 서비스）
      ② CPU ストレス注入（HPA 動作確認）
      ③ Kafka 遅延注入（Saga Workflow 確認）

Phase 3（Month 4-6）: Level 3 達成
  [ ] 本番環境での 1% トラフィックカオス実験
  [ ] SLO 監視下での自動実行
  [ ] 四半期 GameDay 実施

ツール選定:
  LitmusChaos（OSS, K8s 対応）: https://litmuschaos.io/
  Gremlin（商用, 本番向け）:     https://www.gremlin.com/
  AWS FIS（AWS 環境向け）:       https://aws.amazon.com/fis/

カオス実験テンプレート例:
  결제 PG 타임아웃 시뮬레이션:
    → 결제 서비스 → PG 연결에 3000ms 지연 주입
    → 기대 결과: Circuit Breaker OPEN → KakaoPay 자동 전환
    → 결제 성공률 ≥ 99.9% 유지 확인
```

参照:
- https://litmuschaos.io/
- https://principlesofchaos.org/
- https://netflixtechblog.com/tagged/chaos-engineering
