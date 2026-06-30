# PLAN.md — 課題(0) 市場・サービス比較分析 & クラウドネイティブ差別化導出

> 作成日: 2026-06-30 | ブランチ: black | 担当: Team01

---

## 1. プロジェクト概要

本課題は「選択したドメインにおける海外・国内代表サービス」をクラウドネイティブ観点で比較分析し、
自プロジェクトが解決すべきペインポイントと差別化ポイントを定義することを目的とする。

---

## 2. 選定ドメイン

**動画ストリーミングサービス（Video Streaming Platform）**

選定根拠の詳細意思決定プロセスは `PROCESS.md` を参照。

| 区分 | サービス名 | 選定根拠 |
|------|-----------|---------|
| 海外① | Netflix | カオスエンジニアリング先駆者、技術ブログ最多、MSA教科書的事例 |
| 海外② | YouTube (Google) | 世界最大規模、SRE文化の源泉、公開技術情報豊富 |
| 海外③ | Disney+ | モノリス→MSA移行事例、急成長フェーズの障害報告公開 |
| 国内① | AbemaTV (CyberAgent) | 日本最大級ライブ配信、K8s大規模運用ブログ豊富 |
| 国内② | U-NEXT | 国内VOD最大手、NTTドコモ傘下で安定運用事例 |
| 国内③ | dアニメストア | NTTドコモ系、アニメ特化・国内利用者数トップクラス |

---

## 3. 成果物一覧

```
reports/video_streaming/
├── report/
│   ├── 00_executive_summary.md       # エグゼクティブサマリー
│   ├── 01_domain_overview.md         # ドメイン市場概観
│   ├── 02_service_selection.md       # サービス選定根拠（詳細）
│   ├── 03_comparison_matrix.md       # クラウドネイティブ比較マトリクス
│   ├── 04_pain_points.md             # ペインポイント5件以上
│   ├── 05_differentiation.md         # 差別化戦略
│   ├── 06_slo_reliability.md         # SLO・信頼性公示方式比較（選択課題）
│   └── 07_chaos_engineering.md       # カオスエンジニアリング比較（選択課題）
├── architectures/
│   ├── netflix_arch.md               # Netflix推定アーキテクチャ
│   ├── abematv_arch.md               # AbemaTV推定アーキテクチャ
│   ├── youtube_arch.md               # YouTube推定アーキテクチャ
│   └── our_target_arch.md            # 本プロジェクト目標アーキテクチャ
└── diagrams/
    ├── comparison_radar.md           # レーダーチャート（テキスト表現）
    ├── deployment_timeline.md        # デプロイ戦略比較タイムライン
    ├── pain_point_map.md             # ペインポイントマップ
    └── differentiation_canvas.md    # バリュープロポジションキャンバス
```

---

## 4. 作業フェーズ

| フェーズ | 内容 | 主要成果物 | 状態 |
|---------|------|---------|------|
| Phase 0 | 計画・意思決定フレーム構築 | PLAN.md, PROCESS.md | ✅ 完了 |
| Phase 1 | ドメイン概観 & サービス選定 | 01_, 02_ | 🔄 進行中 |
| Phase 2 | 比較マトリクス作成 | 03_, architectures/ | ⬜ 未着手 |
| Phase 3 | ペインポイント抽出 | 04_, pain_point_map | ⬜ 未着手 |
| Phase 4 | 差別化戦略定義 | 05_, differentiation_canvas | ⬜ 未着手 |
| Phase 5 | 選択課題（SLO・カオス） | 06_, 07_ | ⬜ 未着手 |
| Phase 6 | エグゼクティブサマリー統合 | 00_ | ⬜ 未着手 |

---

## 5. 適用ビジネスフレームワーク

| フレームワーク | 適用箇所 | 目的 |
|-------------|---------|------|
| MECE | ペインポイント分類、比較軸設計 | 漏れなくダブりなく |
| ブレインストーミング | ドメイン候補列挙、差別化アイデア発散 | 発散思考 |
| 重み付きスコアリング | ドメイン選定、サービス選定 | 定量的意思決定 |
| 比較マトリクス | クラウドネイティブ技術スタック比較 | 横断比較 |
| バリュープロポジションキャンバス | 差別化ポイント定義 | 顧客価値の言語化 |
| ポーターの5フォース | 市場構造把握 | 競争環境理解 |
| フィッシュボーン図 | ペインポイント根本原因分析 | 因果関係整理 |

---

## 6. トークン・セッション効率化方針

- ドキュメントを **小チャンク**（1ファイル ≦ 400行）に分割
- 図表はMermaid/ASCII形式（外部レンダリング不要、ローカル完結）
- 各フェーズ完了後に `git commit` でチェックポイント作成
- ファイル間参照は相対パスで統一
- セッション中断時も `git log` で進捗再確認可能

---

## 7. 参照情報源カテゴリ

| カテゴリ | 主要ソース |
|---------|---------|
| Netflix技術情報 | https://netflixtechblog.com/ |
| Google/YouTube SRE | https://sre.google/books/ |
| AbemaTV Engineering | https://developers.cyberagent.co.jp/blog/ |
| CNCF Landscape | https://landscape.cncf.io/ |
| 公開障害報告 | StatusPage, GitHub Issues, Postmortem公開 |
