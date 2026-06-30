# PLAN.md — 課題(0) 市場・サービス比較分析 & クラウドネイティブ差別化導出

> 作成日: 2026-06-30 | ブランチ: black | 担当: Team01

---

## 1. プロジェクト概要

本課題は「選択したドメインにおける海外・国内代表サービス」をクラウドネイティブ観点で比較分析し、
自プロジェクトが解決すべきペインポイントと差別化ポイントを定義することを目的とする。

---

## 2. 選定ドメイン

**B2C サブスクリプション & 電子決済プラットフォーム**

電子決済（定期課金・都度課金）を核にもつ B2C デジタルサービス群を対象とする。
コンテンツ配信系・EC/クイックコマース系・EdTech系の 3 サブドメインを横断して比較する。

選定根拠の詳細意思決定プロセスは `PROCESS.md` を参照。

| 区分 | サービス | 国 | サブドメイン | 選定スコア |
|------|---------|----|-----------|-----------:|
| 海外① | Netflix | 🇺🇸 米国 | 動画ストリーミング | 20/20 |
| 海外② | Spotify | 🇸🇪 スウェーデン | 音楽ストリーミング | 18/20 |
| 海外③ | Amazon (Prime) | 🇺🇸 米国 | ECコマース | 17/20 |
| 国内① | Coupang (쿠팡) | 🇰🇷 韓国 | クイックコマース | 19/20 |
| 国内② | Market Kurly (마켓컬리) | 🇰🇷 韓国 | 生鮮デリバリー | 15/20 |
| 国内③ | Inflearn (인프런) | 🇰🇷 韓国 | EdTech/オンライン学習 | 13/20 |

> 注: 「国内」は本チームの主要市場である韓国を指す。

---

## 3. 成果物一覧

```
reports/b2c_platform/
├── report/
│   ├── 00_executive_summary.md       # エグゼクティブサマリー
│   ├── 01_domain_overview.md         # ドメイン市場概観
│   ├── 02_service_selection.md       # サービス選定根拠（詳細）
│   ├── 03_comparison_matrix.md       # クラウドネイティブ比較マトリクス
│   ├── 04_pain_points.md             # ペインポイント 5 件以上
│   ├── 05_differentiation.md         # 差別化戦略
│   ├── 06_slo_reliability.md         # SLO・信頼性公示方式比較（選択）
│   └── 07_chaos_engineering.md       # カオスエンジニアリング比較（選択）
├── architectures/
│   ├── netflix_arch.md               # Netflix 推定アーキテクチャ
│   ├── spotify_arch.md               # Spotify 推定アーキテクチャ
│   ├── coupang_arch.md               # Coupang 推定アーキテクチャ
│   ├── marketkurly_arch.md           # Market Kurly 推定アーキテクチャ
│   └── our_target_arch.md            # 本プロジェクト目標アーキテクチャ
└── diagrams/
    ├── comparison_radar.md           # レーダーチャート（テキスト形式）
    ├── payment_flow.md               # 電子決済フロー比較
    ├── deployment_strategies.md      # デプロイ戦略比較
    ├── pain_point_map.md             # ペインポイントマップ（フィッシュボーン）
    └── differentiation_canvas.md    # バリュープロポジションキャンバス
```

---

## 4. 作業フェーズ

| フェーズ | 内容 | 主要成果物 | 状態 |
|---------|------|---------|------|
| Phase 0 | 計画・意思決定フレーム構築 | PLAN.md, PROCESS.md | ✅ 完了 |
| Phase 1 | ドメイン概観 & サービス選定 | 01_, 02_ | ⬜ 未着手 |
| Phase 2 | 比較マトリクス & アーキテクチャ図 | 03_, architectures/ | ⬜ 未着手 |
| Phase 3 | ペインポイント抽出 | 04_, pain_point_map | ⬜ 未着手 |
| Phase 4 | 差別化戦略定義 | 05_, differentiation_canvas | ⬜ 未着手 |
| Phase 5 | 選択課題（SLO・カオス） | 06_, 07_ | ⬜ 未着手 |
| Phase 6 | エグゼクティブサマリー統合 | 00_ | ⬜ 未着手 |

---

## 5. 適用ビジネスフレームワーク

| フレームワーク | 適用箇所 | 目的 |
|-------------|---------|------|
| MECE | ペインポイント分類、比較軸設計 | 漏れなくダブりなく網羅 |
| ブレインストーミング | ドメイン候補列挙、差別化アイデア | 発散思考 |
| 重み付きスコアリング | ドメイン・サービス選定 | 定量的意思決定 |
| 比較マトリクス | クラウドネイティブ技術比較 | 横断比較の可視化 |
| バリュープロポジションキャンバス | 差別化定義 | 顧客価値の言語化 |
| ポーターの5フォース | 市場構造分析 | 競争環境の構造化 |
| フィッシュボーン図 | ペインポイント根本原因分析 | 因果関係の整理 |

---

## 6. トークン・セッション効率化方針

- ドキュメントを **小チャンク**（1 ファイル ≦ 400 行）に分割
- 図表は Mermaid / ASCII 形式（外部レンダリング不要、ローカル完結）
- 各フェーズ完了後に `git commit` でチェックポイント作成
- セッション中断時も `git log --oneline` で進捗再確認可能

---

## 7. 主要参照情報源

| サービス / 組織 | URL |
|--------------|-----|
| Netflix Tech Blog | https://netflixtechblog.com/ |
| Spotify Engineering | https://engineering.atspotify.com/ |
| Coupang Engineering | https://medium.com/coupang-engineering |
| Market Kurly Tech Blog | https://helloworld.kurly.com/ |
| Inflearn Tech Blog | https://tech.inflearn.com/ |
| AWS Architecture Blog | https://aws.amazon.com/blogs/architecture/ |
| CNCF Landscape | https://landscape.cncf.io/ |
| Google SRE Books | https://sre.google/books/ |
