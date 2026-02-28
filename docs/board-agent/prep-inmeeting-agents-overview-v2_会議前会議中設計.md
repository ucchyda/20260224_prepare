# 会議前／会議中エージェント設計（v2）

本資料は、SmartHR想定の会議監査ワークフローを、  
**会議前（MeetingPrep）** と **会議中（InMeeting）** に分けて整理した顧客説明用の要約です。  

対象の設計は以下を前提にしています。

- `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/20260224_会議エージェント設計書.md`
- `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/design_philosophy_v1.md`
- `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/*-v2.md`
- `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/evidence-gateway-agent-v2.md`
- `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/finance-capital-agent-team-v1.md`（v2の運用で参照）

---

## 1. 会議前エージェント（MeetingPrep）

| 区分 | 英語名 | 日本語名（運用向け呼称） | 役割 |
| --- | --- | --- | --- |
| 主役 | `MeetingPrepOrchestratorAgentV2` | **会議前司令Agent（v2）** | 過去議事録・社内情報・公開情報を統合し、次回会議向けの `agenda_plus_facts` を作成する中心制御役。 |
| サブ | `InternalInfoCollectorAgent` | **社内情報収集Agent** | `minutes`、KPI、RACI、組織、監査情報を時系列で抽出し、`fact_bundle` 化。 |
| サブ | `PublicResearchAgent` | **公開情報収集Agent** | 景況・規制・競合・金利等の外部情報を拾い、会議前ファクトへ接続可能な証拠化を行う。 |
| サブ | `FactCoverageReviewAgent` | **根拠網羅レビュアー** | `agenda_plus_facts` と `fact_bundle` を照合し、`evidence_gap`（根拠不足）を抽出・優先付け。 |
| サブ | `MeetingPrepLeaderAgent` | **会議前リードエージェント** | 各サブエージェントからの素材を統合し、重複排除・優先順位付け・出力整形を実施。 |

### 会議前の入出力（要点）

- 入力:  
  - `meeting_id` / `meeting_scope`  
  - `latest_minutes_id`  
  - `target_minutes_candidates`  
  - `evidence_sources`  
  - `run_constraints`（時間制約は原則 `null`）  
- 出力:  
  - `agenda_plus_facts[]`  
  - `fact_bundle[]`  
  - `evidence_gaps[]`  
  - `open_items[]`  
  - `review_summary`  

### 会議前フロー（時系列）

1. `MeetingPrepOrchestratorAgentV2` が会議素材の範囲を確定  
2. `InternalInfoCollectorAgent` と `PublicResearchAgent` を並列実行し、内外のファクトを収集  
3. `FactCoverageReviewAgent` が不足/不整合を抽出  
4. `MeetingPrepLeaderAgent` が最終整形して `agenda_plus_facts` を固定  

---

## 2. 会議中エージェント（InMeeting）

| 区分 | 英語名 | 日本語名（運用向け呼称） | 役割 |
| --- | --- | --- | --- |
| 主役 | `InMeetingOrchestratorAgentV2` | **会議中司令Agent（v2）** | 文字起こしを時系列監査し、疑義をアラート化。LowLatency前提で最小件数の投稿候補を返す。 |
| サブ | `LogicalJumpCriticAgent` | **論理飛躍監査Agent** | 発言の断定・因果の飛躍・定義混在を検知し、論理破綻疑義を抽出。 |
| サブ | `PremiseQuestionAgent` | **前提疑義Agent** | 事前ファクトと発言の双方を照合し、前提の根拠・期限・責任者を疑う。 |
| サブ | `EvidenceGatewayAgent` | **根拠ゲートウェイ** | 監査で必要な `evidence_id` / `evidence_source` を決定し、欠損時は `evidence_gap` として保留条件を返す。 |
| サブ | `FinanceCapitalEvidenceRoutingAgent` | **財務資本ルータAgent** | 財務・資本領域の発言だけを対象に、必要最小限で該当Workerへ分岐。 |
| Worker | `treasury-liquidity-worker` | **資金繰りWorker** | キャッシュ・流動性・支払い余力の整合を監査。 |
| Worker | `capital-markets-intel-worker` | **資本市場Worker** | 金利・調達・市場環境前提の更新漏れを監査。 |
| Worker | `capital-structure-worker` | **資本構成Worker** | 借入、満期、レバレッジ、コベナンツ、格付制約を監査。 |
| Worker | `capital-allocation-worker` | **資本配分Worker** | 投資／還元／維持の配分順位と資金条件整合を監査。 |
| Worker | `investment-diligence-worker` | **投資デューデリWorker** | NPV/IRR/回収、感度条件、承認前提の妥当性を監査。 |
| Worker | `financial-risk-governance-worker` | **財務ガバナンスWorker** | 契約条項、承認権限、規制・監査条件の遵守を監査。 |
| Worker | `peer-benchmarking-worker` | **外部比較Worker** | 公開情報や競合比較との乖離を監査。 |

### 会議中の入出力（要点）

- 入力:  
  - `meeting_id`  
  - `transcript_segment`（speaker / line / time / utterance_id / text）  
  - `agenda_plus_facts_path`  
  - `fact_bundle_ref[]`  
  - `urgency_profile=LowLatency`  
- 出力:  
  - `alerts[]`（`alert_id` / `severity` / `issue_type` / `trigger` / `claim_excerpt` / `evidence` / `question_text` / `action`）  

### 会議中フロー（時系列）

1. `InMeetingOrchestratorAgentV2` が文字起こしセグメントを受信  
2. `LogicalJumpCriticAgent` と `PremiseQuestionAgent` を先行実行  
3. 高リスク・不明確のみ `EvidenceGatewayAgent` を起動  
4. 財務・資本領域のみ `FinanceCapitalEvidenceRoutingAgent` 経由でWorkerを選定（必要最小）  
5. `action=post_if` のみに投稿候補化（`hold_if` は保留条件付き）  

---

## 3. 会議前と会議中の違い（顧客説明用）

| 観点 | 会議前（MeetingPrep） | 会議中（InMeeting） |
| --- | --- | --- |
| 主目的 | 次回会議を回すための「準備品質」を高める | 会議進行中に判断逸脱を即時検知する |
| 時間制約 | 完全網羅寄り（`time_budget_minutes`は原則`null`） | `LowLatency`（最小実行） |
| 主要入力 | 過去議事録、社内データ、公開情報 | 文字起こし、事前`agenda_plus_facts` |
| 主要出力 | `agenda_plus_facts`、`fact_bundle`、`evidence_gaps` | `alerts` |
| 補足観点 | `open_items`の明示が必須寄り | 投稿可能性（`action=post_if`）の選別が必須 |

## 4. 参考ファイル

- `meeting-plus_facts` 実サンプル  
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/agenda/agenda_plus_facts_v2_20260224.md`
- 会議中監査実サンプル  
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/alerts/inmeeting_alerts_v2_20260224.json`
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/transcript/論理飛躍監査_チャット質問/20260224_議事録_SmartHR_30分_論理飛躍_チャット投稿質問_v2.md`

> 注記: `MeetingPrepLeaderAgent` は主要設計書で責務（重複排除・優先順位付け・出力整形）が定義されており、現在は専用プロンプトファイルが別管理のため、設計整合として本書では上記名称と責務で固定しています。
