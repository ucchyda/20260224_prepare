# 取締役会アシストAgent 設計思想 v1.2（実装前確定版）

- **Version:** v1.2.0（実装前暫定）
- **更新方針:** v1.xは初期検証段階。監査精度とデータ辞書の実運用結果を受けて更新前提。
- **更新日時:** 2026-02-22
- **対象:** 任意企業の取締役会（業種非依存）を前提に、会社別Profileで最適化

## 1. 役割の再定義

### 1.1 役割1（会議後の準備エンジン）
1時間会議に対し、次回会議前提資料を「議事録的要約 + 情報探索 + 論点化」で作る。

- **入力**  
  - 過去議事録（決議、未解決、期限付きアクション、前提資料）
  - KPI時系列、予実、財務、資金繰り、組織、公開情報
  - 取締役会で既に約束されたルール（3か月レビュー、監査条件、予算承認条件）
- **出力**  
  - 次回アジェンダ（優先度付き）
  - エージェント前提仮説（追跡テーマ）
  - 情報不足リスト（次回会議前に補完すべきデータ）
  - 過去発言との整合/乖離チェック（クロスチェック）

### 1.2 役割2（会議中監査エンジン）
会議中は発言を主張単位で監査し、根拠の妥当性と一貫性を確認する。

- **入力**
  - 発言（speaker, timestamp, raw_text, normalized_text）
  - 役割1で作成したアジェンダ・仮説
  - 議題参照IDと証拠ID
- **処理**
  - 主張抽出 → 証拠要求 → 整合性検証 → 重要度付与 → 警報
- **出力**
  - High/Medium/Lowアラート
  - 根拠リンク付きの指摘文
  - 確認質問（意図確認・再検証要求）
  - 重要度順リスクメモ

### 1.2 役割1+2に対するv1.2拡張（財務・資本）

- `FinancialCapitalOrchestrator` と `investment-diligence-worker` を中核に、5観点（資金余力・資本コスト・資本構成・資本配分・下方耐性）と投資評価を統合する。
- 会議中アラートの監査観点は、既存カテゴリを維持しつつ `analysis_taxonomy` で内訳保持する。
- 投資ワーカー固有の監査サブカテゴリとして次を追加する（実装時は既存カテゴリへ正規化）:
  - `investment_diligence`（NPV/IRR/回収・感度検証）
  - `wacc_or_threshold_changed`（資本コスト前提の更新漏れ）
  - `sensitivity_missing`（シナリオ感度欠損）
- `owner_deadline_missing` は引き続き High 重みを優先的に扱う。

## 2. 監査観点（10観点）と実装カテゴリの対応

- `premise_drift`（前提誤認）  
  - 実装カテゴリ: `evidence_gap`
- `premise_clarity`（定義不明瞭）  
  - 実装カテゴリ: `definition_mismatch`
- `naive_reality_check`（素朴な反証）  
  - 実装カテゴリ: `causal_leap`
- `history_contradiction`（先行発言との矛盾）  
  - 実装カテゴリ: `causal_leap`
- `financial_balance_breach`（B/S・F/C整合崩れ）  
  - 実装カテゴリ: `definition_mismatch`
- `degradation_ignored`（悪化指標の無視）  
  - 実装カテゴリ: `evidence_gap`
- `milestone_gap`（3か月・期限逸脱）  
  - 実装カテゴリ: `owner_deadline_missing`
- `external_signal_omission`（外部環境の取りこぼし）  
  - 実装カテゴリ: `evidence_gap`
- `governance_rule_violation`（ルール逸脱）  
  - 実装カテゴリ: `compliance_risk`（新設）
- `investment_diligence`（投資前提の未検証）  
  - 実装カテゴリ: `evidence_gap`  
  - 補足: `NPV_input_missing`, `sensitivity_missing`, `wacc_or_threshold_changed` を最短で検知

- `analysis_taxonomy` は上記観点の監査内訳として `CriticAlert.analysis_taxonomy` に保持し、既存カテゴリは評価互換を維持。

