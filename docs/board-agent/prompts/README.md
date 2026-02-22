# Agent Prompt 一覧（v1.2）

本フォルダは、取締役会アシストAgentの実装で直接呼び出すための各プロンプト定義です。  
構成は `docs/board-agent/finance-capital-agent-team-v1.md` のAgent構成に対応し、共通の監査要件を反映しています。

## 参照方針

- まず読む順: `00-shared-contract.md` → 個別Agentプロンプト
- 実行時は、以下の3点を必ず共通前提にしてください
  - `evidence_id`, `evidence_source`, `question`（または `required_follow_up`）はアラートに必須
  - `confidence` と `impact_score` は 0.0〜1.0
  - High/Mediumの優先順は `severity > impact_score > confidence > evidence_depth` の順

## ファイル構成

| 種別 | パス |
| --- | --- |
| 共通契約 | `00-shared-contract.md` |
| コアオーケストレーション | `board-orchestrator-agent.md` |
| 参照解決 | `evidence-gateway-agent.md` |
| 財務資本統括 | `financial-capital-orchestrator.md` |
| 事前準備 | `preboard-synthesis-agent.md` |
| 会議中監査 | `inmeeting-critic-agent.md` |
| Worker（資金） | `treasury-liquidity-worker.md` |
| Worker（資本市場） | `capital-markets-intel-worker.md` |
| Worker（資本構成） | `capital-structure-worker.md` |
| Worker（資本配分） | `capital-allocation-worker.md` |
| Worker（投資監査） | `investment-diligence-worker.md` |
| Worker（ガバナンス） | `financial-risk-governance-worker.md` |
| Worker（外部比較） | `peer-benchmarking-worker.md` |

## 使い方（実装向け）

1. `financial-capital-agent-team-v1.md` で Agent の役割定義を確認  
2. `00-shared-contract.md` で出力制約を確認  
3. 監査観点に応じたAgentを並列/順次実行  
4. `BoardOrchestratorAgent` が最終サマリを統合
