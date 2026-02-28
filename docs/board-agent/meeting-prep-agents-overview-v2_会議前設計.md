# 会議前エージェント構成（v2）  

本資料は会議前監査（MeetingPrep）領域のみを取りまとめた定義です。  
会議中の設計は、`/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/inmeeting-agents-overview-v2_会議中監査.md` を参照してください。

## 1. 会議前の全体像

| 区分 | 英語名 | 日本語名（運用向け） | 役割 |
| --- | --- | --- | --- |
| 主役 | `MeetingPrepOrchestratorAgentV2` | **会議前司令Agent（v2）** | 過去議事録、社内情報、公開情報を統合して `agenda_plus_facts` を作成する中心制御役。 |
| サブ | `InternalInfoCollectorAgent` | **社内情報収集Agent** | `minutes`、KPI、RACI、監査、財務などの社内データを抽出し、`fact_bundle` に整形。 |
| サブ | `PublicResearchAgent` | **公開情報収集Agent** | 景況・規制・競合・金利など外部観点を収集し、会議前ファクトへ接続可能な形で整形。 |
| サブ | `FactCoverageReviewAgent` | **根拠網羅レビュアー** | `agenda_plus_facts` と `fact_bundle` を照合し、根拠未確定（`evidence_gap`）を抽出。 |
| サブ | `MeetingPrepLeaderAgent` | **会議前リードエージェント** | 重複排除、優先順位付け、欠落補正、最終出力整形を実施。 |

## 2. 会議前で扱う主要I/F

- 入力
  - `meeting_id`
  - `meeting_scope`
  - `latest_minutes_id`
  - `target_minutes_candidates[]`
  - `evidence_sources[]`
  - `run_constraints`（会議前は時間制約なし運用が前提）
- 出力
  - `meeting_id`
  - `agenda_plus_facts[]`
  - `fact_bundle[]`
  - `evidence_gaps[]`
  - `open_items[]`
  - `review_summary`

## 3. 会議前フロー（時系列）

1. `MeetingPrepOrchestratorAgentV2` が対象会議（`meeting_id`）と素材候補を確定。  
2. `InternalInfoCollectorAgent` と `PublicResearchAgent` を起動して社内/公開情報を収集。  
3. `FactCoverageReviewAgent` が `agenda_plus_facts` の根拠欠損を検出。  
4. `MeetingPrepLeaderAgent` が最終統合し、重複除去・優先順位付け・open item分離。  
5. `agenda_plus_facts` を監査可能なJSON/MDとして出力。  

## 4. 監査品質ルール（v2）

- `agenda_plus_facts` の各項目に最低1件の `evidence_ref` を付与する。  
- 根拠が未確定の主張は `evidence_gaps` として明示し、推測で補完しない。  
- `open_items` は期限・責任者候補（RACI）を示せる範囲で追記。  
- `Risk` は `High / Medium / Low` で明示し、未解決項目を上位に保つ。  

## 5. 監査観点（会議前）

- 未解決課題の継続性（前回未完了項目の再提示）  
- 根拠の母集団定義、期間定義、比較基準の明示  
- 次回会議で即実行可能なアクション候補  
- 社内情報と公開情報の整合確認  

## 6. 参考ファイル

- 会議前主役定義: `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/meeting-prep-orchestrator-agent-v2.md`  
- 社内収集: `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/internal-info-collector-agent-v2.md`  
- 公開情報収集: `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/public-research-agent-v2.md`  
- 根拠欠損レビュー: `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/fact-coverage-review-agent-v2.md`  
- 会議前実装例: `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/agenda/agenda_plus_facts_v2_20260224.json`  
- 会議前実行導線: `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/board-agent-finalized-guide-v1.2.md`（全体）
