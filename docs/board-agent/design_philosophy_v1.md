# 取締役会アシストAgent 設計思想 v1.2（v2.0拡張版）

- **Version:** v2.0.0
- **更新日時:** 2026-02-24
- **対象:** 取締役会運用（SmartHR想定）

## 1. 役割の再定義

### 1.1 役割1（会議前準備）

- **名称**: `MeetingPrepOrchestratorAgentV2`
- **目的**: 過去議事録＋社内データ＋公開情報を最終的に網羅し、次回会議の「実行に効くアジェンダ」を作成
- **特性**: 時間制約がない前提で、再現・再調査を優先
- **追加要件**:
  - 過去会議の未解決・期限未確定項目を漏れなく抽出
  - `evidence_id` と `evidence_source` を各主張に紐付け
  - `evidence_gap` を明示し、追加確認を最短化

### 1.2 役割2（会議中監査）

- **名称**: `InMeetingOrchestratorAgentV2`
- **目的**: 文字起こしを時系列で監査し、会議進行中の判断崩れを即時アラート化
- **特性**: 会議速度優先。追加調査は高確度逸脱時のみ実施
- **追加要件**:
  - 事前アジェンダ（`agenda_plus_facts`）を最上位のコンテキストとして参照
  - 発言ごとに疑義がある場合のみ `EvidenceGatewayAgent` を呼ぶ（low-latency）
  - `evidence_id` + `evidence_source` + `確認質問` は全出力必須

## 2. v2構成（最小）

### 会議前サブエージェント

1. `InternalInfoCollectorAgent`
2. `PublicResearchAgent`
3. `FactCoverageReviewAgent`
4. `MeetingPrepLeaderAgent`

### 会議中サブエージェント

1. `LogicalJumpCriticAgent`
2. `PremiseQuestionAgent`
3. `EvidenceGatewayAgent`
4. `FinanceCapitalTeam`（7ワーカー）
   - `treasury-liquidity-worker`
   - `capital-markets-intel-worker`
   - `capital-structure-worker`
   - `capital-allocation-worker`
   - `investment-diligence-worker`
   - `financial-risk-governance-worker`
   - `peer-benchmarking-worker`

## 3. データ観測モデル（v2）

### 3.1 会議前入力

- `meeting_id`
- `meeting_scope`
- `latest_minutes_id`
- `target_minutes_candidates`
- `evidence_sources`（minutes/meta, smarthr/meta, meetings, kpi, public）
- `run_constraints`
  - `time_budget_minutes`（v2会議前: `null`）
  - `priority`（`completeness` を推奨）

### 3.2 会議中入力

- `meeting_id`
- `transcript_segment`（speaker / line / time / utterance_id / text）
- `agenda_plus_facts_path`
- `fact_bundle_ref`
- `urgency_profile`（会議中固定: `LowLatency`）

### 3.3 会議前出力（MeetingPrepOutput v2）

- `meeting_id`
- `agenda_plus_facts[]`
- `fact_bundle[]`
- `evidence_gaps[]`
- `open_items[]`
- `review_summary`

### 3.4 会議中出力（InMeetingAlertOutput v2）

- `alert_id`
- `severity`（High / Medium）
- `issue_type`（`logical_jump` / `premise_question` / `evidence_gap`）
- `trigger`（speaker / line / time / utterance_id）
- `claim_excerpt`
- `evidence`（`evidence_id` と `evidence_source`）
- `why_flagged`
- `question_text`
- `action`（`post_if` / `hold_if`）

## 4. 主要監査観点（実装互換維持）

- `causal_leap`（既存カテゴリ）
- `definition_mismatch`
- `owner_deadline_missing`
- `evidence_gap`
- `compliance_risk`
- `analysis_taxonomy` は `premise_drift / financial_balance_breach / investment_diligence / etc.` を維持して分析補助

## 5. 重要ルール

- 根拠なし断定は `evidence_gap` を最優先
- 数値定義（期間・母集団・単位）が混在する場合は `definition_mismatch` を優先
- `owner_deadline_missing` は会議中High化しやすい状態で扱う
- `PremiseQuestionAgent` は `事前ファクト` と `会議発言` の双方を疑問化

## 6. 受け入れ条件（v2）

### 会議前

- 7件以上の議題候補
- Open item 2件以上
- 情報不足要求 3件以上
- それぞれの `agenda_plus_facts` に `evidence_id` / `evidence_source` を紐付け

### 会議中

- High/Medium最低2件以上の再現（対象ケース）
- 全投稿候補で `evidence_id` + `evidence_source` + `question_text` が必須
- 会議前成果物を最優先参照し、追加調査は必要時のみ

## 7. 変更履歴（v2）

- 2026-02-24: v2.0（会議前/会議中主役分離、premise疑義の発話検証追加）
- 2026-02-22: v1.2.0（v1.2実装前暫定）

