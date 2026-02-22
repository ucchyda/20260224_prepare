# 財務・資本特化 Agent Team（v1.2）設計（SmartHR想定）

## 0) 目的

- 既存Board Agent（会議前準備/会議中監査）の上に、財務・資本だけを扱う2階層サブチームを追加する。
- 既存カテゴリ（`evidence_gap / causal_leap / definition_mismatch / owner_deadline_missing / compliance_risk`）を保持したまま、`analysis_taxonomy` で5観点＋投資を内部監査化する。
- 投資評価を独立監査し、会議前/会議中で再利用できる「財務根拠メモ」を維持する。

## 1) Agent構成（v1.2）

### 1.1 オーケストレーション層

1. **BoardOrchestratorAgent**  
   既存コア。発言の重要テーマ抽出、アラート統合、表示抑制、優先順位付けを実行。

2. **EvidenceGatewayAgent**（新規）  
   `evidence_id` 解決、`evidence_source` 解決、欠損時の即時 `evidence_gap` 変換を担当。

3. **FinancialCapitalOrchestrator**（新規）  
   財務・資本サブチームの状態を集約し、`capital_strategy_priority` を算出。

4. **PreBoardSynthesisAgent**（既存）  
   会議前のアジェンダ候補、未解決、追加情報要求を作成。財務観点のクロスチェックを組み込む。

5. **InMeetingCriticAgent**（既存）  
   会議中の発言を主張単位で監査し、カテゴリ化されたアラートを出力。

### 1.2 財務・資本Worker（v1.2）

6. **treasury-liquidity-worker**  
   - 主担当: 資金余力（短期〜中期）  
   - 主監査: `treasury` 系の資金繰り整合、`liquidity_runway_months`、コミットライン有効性  

7. **capital-markets-intel-worker**  
   - 主担当: 資本コスト・調達市場  
   - 主監査: 調達コスト変動、金利・スプレッド変化時の前提更新、再計算要件

8. **capital-structure-worker**  
   - 主担当: 資本構成・レバレッジ  
   - 主監査: 負債満期、コベナンツ、格付け余力、同時点定義混在検知

9. **capital-allocation-worker**  
   - 主担当: 資本配分意思決定  
   - 主監査: 投資/還元/維持の価値配分、`decision_timing` と資金制約の整合

10. **investment-diligence-worker**（新規）  
   - 主担当: 投資案件評価の収益性とリスク  
   - 主監査: NPV/IRR/回収期間、シナリオ感度、条件変更時の再算定漏れ  
   - 必須サブチェック: `NPV_input_missing`, `sensitivity_missing`, `wacc_or_threshold_changed`

11. **financial-risk-governance-worker**  
   - 主担当: 下方耐性・契約/統制制約  
   - 主監査: 事後対応可否、規制・監査条件違反、実行遅延リスク

12. **peer-benchmarking-worker**（任意）  
   - 主担当: 外部比較の外れ値警戒  
   - 主監査: 市場シグナルと意思決定の乖離度、公開情報未参照の補足要否

## 2) 監査観点（5観点＋投資）

### 2.1 高位観点（analysis_taxonomy）

- `liquidity`（資金余力）
- `capital_cost`（資本コスト）
- `capital_structure`（資本構成）
- `capital_allocation`（資本配分）
- `downside_resilience`（下方耐性）
- `investment`（投資評価）

### 2.2 既存カテゴリへのマッピング

- `analysis_taxonomy: investment` の未検証は `evidence_gap` または `definition_mismatch` で出し分ける。
- `NPV_input_missing` / `sensitivity_missing` / `wacc_or_threshold_changed` は `analysis_taxonomy` メタとしてアラートへ付与する。
- `owner_deadline_missing` は `owner_deadline_missing` を優先し、`capital`/`financial` リスクと同等以上の優先度で提示する。

## 3) I/F拡張（監査実行）

### 3.1 InMeetingInput 拡張

`InMeetingInput` は既存互換を維持し、以下を任意追加。

```json
{
  "planning_horizon_months": 24,
  "policy_constraints": {
    "target_credit_rating": "A-/A",
    "dividend_policy": "配当維持",
    "investment_cap": 1200
  },
  "scenario_set": ["base", "stress_down", "stress_up"],
  "scope_level": "group",
  "scope_id": "SM-ALL",
  "threshold_profile": {
    "high_alert_runway_months": 4,
    "max_cash_gap_bps": 60
  }
}
```

### 3.2 InMeeting Critic 出力拡張

`CriticAlert` に以下追加。

- `analysis_taxonomy`
- `impact_dimension`（`financial` / `market` / `governance` / `compliance`）
- `question`
- `required_follow_up`（owner / deadline / action）

### 3.3 InMeetingOutput 拡張

`InMeetingOutput` の追加候補:

- `capital_diagnostic`
  - `financial_risk_scores`
    - `liquidity_runway_months`, `wacc_delta_bps`, `capital_structure_headroom`, `capital_allocation_value_score`, `downside_resilience_score`
  - `recommend_options`（A/B/C）
  - `decision_triggers`（再計算条件）

## 4) Dispatcher ルール（v1.2）

1. 資金繰り制約の言及は `treasury-liquidity-worker` を最優先で起動。  
2. 借入・リファイ・調達・市場環境言及は `capital-markets-intel-worker`。  
3. レバレッジ・満期・格付・借換は `capital-structure-worker`。  
4. CAPEX/M&A/開発投資/新規事業言及は `investment-diligence-worker`。  
5. 還元・自社株買い・配当・研究開発圧縮は `capital-allocation-worker`。  
6. 監査条件・契約改定・サイバー・規制リスクは `financial-risk-governance-worker`。  
7. 外部比較乖離が大きい場合にのみ `peer-benchmarking-worker`。  

## 5) 実装時の警戒ポイント

- `CAPAL-2026-Q1` と `TREAS-13W-2026-01` の期間整合が崩れた場合は同一発言で `definition_mismatch` を高優先化。  
- 投資案件が財務KPI/予実に反映されない場合は `investor_materiality` 要件不足として `evidence_gap`。  
- 影響額・責任者・期限が欠落した投資案件は `owner_deadline_missing` を優先。  
- ルール変更（監査同席、承認フロー、閾値）を受けた議論は `compliance_risk` を最低中優先で保持。  

## 6) 導入順（推奨）

1. `FinancialCapitalOrchestrator` + `treasury-liquidity-worker` + `capital-allocation-worker`  
2. `investment-diligence-worker` を追加し、投資トランスクリプトのgoldenを有効化  
3. `capital-markets-intel-worker` + `capital-structure-worker` を追加  
4. 運用に応じて `financial-risk-governance-worker` + `peer-benchmarking-worker` を追加

## 7) 受け入れしきい値（v1.2）

- High閾値: 0.70 / Medium閾値: 0.55  
- 投資特化トランスクリプトで High/Medium 合計3件以上  
- 全アラートは `evidence_id` と `evidence_source`、`question` または `required_follow_up` を付与
