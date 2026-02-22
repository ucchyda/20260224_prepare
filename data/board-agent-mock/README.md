# Board Agent モックデータ 参照ガイド（2026-02-22）

ここでは、取締役会アシストAgentの検証に必要なファイルを、Codex/Claude Codeが最短で読み解ける形で整理しています。

## 1. 全体方針

- コア検証対象：`/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/`
- SmartHR検証の起点：`/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/smarthr/`
- 補助データは`smarthr-dummy-v1`と`smarthr_package_v1`。

## 2. 参照起点（最重要）

| タイプ | パス | 用途 |
| --- | --- | --- |
| 会議素材構成 | `meetings/README.md` | `agenda/minutes/transcript/registry/meta` の読み取り規則 |
| 会議IDと証拠辞書 | `meetings/board-minutes-meta.json` | 会議ID、内部/公開参照、RACI・公開キーの紐付け |
| SmartHR証拠辞書 | `smarthr/meta/smarthr-data-catalog-v1.md` | 論点別に使うデータの粒度と用途を確認 |
| SmartHR証拠マップ | `smarthr/meta/smarthr-evidence-map.json` | `evidence_id` の正式解像度情報（data_type, period, owner, verified日） |
| 検証シナリオ | `runbook/test-scenarios.md`, `runbook/test-scenarios-smarthr.md` | 実装前の受け入れ観点 |
| 期待検知 | `golden_cases/board-input-expected-outputs-smarthr-v1.json` | 会議中監査の期待アラート・必須属性 |

## 3. 直近会議の起点（SM-2026-02-22前提）

| 会議 | 議事録 |
| --- | --- |
| SM-2026-01-06 | `meetings/minutes/20260106_議事録_SmartHR.md` |
| SM-2026-01-13 | `meetings/minutes/20260113_議事録_SmartHR.md` |
| SM-2026-01-20 | `meetings/minutes/20260120_議事録_SmartHR.md` |
| SM-2026-01-27 | `meetings/minutes/20260127_議事録_SmartHR.md` |
| SM-2026-02-03 | `meetings/minutes/20260203_議事録_SmartHR.md` |
| SM-2026-02-10 | `meetings/minutes/20260210_議事録_SmartHR.md` |
| SM-2026-02-17 | `meetings/minutes/20260217_議事録_SmartHR.md` |
| SM-2026-02-22 | `meetings/minutes/20260222_議事録_SmartHR.md` |

## 4. 証拠群（カテゴリ別）

### 財務
- `smarthr/finance/balance-sheet-smarthr-2026-02.json`
- `smarthr/finance/income-statement-smarthr-2026-02.json`
- `smarthr/finance/cash-flow-smarthr-2026-02.json`
- `smarthr/finance/treasury-runway-13w-2026-01.json`
- `smarthr/finance/debt-maturity-vs-covenants-smarthr-2026-02.json`
- `smarthr/finance/capital-allocation-pipeline-smarthr-2026-q1.json`
- `smarthr/finance/forecast-vs-actual-q4-2025-to-q1-2026.json`

### 組織・人事
- `smarthr/org/org-chart-2026-02.json`
- `smarthr/org/owner-deadline-matrix-2026-q1.json`
- `smarthr/org/compensation-and-headcount-2026.json`

### 業務・運用・法務
- `smarthr/sales/customer-economic-metrics-2026-q1.json`
- `smarthr/operations/sla-incident-regression-smarthr-2026-q1.json`
- `smarthr/compliance/compliance-audit-findings-smarthr-2026-q1.json`
- `smarthr/compliance/partner-contract-legal-conditions-smarthr-2026.json`
- `smarthr/product/roadmap-v2-smarthr-2026-q1.json`

### 公開シグナル
- `public/candidate-press-release.json`
- `public/news-feed-2026-02.json`
- `smarthr/public/smarthr-news-2026-to-2025.json`

## 5. 実装時の最短チェック（Codex向け）

1. `board-minutes-meta.json` から SmartHR会議ID群を取得。  
2. `smarthr-evidence-map.json` / `smarthr-data-catalog-v1.md` でevidence_id整合を確認。  
3. `test-scenarios.md` → `smarthr-prep` から `smarthr-inv` まで順に再現。  
4. 各アラートに `evidence_id` `evidence_source` `question/required_follow_up` があるかを必ず検査。

## 6. 期待する最終状態

- 会議前: 次回議題7件以上（または運用ルール上の上限）を安定生成
- 会議中: High/Mediumが重要観点を網羅し、優先順位が崩れない
- 投資ケース（`investment-diligence`）を追加しても既存カテゴリ互換を維持

---

更新日: 2026-02-22
