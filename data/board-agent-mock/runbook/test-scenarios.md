# Board Agent v1.2 テストシナリオ（Runbook）

## 前提
- データルート: `data/board-agent-mock/`
- 設計ドキュメント: `docs/board-agent/design_philosophy_v1.md`
- テスト目標: 会議前準備と会議中監査（v1.2）を最小条件で再現可能にする

## シナリオ1: 会議前準備品質
- **目的**: 2回分議事録＋公開情報を統合して、次回アジェンダを優先度付きで作成できるか確認する。
- **入力**:
  - `meetings/board-minutes-2026-01.md`
  - `meetings/board-minutes-2026-02-draft.md`
  - `public/news-feed-2026-02.json`
  - `kpi/kpi-series.json`
- **期待結果**:
  - アジェンダ候補を6件以上作成
  - 未解決項目を2件以上検知
  - 追加情報要求を3件以上生成
- **合格条件**:
  - 各提案に `evidence_id` が1件以上付与
  - 重要度（High/Medium/Low）を付与

## シナリオ2: 会議中監査（論理検出）
- **目的**: 断定的発言に対する基本アラートが再現できるか確認する。
- **入力**: `golden_cases/board-input-expected-outputs.json` の `case-in-meeting-01`
- **期待結果**:
  - `evidence_gap`, `causal_leap`, `definition_mismatch`, `owner_deadline_missing` が検出
  - それぞれに `evidence_id`, `evidence_source`, `確認質問` が付与
- **合格条件**:
  - High以上アラートを最低2件検知
  - `impact_score` が `confidence` より低くならない（運用時デフォルト）

## シナリオ3: ノイズ耐性
- **目的**: 曖昧表現・長文・同義語ノイズで重要順が崩れないことを確認する。
- **入力**: `golden_cases/board-input-expected-outputs.json` の `case-noise-01`
- **期待結果**:
  - ノイズに影響されず、High/Mediumの優先順位が維持される
  - 根拠不足と定義不一致のアラートが残る

## シナリオ4: 正常系の品質比較
- **目的**: 監査結果の誤検知を把握する。
- **入力**: `golden_cases/board-input-expected-outputs.json` の `case-prep-01`
- **期待結果**:
  - 期待された議題上位順位（少なくとも上位4件）と大きな乖離がない
  - 未解決項目検出と公開情報参照の整合が成立

## シナリオ5: SmartHR特有シナリオ
- **目的**: 財務・資金繰り・監査・提携の複合監査を確認する。
- **入力**:
  - `smarthr/` 配下のデータ
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json`
- **期待結果**:
  - 会議前: アジェンダ候補7件以上、追加要求3件以上、高優先リスク3件以上
  - 会議中: High/Mediumの監査を並行取得

## シナリオ6: v1.1カテゴリ網羅シナリオ
- **目的**: 8観点（v1.1）で観点ごとの検知可否を確認する。
- **入力**: `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-meeting-03`〜`smarthr-meeting-11`
- **期待結果**:
  - `premise_drift`: 1件以上
  - `premise_clarity`: 1件以上
  - `naive_reality_check`: 1件以上
  - `history_contradiction`: 1件以上
  - `financial_balance_breach`: 1件以上
  - `degradation_ignored`: 1件以上
  - `milestone_gap`: 1件以上
  - `external_signal_omission`: 1件以上
  - `governance_rule_violation`: 1件以上
- **合格条件**:
  - 各カテゴリで `analysis_taxonomy` と `evidence_id` が明示される
  - High/Mediumの検知数が閾値（High>=3）を満たす

## シナリオ7: v1.2投資特化シナリオ
- **目的**: 投資ダイリジェンス監査を v1.2 仕様で再現する。
- **入力**: 
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-inv-01`、`smarthr-inv-02`、`smarthr-inv-03`
- **期待結果**:
  - 投資前提欠落、資金繰り矛盾、定義混在の再現
  - 各アラートに `analysis_taxonomy` と `question`/`required_follow_up` が付与
- **合格条件**:
  - High/Medium合計3件以上検知
  - `evidence_id` と `evidence_source` の欠損なし

## 実行メモ
- 実装前段階の検証資料として保管する。
- 実行ごとの結果は `board_readiness_score`、`priority`、`evidence_id` をセットで保管する。
