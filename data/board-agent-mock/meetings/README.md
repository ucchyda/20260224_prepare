# meetings ディレクトリ構成（取締役会エージェント用）

このディレクトリは、会議資料（議事録／アジェンダ／文字起こし）を
**Codex/Claudeが初見で読み解きやすい**ように分離しています。

## 主要フォルダ

- `agenda/`
  - 会議前に使うアジェンダ（次回会議向け）
  - 現時点では `2026-02-24` 付近の提案資料を配置
- `minutes/`
  - 議事録（意思決定・未解決項目・議事要約を含む）
  - 今回は SmartHR想定で、`2026-01`〜`2026-02-22`までの実データ近似を配置
- `transcript/`
  - 会議発言の逐語データ
  - 現行: `partial/`（未完）/ `complete/`（議事録化済み）
  - 現時点: `transcript/partial/20260224_議事録_SmartHR_30分.md`
- `alerts/`
  - 会議中監査の指摘（Teams投稿想定）を会議別に格納
- `registry/`
  - 会議IDとファイル対応を管理する索引
  - 例: `board-meeting-index.json`
- `meta/`
  - 会議素材運用ルール（ファイル命名規則、更新粒度、必読項目）
  - 例: `meeting-asset-guidelines.md`
- `inmeeting-critic-guide-v1.2.md`
  - Teams連携を前提にした会議中監査（論理破綻指摘）汎用ガイド
- `_legacy/`
  - 過去の試行的フォルダを保管する退避場所

## 既定パス

- 会議索引: `board-minutes-meta.json`
- SmartHR議事録: `minutes/2026-01-06`〜`minutes/2026-02-22`
- 文字起こし: `transcript/partial/20260224_議事録_SmartHR_30分.md`
- 会議中監査（監査ログ/投稿文）: `alerts/20260224_inmeeting-critic-guide.md`

## 参照優先順位（Agent用）
1. `meetings/board-minutes-meta.json`（会議ID）
2. `meetings/registry/board-meeting-index.json`（素材パス）
3. `agenda/` / `minutes/` / `transcript/` の本文
4. `meta/meeting-asset-guidelines.md`（運用ルール）
