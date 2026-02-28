# Board Agent v1.2 / v2 ルールベース検証シナリオ

## 前提
- データルート: `data/board-agent-mock/`
- 設計文書: `docs/board-agent/design_philosophy_v1.md`
- 受入目標: 会議前準備と会議中監査（v2分離）を再現可能にする

## シナリオ1: 会議前準備品質（v1現行）
- **目的**: 過去2回分議事録＋公開情報を統合し、次回アジェンダを優先度付きで作成できるか確認する。
- **入力**: `board-minutes-2026-01.md`, `board-minutes-2026-02-draft.md`, `public/news-feed-2026-02.json`, `kpi/kpi-series.json`
- **期待結果**: 6件以上
- **合格条件**:
  - 各提案に `evidence_id` を1件以上付与
  - 重要度（High/Medium/Low）を付与

## シナリオ2: 会議中監査（論理検出）
- **目的**: 断定的発言に対する基本アラートが再現できるか確認する。
- **入力**: `golden_cases/board-input-expected-outputs.json` の `case-in-meeting-01`
- **期待結果**:
  - `evidence_gap`, `causal_leap`, `definition_mismatch`, `owner_deadline_missing` を検出
- **合格条件**:
  - High以上アラートを最低2件検知

## シナリオ3: ノイズ耐性
- **目的**: 曖昧表現・長文・同義語ノイズで重要順が崩れないことを確認する。
- **入力**: `golden_cases/board-input-expected-outputs.json` の `case-noise-01`
- **期待結果**:
  - High/Medium優先順位を維持

## シナリオ4: 正常系の品質比較
- **目的**: 監査結果の誤検知を把握する。
- **入力**: `golden_cases/board-input-expected-outputs.json` の `case-prep-01`
- **期待結果**:
  - 期待順位と重要テーマの乖離を把握

## シナリオ5: SmartHR特有シナリオ（v1）
- **目的**: 財務・資金繰り・監査・提携の複合監査を確認する。
- **入力**:
  - `golden_cases/board-input-expected-outputs-smarthr-v1.json`
  - `smarthr/` 配下のデータ

## シナリオ6: v1.2カテゴリ網羅
- **目的**: 8観点で検知可否を確認する。
- **入力**: `golden_cases/board-input-expected-outputs-smarthr-v1.json` の `smarthr-meeting-03`〜`smarthr-meeting-11`
- **期待結果**:
  - 各観点で少なくとも1件以上検知

## シナリオ7: v1.2投資シナリオ
- **目的**: 投資監査（NPV/IRR/感度）を再現する。
- **入力**: `smarthr-inv-01`〜`smarthr-inv-03`
- **期待結果**:
  - High/Medium合計3件以上検知

## シナリオ8: 会議前v2分離運用
- **目的**: `MeetingPrepOrchestratorAgentV2` 単独で `agenda_plus_facts` を作成し、`v2_assets` 参照に接続できる。
- **入力**:
  - `meetings/registry/board-meeting-index.json` の `SM-2026-02-24`
  - `meetings/board-minutes-meta.json`
  - 全社内情報（minutes/kpi）
- **期待結果**:
  - `agenda_plus_facts_v2_20260224.md/json` が最低7件要件に適合
  - `evidence_gaps` を3件以上検出
  - `open_items` を2件以上検出

## シナリオ9: 会議中v2分離運用
- **目的**: `InMeetingOrchestratorAgentV2` が `PremiseQuestionAgent` と `LogicalJumpCriticAgent` を起点に監査を実行し、会議前事実を優先参照できる。
- **入力**:
  - `agenda_plus_facts_v2_20260224.json`
  - `transcript/partial/20260224_議事録_SmartHR_30分.md`
  - `golden_cases/board-input-expected-outputs.json` の `case-in-meeting-01`
- **期待結果**:
  - 少なくとも1件 `logical_jump`、1件 `premise_question`、1件 `evidence_gap` を生成
  - 全件で `evidence_id` / `evidence_source` / `確認質問` が存在
  - 会議中追加調査は `action=hold_if` を含めて運用可能
