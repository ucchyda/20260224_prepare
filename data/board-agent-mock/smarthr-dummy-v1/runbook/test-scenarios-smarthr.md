# SmartHR想定 Board Agent v1（追加）

## 事前準備（smarthr拡張）

### シナリオA: 会議前準備品質（SmartHR）
- **目的**: 8回分議事録+内部データセット+公開情報を統合し、優先度付きアジェンダを生成
- **入力**:
  - `smarthr/finance/balance-sheet-smarthr-2026-02.json`
  - `smarthr/finance/cash-flow-smarthr-2026-02.json`
  - `smarthr/finance/treasury-runway-13w-2026-01.json`
  - `smarthr/finance/debt-maturity-vs-covenants-smarthr-2026-02.json`
  - `smarthr/sales/customer-economic-metrics-2026-q1.json`
  - `smarthr/compliance/compliance-audit-findings-smarthr-2026-q1.json`
  - `smarthr/org/owner-deadline-matrix-2026-q1.json`
  - `smarthr/public/smarthr-news-2026-to-2025.json`
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-prep-01`
- **期待**:
  - アジェンダ候補7件以上
  - 未解決項目2件以上
  - High優先度の追加情報要求3件以上
- **合格条件**:
  - `evidence_id`付きで優先順位の妥当性を説明
  - 資金繰り・監査・提携の3軸が含まれること

### シナリオB: 会議中監査（財務・資金論点）
- **目的**: 断定的・根拠不足な発言を即時検知
- **入力**: `golden_cases/.../smarthr-meeting-01` の transcript
- **期待アラート**:
  - `causal_leap`, `evidence_gap`, `definition_mismatch`, `owner_deadline_missing`
  - 根拠元に `DEBT-2026-02`, `TREAS-13W-2026-01`, `RACI-2026-Q1` を含む
- **合格条件**:
  - High以上を3件以上検知
  - 各アラートに `確認質問` を保持

### シナリオC: 会議中監査（運用・監査論点）
- **目的**: 実務運用の矛盾（重大インシデント・監査同席）をHigh化
- **入力**: `smarthr-meeting-02`
- **期待アラート**: High 2件以上
- **合格条件**:
  - `evidence_gap` が監査台帳と一致し、提携条件の未確定が可視化される

### シナリオD: ノイズ耐性
- **目的**: 曖昧表現が混在しても優先順位崩れがない
- **入力**: `smarthr-noise-01`
- **期待**:
  - High/Mediumの順序は維持
  - 定義不一致系の指摘が残る

### シナリオE: 逆整合チェック
- **目的**: 収益・監査・母集団定義の不一致を引きずり出す
- **入力**: 会議中断片と `golden_cases` の `smarthr-meeting-01` 組合せ
- **期待**: definition_mismatchの再検知と追加調査質問の提示