運用上は、上記観点を「観点メタ」として残し、既存カテゴリはアラート実装との互換を維持する。

## 3. データ観測モデル（v1.1）

### 3.1 会議前入力
- `meeting_minutes`: 過去1〜8回の議事録要約（決議、未解決、期限、前提）
- `internal_data`: 財務、KPI、組織、法務、運用、営業、商品
- `public_signals`: 規制、景況、競合、提携
- `policy_profile`: ルール・権限・閾値

### 3.2 会議中入力
- トランスクリプト（主張単位）
- 当日配布資料要旨
- 会議前監査で抽出されたアジェンダ

### 3.3 監査時点の必須属性
- `meeting_id`
- `data_period`
- `evidence_id`
- `evidence_source`
- `owner_candidate` / `deadline_candidate`（該当する場合は必須）

## 4. 出力インターフェース（公開I/F）

### 4.1 会議前

```json
{
  "BoardPrepInput": {
    "meeting_ids": ["M-2026-01", "M-2026-02"],
    "internal_reference_ids": ["BS-2026-02", "RACI-2026-Q1"],
    "public_reference_ids": ["PUB-2026-02-01"],
    "analysis_window": "month"
  },
  "BoardPrepOutput": {
    "agenda_items": [...],
    "open_items": [...],
    "missing_info_requests": [...],
    "board_readiness_score": 78.5,
    "risk_matrix": {
      "financial": "medium",
      "compliance": "high",
      "market": "low",
      "governance": "high"
    }
  }
}
```

```json
{
  "AgendaItem": {
    "topic": "string",
    "priority": "High|Medium|Low",
    "reasoning": "string",
    "linked_evidence": ["evidence_id"],
    "owner_candidates": ["役員/部門名"],
    "deadline_candidate": "YYYY-MM-DD",
    "expected_decision_type": "可決|要検討|保留|追加調査"
  }
}
```

### 4.2 会議中

```json
{
  "InMeetingInput": {
    "meeting_id": "SM-2026-02-22",
    "utterance": {
      "speaker": "CFO",
      "timestamp": "00:23:41",
      "raw_text": "string",
      "normalized_text": "string"
    },
    "current_agenda_snapshot": ["agenda_id_1", "agenda_id_2"],
    "context_pack": ["evidence_id_1", "evidence_id_2"],
    "planning_horizon_months": 24,
    "policy_constraints": {
      "target_credit_rating": "A-/A",
      "capital_return_policy": "配当維持",
      "investment_cap_exposure": 1.0
    },
    "scenario_set": {
      "base": "forecast",
      "stress_down": "利回り上振れ・解約率悪化"
    },
    "scope_level": "group",
    "scope_id": "SM-GRP",
    "threshold_profile": {
      "liquidity_runway_month_min": 4,
      "max_deviation_bps": 60
    }
  },
  "CriticAlert": {
    "alert_id": "SMA-01",
    "severity": "High|Medium|Low",
    "category": "evidence_gap|causal_leap|definition_mismatch|owner_deadline_missing|compliance_risk",
    "statement": "string",
    "evidence_ids": ["BS-2026-02"],
    "evidence_source": "data/board-agent-mock/smarthr/finance/balance-sheet-smarthr-2026-02.json",
    "analysis_taxonomy": "financial_balance_breach",
    "impact_dimension": ["financial", "governance", "market"],
    "question": "string",
    "required_follow_up": {
      "type": "owner_deadline",
      "owner": "CFO",
      "deadline": "2026-02-xx"
    },
    "confidence": 0.91,
    "impact_score": 0.82
  }
}
```

### 4.3 会議中（v1.2出力拡張）

