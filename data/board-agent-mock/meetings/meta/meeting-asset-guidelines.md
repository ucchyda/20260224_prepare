# 会議素材運用ガイド（meetings meta）

目的: Codex/Claudeが「どの資料をどこで読むか」を迷わないよう、会議素材の運用ルールを固定する。

## 命名ルール

- minutes: `YYYYMMDD_議事録_SmartHR.md`
- transcript: `YYYYMMDD_議事録_*.md`
- agenda（v1）: `YYYYMMDD_*` または `board-minutes-YYYY-MM-draft.md`
- agenda（v2）: `agenda_plus_facts_v2_YYYYMMDD.md` / `.json`
- alerts（v2）: `inmeeting_alerts_v2_YYYYMMDD.json`
- transcript chat（v1/v2）: `..._論理飛躍_チャット投稿質問.md`
- registry: `board-meeting-index.json` のように素材を機械参照できる形式

## 更新ルール（v1.2＋v2）

- 週次の会議で新規材料が出る場合:
  1. `minutes/` or `transcript/` に日時付きファイルを追加
  2. `registry/board-meeting-index.json` の `items` に `meeting_id` を追加
  3. v2素材がある場合は対象 `items[].v2_assets` を更新
  4. `board-minutes-meta.json` の `meetings` 配列を必要に応じて更新
- v2運用時はv1資産を残したまま、`agenda_plus_facts_v2_*` と `inmeeting_alerts_v2_*` を追加する

## 優先参照順

1. 会議前
   1. `minutes`（意思決定）
   2. `agenda_plus_facts_v2`（準備成果）
   3. `agenda`（既存）
   4. `transcript`
   5. `registry`
2. 会議中
   1. `agenda_plus_facts_v2`（最優先）
   2. `transcript`
   3. `minutes`
   4. `agenda`（既存）

## 参照時の優先条件

- 同一会議IDがある場合、v2では `agenda_plus_facts_v2 > minutes > transcript > agenda` を優先
- 未解決アクションの確認が必要な場合は最優先で `RACI/期限` が明示されたファイルを優先
- 根拠抽出時は同時に `evidence_id` が付与可能な素材を併用
- 会議中監査は最初に `v2_assets` を確認し、必要時のみ追加検索を行う
