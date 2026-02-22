# 会議素材運用ガイド（meetings meta）

目的: Codex/Claudeが「どの資料をどこで読むか」を迷わないよう、会議素材の運用ルールを固定する。

## 命名ルール
- minutes: `YYYYMMDD_議事録_SmartHR.md`
- transcript: `YYYYMMDD_議事録_*.md`
- agenda: `YYYYMMDD_*` または `board-minutes-YYYY-MM-draft.md`
- registry: `board-meeting-index.json` のように素材を機械参照できる形式

## 更新ルール（v1.2）
- 週次の会議で新規材料が出る場合:
  1. `minutes/` or `transcript/` に日時付きファイルを追加
  2. `registry/board-meeting-index.json` の `items` に `meeting_id` を追加
  3. `board-minutes-meta.json` の `meetings` 配列を必要に応じて更新
  4. 変更時は `data/board-agent-mock/README.md` の関連テーブルを同期

## 優先参照順
1. `minutes`（意思決定）
2. `transcript`（発言証跡）
3. `agenda`（次回会議の準備）
4. `registry`（関連性解決）

## 参照時の優先条件
- 同一会議IDがある場合、`minutes > transcript > agenda` の順で読み込む
- 未解決アクションの確認が必要な場合は最優先で `RACI/期限` が明示されたファイルを優先
- 根拠抽出時は同時に `evidence_id` が付与可能な素材を併用