```json
{
  "InMeetingOutput": {
    "alerts": ["..."],
    "summary_risks": ["..."],
    "deferred_checks": ["..."],
    "capital_diagnostic": {
      "financial_risk_scores": {
        "liquidity_runway_months": 3.2,
        "capital_structure_headroom": 1.1,
        "downside_resilience_score": 0.61,
        "wacc_delta_bps": 45,
        "capital_allocation_value_score": 0.58,
        "investment_readiness_score": 62
      },
      "priority_tags": ["liquidity", "capital_allocation", "investment"],
      "next_checks": [
        "NPV/IRRの数式母集団確認",
        "シナリオ感度の欠損補完",
        "資金繰り再計算の有効期限確認"
      ]
    }
  }
}
```

## 5. 監査ロジック（実装順）

1. **構文・発話整形**: あいづち/間投詞除去、否定語の正規化、数字と期間の抽出。  
2. **主張抽出**: 断定、条件付き表現、期限言及、責任者言及、定量比較を分離。  
3. **参照要求生成**: 主張ごとに必須エビデンスを定義し、欠落なら`evidence_gap`。  
4. **整合判定**: 会議前資料、過去議事録、公開情報を横断して矛盾を検知。  
5. **優先順位付け**: severity + impact_score + confidence で並べ替え。  
   - High閾値: 0.70  
   - Medium閾値: 0.55  
6. **投資監査レイヤー**（v1.2）  
   - `investment_diligence_worker` が以下を必須で確認。  
   - NPV/IRR/回収期間の根拠式  
   - 感度シナリオ（上振れ/下振れ）  
   - 資本コスト・閾値更新時の再計算要件

## 6. 重要実装ルール
- 根拠不足は短い疑問文で終えず、必ず「確認質問」と「次アクション」を付与する。  
- `owner_deadline_missing` は単独でもHigh化しやすいように重み付与。  
- 財務論点（B/S, P/L, C/F, CF, 資金繰り）は同一議論内で母集団が混在しない場合、`definition_mismatch`を優先する。  
- 3か月以上前に決定した施策が実行未完了で再登場する場合、`milestone_gap`を優先して検知する。  
- 外部環境（規制/景況/競合）を無視して意思決定に直結する発言が出た場合は`external_signal_omission`を検知。  
- ルール逸脱（監査同席、情報保持期間、承認ライン等）は`compliance_risk`として最上位に残す。  

## 7. 受入条件（v1.1）
- 会議前: アジェンダ7件以上、未解決2件以上、追加情報要求3件以上。  
- 会議中: High/Mediumが期待カテゴリに対して再現され、`evidence_id` + `evidence_source` + `question`を付与。  
- ノイズ耐性: あいまい表現・長文でも優先順位が崩れないこと。  
- 逆整合: 2つ以上のカテゴリで`definition_mismatch`/`evidence_gap`が再検知できること。  

## 8. 受入条件（v1.2）
- 会議中（v1.2）で、`investment` 系ケースを含むトランスクリプトを入れた場合、High/Medium合計3件以上再現すること。  
- `InMeetingInput` の任意拡張フィールド（`planning_horizon_months` 等）を受け入れ、未提供時も既存スキーマで実行可能。  
- `InMeetingOutput` の `capital_diagnostic` が少なくとも `financial_risk_scores` を返却すること。  
- 重要アラートは `question`（確認質問）または `required_follow_up` を欠落なく返却すること。  

## 9. 変更履歴
- 2026-02-22: v1.1.0（本実装前版）を追加  
- 2026-02-22: v1.0.0 から監査観点を8分類で拡張、I/Fを明確化
- 2026-02-22: v1.2.0（実装前版）として財務・資本チーム（投資特化）と新規観点を追加

## 10. 保留事項（v1.2）
- 投資観点のしきい値（NPV最小値・IRR最低値・最低回収期間）を業態別に最適化する運用ルール
- `analysis_taxonomy` の最終コードセットの辞書化
- `capital_diagnostic` のランキング式（重み）最適化

## 11. 保留事項（v1.x）
- アラートの最終重み（severity重みづけ）最適化
- 同義語辞書と発言ノイズ耐性チューニング
- 機微情報扱いと監査ログ保持ルール（保存期間・参照権限）
