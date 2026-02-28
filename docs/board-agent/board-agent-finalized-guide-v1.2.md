# 取締役会アシストAgent 最終化ガイド（v2.0）

このガイドは、`/Users/rikuuchida/Documents/20260224_prepare` 配下で v2.0 のエージェント構成を実行するための最短導線です。  
SmartHRのダミーデータは**読み取り前提**で、編集対象外です。

## 1. まず読む順（最短オンボーディング）

### 1-1. 先に読む設計文書

| 優先 | ファイル | 目的 |
| --- | --- | --- |
| 1 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/20260224_会議エージェント設計書.md` | v2全体像（2主要エージェント） |
| 2 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/design_philosophy_v1.md` | 役割分離、監査観点、受入条件 |
| 3 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/finance-capital-agent-team-v1.md` | 財務資本サブチーム（7ワーカー） |
| 4 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/README.md` | 実行時のプロンプト一覧 |

### 1-2. 次に読むデータ仕様

| 優先 | ファイル | 役割 |
| --- | --- | --- |
| 1 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/smarthr/meta/smarthr-data-catalog-v1.md` | SmartHR想定データの種類と粒度 |
| 2 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/smarthr/meta/smarthr-evidence-map.json` | evidenceの解決ルール |
| 3 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/board-minutes-meta.json` | 会議IDと内部参照IDs |
| 4 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/registry/board-meeting-index.json` | 会議前・会議中v2成果物の参照先 |

### 1-3. 実装検証を開く順

| 優先 | ファイル | 役割 |
| --- | --- | --- |
| 1 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/runbook/test-scenarios.md` | 変更前後の受入シナリオ |
| 2 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/golden_cases/board-input-expected-outputs.json` | 会議前/会議中v2期待値（汎用） |
| 3 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/golden_cases/board-input-expected-outputs-smarthr-v1.json` | SmartHR想定運用比較 |

## 2. v2役割分離（会議前と会議中）

### 2.1 主役エージェント

- `MeetingPrepOrchestratorAgentV2`（会議前）  
  - 目的: 過去議事録・社内情報・公開情報を横断し、`agenda_plus_facts` を生成
- `InMeetingOrchestratorAgentV2`（会議中）  
  - 目的: 文字起こしに対し即時監査。`agenda_plus_facts` を最優先参照し、必要時のみ追加情報を取得

### 2.2 会議前の構成（必須）

1. `InternalInfoCollectorAgent`
2. `PublicResearchAgent`
3. `FactCoverageReviewAgent`
4. `MeetingPrepLeaderAgent`

### 2.3 会議中の構成（必須）

1. `LogicalJumpCriticAgent`
2. `PremiseQuestionAgent`
3. `EvidenceGatewayAgent`
4. `FinanceCapitalAgent`
   - `treasury-liquidity-worker`、`capital-markets-intel-worker`、`capital-structure-worker`、`capital-allocation-worker`、`investment-diligence-worker`、`financial-risk-governance-worker`、`peer-benchmarking-worker`

## 3. 実行I/F（v2合同）

### 3.1 会議前入力 `MeetingPrepInput`（v2）

```json
{
  "meeting_id": "SM-2026-02-22",
  "meeting_scope": "board-prepare",
  "latest_minutes_id": "SM-2026-02-22",
  "target_minutes_candidates": ["SM-2026-02-22", "SM-2026-02-17", "SM-2026-02-10"],
  "evidence_sources": [
    "data/board-agent-mock/meetings/board-minutes-meta.json",
    "data/board-agent-mock/kpi"
  ],
  "run_constraints": {
    "time_budget_minutes": null,
    "priority": "completeness"
  }
}
```

### 3.2 会議中入力 `InMeetingAuditInput`（v2）

```json
{
  "meeting_id": "SM-2026-02-24",
  "transcript_segment": {
    "speaker": "CFO",
    "line": 245,
    "time": "00:13:22",
    "utterance_id": "UT-2026-02-24-245",
    "text": "..."
  },
  "agenda_plus_facts_path": "meetings/agenda/agenda_plus_facts_v2_20260224.json",
  "fact_bundle_ref": [
    "SM-2026-02-22#OF-01",
    "kpi_time_to_fill"
  ],
  "urgency_profile": "LowLatency"
}
```

### 3.3 出力 I/F（v2要約）

- 会議前: `agenda_plus_facts` + `fact_bundle` + `evidence_gaps` + `open_items`
- 会議中: `alerts`（各アラートは `issue_type` と `evidence_id`/`evidence_source`/`確認質問` 必須）

## 4. 実行手順（Codex向け）

1. 役割と参照先を把握（上記 1. のリード順）
2. `board-meeting-index.json` の `v2_assets` で当該会議のv2素材を確認
3. 会議前シナリオで `agenda_plus_facts` を生成し、 `evidence_gaps` を検査
4. 会議中シナリオで `InMeetingOrchestratorAgentV2` を起動し、文字起こしを時系列で監査
5. 投稿候補は `action=post_if` のみ出力
6. 投稿できない場合は `action=hold_if` で根拠ギャップを保留

## 5. 受け入れ条件（v2）

- 会議前が会議中から独立して実行できること
- 会議中は `agenda_plus_facts` を最優先参照し、未確定・高リスク時のみ追加検索
- `PremiseQuestionAgent` が `事前ファクト` と `会議発言` の双方を疑問化できること
- 全ての投稿候補で `evidence_id` + `evidence_source` + `確認質問` を併記
- SmartHRダミーデータ（`data/board-agent-mock/smarthr/*`）を非編集で維持

## 6. 重要運用ルール

- 1会議あたりの投稿候補は優先度制御し、High/Mediumを維持
- 同一観点の連続重複は時間間隔・主張差分で抑制
- 会議中における追加調査は高確度の逸脱が疑われる場合に限定

