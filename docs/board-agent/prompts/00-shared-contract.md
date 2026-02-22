# 共通プロンプト契約（v1.2）

## 目的
すべてのAgentに共通する監査品質と出力規約を固定する。

## 1) 回答ルール（必須）
- 各アラートは以下を必ず持つ。  
  - `alert_id`  
  - `severity`（High / Medium / Low）  
  - `category`（`evidence_gap` / `causal_leap` / `definition_mismatch` / `owner_deadline_missing` / `compliance_risk`）  
  - `statement`  
  - `evidence_ids`（配列）  
  - `evidence_source`（1件以上）  
  - `question` または `required_follow_up`  
  - `confidence`（0〜1）  
  - `impact_score`（0〜1）
- `analysis_taxonomy` は観点メタとして可能な限り付与（例: `financial_balance_breach`, `investment`, `premise_drift` など）。
- `owner`/`deadline` が分かる場合は `required_follow_up` に明記。
- `evidence_id` が見つからない主張は `evidence_gap` を最優先。
- 数値混在の比較（財務・KPI）は、期間・母集団定義の不一致時 `definition_mismatch` を優先。

## 2) 出力形の例

### InMeetingAlert
```json
{
  "alert_id": "SMA-XX",
  "severity": "High",
  "category": "evidence_gap",
  "statement": "string",
  "evidence_ids": ["BS-2026-02"],
  "evidence_source": "data/board-agent-mock/smarthr/finance/balance-sheet-smarthr-2026-02.json",
  "analysis_taxonomy": "financial_balance_breach",
  "impact_dimension": ["financial", "governance"],
  "question": "何をもって根拠を満たすと判断しますか。",
  "required_follow_up": {
    "type": "owner_deadline",
    "owner": "CFO",
    "deadline": "2026-02-28"
  },
  "confidence": 0.92,
  "impact_score": 0.87
}
```

### InMeetingOutput（最小）
```json
{
  "alerts": ["<InMeetingAlert配列>"],
  "summary_risks": [
    "短期資金繰りを満たす追加根拠が必要"
  ],
  "deferred_checks": [
    "CAPAL前提と資金繰りの期間定義再確認"
  ]
}
```

## 3) 参照優先順位（共通）
1. `evidence_id` が正確に解決できるもの  
2. `evidence_id` が解決できない場合の `evidence_gap`  
3. `analysis_taxonomy` とカテゴリ整合  
4. `owner/deadline` の有無

## 4) エスカレーション条件
- `category` が未定義、`evidence_ids` が空、`question` も `required_follow_up` も空の場合  
→ その出力を `invalid_alert` として再生成を要求
