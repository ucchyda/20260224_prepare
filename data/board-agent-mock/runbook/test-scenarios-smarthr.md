# SmartHR想定 Board Agent v1.2（追加）

## 事前準備（smarthr拡張）

### シナリオA: 会議前準備品質（SmartHR）
- **目的**: 8回分議事録+内部データセット+公開情報を統合して、優先度付きアジェンダを生成できるか確認する。
- **入力**:
  - `smarthr/finance/balance-sheet-smarthr-2026-02.json`
  - `smarthr/finance/cash-flow-smarthr-2026-02.json`
  - `smarthr/finance/treasury-runway-13w-2026-01.json`
  - `smarthr/finance/debt-maturity-vs-covenants-smarthr-2026-02.json`
  - `smarthr/sales/customer-economic-metrics-2026-q1.json`
  - `smarthr/compliance/compliance-audit-findings-smarthr-2026-q1.json`
  - `smarthr/compliance/partner-contract-legal-conditions-smarthr-2026.json`
  - `smarthr/org/owner-deadline-matrix-2026-q1.json`
  - `smarthr/public/smarthr-news-2026-to-2025.json`
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-prep-01`
- **期待**:
  - アジェンダ候補7件以上
  - 未解決項目2件以上
  - High優先度の追加情報要求3件以上
- **合格条件**:
  - `evidence_id`付きで優先順位を説明できる
  - 資金繰り・監査・提携の3軸がトップ7に入る

### シナリオB: 会議中監査（財務・資金論点）
- **目的**: 財務・資金論点の断定と根拠欠如を即時検知する。
- **入力**:
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-meeting-01`
- **期待アラート**:
  - `causal_leap`, `evidence_gap`, `definition_mismatch`, `owner_deadline_missing`
  - 根拠元に `DEBT-2026-02`, `TREAS-13W-2026-01`, `RACI-2026-Q1` を含む
- **合格条件**:
  - High以上を3件以上検知
  - 各アラートに `確認質問` を保持

### シナリオC: 会議中監査（運用・監査論点）
- **目的**: 実務運用の矛盾（重大インシデント・監査同席）をHigh化する。
- **入力**:
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-meeting-02`
- **期待アラート**:
  - High 2件以上
- **合格条件**:
  - `evidence_gap` が監査台帳と一致する
  - 提携条件の未確定が可視化される

### シナリオD: ノイズ耐性
- **目的**: 曖昧表現・長文が混在しても優先順位崩れがないことを確認する。
- **入力**:
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-noise-01`
- **期待**:
  - High/Mediumの順序は維持
  - 定義不一致系の指摘が残る

### シナリオE: 逆整合チェック
- **目的**: 収益・監査・母集団定義の不一致を再検知する。
- **入力**:
  - `smarthr-meeting-01` と当該会議録の `transcript`
- **期待**:
  - `definition_mismatch` の再検知
  - `分析確認質問` が再実行される

### シナリオF: v1.1監査カテゴリ完全性
- **目的**: 8観点のカテゴリが監査トリガに残ることを確認する。
- **入力**:
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-meeting-03`〜`smarthr-meeting-11`
- **期待アラート**:
  - `analysis_taxonomy` が次を満たす:  
    `premise_drift`, `premise_clarity`, `naive_reality_check`, `history_contradiction`, `financial_balance_breach`, `degradation_ignored`, `milestone_gap`, `external_signal_omission`, `governance_rule_violation`
  - 各カテゴリの最低1件検知
- **合格条件**:
  - High/Mediumの重要度が崩れない
  - ルール逸脱系（`governance_rule_violation`）がHigh扱いで残る

### シナリオG: v1.2 財務・資本（資金繰り vs 資本配分）
- **目的**: 資金ショート予兆がある状態で改善/投資方針が矛盾していないかを検知する。
- **入力**:
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-inv-02`
- **期待アラート**:
  - `evidence_gap`（資金繰りと配分の不整合）
  - `definition_mismatch`（期首〜期末の母集団混在）
- **合格条件**:
  - High以上を2件以上
  - `analysis_taxonomy` が `financial_balance_breach` or `capital_structure` を含む

### シナリオH: v1.2 投資前提欠落
- **目的**: NPV/IRR/回収とシナリオ感度の未提示による投資監査を確認する。
- **入力**:
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-inv-01`
- **期待アラート**:
  - High/Mediumの合計3件以上
  - `question` または `required_follow_up` が全件付与
  - `analysis_taxonomy` に `investment` が含まれる
- **合格条件**:
  - `evidence_id` / `evidence_source` が欠落しない

### シナリオI: v1.2 ノイズ耐性（投資含む）
- **目的**: 曖昧語や長文が混在しても、投資・資本配分のHigh優先が崩れないことを確認する。
- **入力**:
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-inv-03`
- **期待**:
  - High優先が維持される
  - `definition_mismatch` または `causal_leap` が残る
- **合格条件**:
  - 既存の `smarthr-noise-01` と同様に、優先度順が崩れない
