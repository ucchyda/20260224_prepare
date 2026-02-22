# 取締役会アシストAgent 最終化ガイド（v1.2）

このフォルダを初めて見た Codex/Claude Code が、最初に理解して実装に進めるための利用ガイドです。

## 1. まず読む順（最短オンボーディング）

### 1-1. 先に読む設計文書

| 優先 | ファイル | 目的 |
| --- | --- | --- |
| 1 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/20260224_会議エージェント設計書.md` | 2026-02-24 Prepare の運用に即した会議エージェント全体像（初見向け） |
| 2 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/design_philosophy_v1.md` | 役割、監査観点、I/F、受入条件を定義 |
| 3 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/finance-capital-agent-team-v1.md` | Agent Team とWorker分担（v1.2） |
| 4 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/design_themes.md` | 設計思想・汎用核の方向性 |

### 1-2. 次に読むデータ仕様

| 優先 | ファイル | 役割 |
| --- | --- | --- |
| 1 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/smarthr/meta/smarthr-data-catalog-v1.md` | SmartHR想定データの種類/粒度/用途を把握 |
| 2 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/smarthr/meta/smarthr-evidence-map.json` | evidence_id と参照ルール（data_type/period/owner） |
| 3 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/board-minutes-meta.json` | 会議ID↔証拠IDの接続情報（会議前・会議中の必須入力） |

### 1-3. 実装検証を開く順

| 優先 | ファイル | 役割 |
| --- | --- | --- |
| 1 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/runbook/test-scenarios.md` | 全体シナリオ（v1.2含む） |
| 2 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/runbook/test-scenarios-smarthr.md` | SmartHR向けシナリオ（A〜I、投資含む） |
| 3 | `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/golden_cases/board-input-expected-outputs-smarthr-v1.json` | 監査期待値（会議中） |

### 1-4. 実装で最初に読むプロンプト

| 優先 | ファイル | 役割 |
| --- | --- | --- |
| 1 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/00-shared-contract.md` | 全Agent共通の制約・出力規約 |
| 2 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/board-orchestrator-agent.md` | 統合制御 |
| 3 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/evidence-gateway-agent.md` | 根拠解決 |
| 4 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/inmeeting-critic-agent.md` | 会議中監査 |
| 5 | `/Users/rikuuchida/Documents/20260224_prepare/docs/board-agent/prompts/financial-capital-orchestrator.md` | 財務資本統合 |

### 1-5. 実装Agentプロンプト（v1.2）

| Agent | プロンプト | 補足 |
| --- | --- | --- |
| BoardOrchestratorAgent | `prompts/board-orchestrator-agent.md` | 優先順位統合と表示制御 |
| EvidenceGatewayAgent | `prompts/evidence-gateway-agent.md` | 証拠不足を即時アラート |
| FinancialCapitalOrchestrator | `prompts/financial-capital-orchestrator.md` | 資本戦略スコア |
| PreBoardSynthesisAgent | `prompts/preboard-synthesis-agent.md` | 次回アジェンダ生成 |
| InMeetingCriticAgent | `prompts/inmeeting-critic-agent.md` | リアルタイム監査 |
| treasury-liquidity-worker | `prompts/treasury-liquidity-worker.md` | 13週資金繰り監査 |
| capital-markets-intel-worker | `prompts/capital-markets-intel-worker.md` | 市場・金利前提監査 |
| capital-structure-worker | `prompts/capital-structure-worker.md` | 構成/コベナンツ監査 |
| capital-allocation-worker | `prompts/capital-allocation-worker.md` | 配分優先順監査 |
| investment-diligence-worker | `prompts/investment-diligence-worker.md` | NPV/IRR/感度監査 |
| financial-risk-governance-worker | `prompts/financial-risk-governance-worker.md` | 監査同席・契約リスク |
| peer-benchmarking-worker | `prompts/peer-benchmarking-worker.md` | 外部比較乖離 |

## 2. 「会議後に何ができるか」実装イメージ

### 2-1. 直近会議を起点とした次回アジェンダ生成（最重要）

対象は「最後の議事録」
`/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/minutes/20260222_議事録_SmartHR.md`
（会議ID: `SM-2026-02-22`）。

#### 期待する入力（最小）

| 種別 | 具体例 |
| --- | --- |
| 過去会議（1〜8回） | `board-minutes-meta.json` の SmartHR 会議群（`SM-2026-01-06`〜`SM-2026-02-22`） |
| 財務・運用データ | `BS-2026-02`, `IS-2026-02`, `CF-2026-02`, `TREAS-13W-2026-01`, `CAPAL-2026-Q1` |
| 組織・責任情報 | `RACI-2026-Q1`, `ORG-2026-02`, `HR-HC-2026-02` |
| 外部シグナル | `PUB-2025-11-01`, `PUB-2026-01-28`, `PUB-2026-02-10`, `PUB-2026-02-20` |

#### 期待する出力

`BoardPrepOutput` 風のフォーマットで、以下を最低限揃える。

| 項目 | 期待 |
| --- | --- |
| agenda_items | 優先順位付けされた次回議題（上位7件以上） |
| open_items | 未解決/未完了項目（2件以上） |
| missing_info_requests | 追加情報要求（3件以上） |
| linked_evidence | 各議題ごとに `evidence_id` を1件以上必ず付与 |

#### 成功判定（v1.2）

- `board-minutes-2026-01`, `board-minutes-2026-02-draft` まで使った基準を満たし、
- SmartHR向け `smarthr-meeting` 系が再現できること。

### 2-2. 「会議中」に対する監査モード

1. 発言（`speaker, timestamp, raw_text, normalized_text`）を `InMeetingInput` に変換。
2. 発言ごとに主張を抽出し、関連する `evidence_id` を参照。
3. `evidence_gap / causal_leap / definition_mismatch / owner_deadline_missing / compliance_risk` を分類。
4. `analysis_taxonomy` へ観点（例: `financial_balance_breach`, `investment`, `governance_rule_violation`）を付与。
5. High/Medium 優先度で提示し、`question` と `required_follow_up` を必ず付与。

#### 会議中監査の期待（SmartHR v1.2）

| 観点 | 監査対象例 | 期待される検知 |
| --- | --- | --- |
| 財務整合 | 資金繰りと配分判断の同時進行 | `definition_mismatch` / `evidence_gap` |
| 投資評価 | NPV・IRR・感度不足 | `evidence_gap`（`NPV_input_missing` 系） |
| 期限管理 | 3か月ルール逸脱 | `owner_deadline_missing` |
| ルール準拠 | 監査同席や契約条項の無視 | `compliance_risk` |

### 2-3. Golden Cases での最低再現条件

`smarthr-inv-01` ～ `smarthr-inv-03` が主観検証の最短基準です。

| 条件 | 条件値 |
| --- | --- |
| 高/中アラート再現 | 3件以上 |
| `analysis_taxonomy` 補足 | High/Medium件で存在 |
| 重要属性 | `evidence_id`, `evidence_source`, `question/required_follow_up` |

## 3. Agent Team（構築順）

### 3-1. v1.2推奨のAgent構成

| 役割 | Agent名 | 役割 |
| --- | --- | --- |
| 中核 | BoardOrchestratorAgent | アラート統合、優先順位、抑制/提示制御 |
| 中核 | EvidenceGatewayAgent | evidence解決、欠損の即時 `evidence_gap` 化 |
| 中核 | FinancialCapitalOrchestrator | 財務・資本ロジックの統合評価 |
| 中核 | PreBoardSynthesisAgent | 会議前のアジェンダ作成と不足情報抽出 |
| 中核 | InMeetingCriticAgent | 発言監査の主判断 |
| Worker | treasury-liquidity-worker | 13週資金繰り・短期安全余力 |
| Worker | capital-markets-intel-worker | 資本コスト・調達市場前提 |
| Worker | capital-structure-worker | 満期・コベナンツ・レバレッジ |
| Worker | capital-allocation-worker | 投資/維持/還元配分の整合 |
| Worker | investment-diligence-worker | NPV/IRR/感度・再計算要件 |
| Worker | financial-risk-governance-worker | 監査・契約・承認・規制適合 |

### 3-2. 構築時の実運用ルール（最小）

1. `analysis_taxonomy` を内部メタに保持、カテゴリ互換は現行カテゴリを維持。
2. `owner_deadline_missing` は可能ならHigh化。
3. 財務系の母集団不一致は最優先で `definition_mismatch`。
4. `question` がない指摘は出さない。

## 4. 開始直後の実行手順（Codex向け）

1. 会議IDとevidenceの対応を読む：`board-minutes-meta.json`
2. 最終議事録 `SM-2026-02-22` と直近KPIを拾い、`InMeeting` or `PreBoard` のどちらを先に検証するかを決定。
3. まず「会議前」シナリオ（シナリオ1〜2）で `agenda_items/open_items/missing_info_requests` を再現。
4. 次に `smarthr-meeting-01`、`smarthr-inv-01` 系で会議中監査。

## 5. ユーザー向け：このディレクトリでできること

- 次回会議の「準備資料」が自動で作れます（優先度付きアジェンダ・未完了・追加確認）
- 会議中に、発言ごとに「根拠不足」「飛躍」「定義不一致」「期限・責任の欠落」「ルール逸脱」を可視化できます
- 投資ディスカッションは `investment-diligence-worker` でNPV・IRR・感度まで監査できます

必要に応じて次は、以下を同時に追加するのが有効です。

- 追加のGolden Case（実データ差分）
- 社内データの更新（`smarthr-evidence-map.json` の hash 更新）
- Agent Teamの実装テスト時に使う `InMeetingInput` サンプル

## 6. 重要な注意点（運用）

- 主要な参照元は `data/board-agent-mock/smarthr/` と `data/board-agent-mock/meetings`。
- `smarthr-dummy-v1` と `smarthr_package_v1` は補助保管。最終判定の起点には `smarthr/` を優先。
- `evidence_id` がない主張は、原則 `evidence_gap` に寄せる方針。
