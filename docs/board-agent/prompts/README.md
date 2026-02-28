# Agent Prompt 一覧（v2運用追加）

本フォルダは、取締役会アシストAgent実装で直接呼び出すためのプロンプト定義です。  
v1系は既存互換、v2系は会議前/会議中の主役分離を明示します。

## 参照方針

- まず読む順: `00-shared-contract.md` → 目的別プロンプト
- 高優先条件:
  - `evidence_id`, `evidence_source`, `確認質問` は投稿候補に必須
  - 高頻度処理は `evidence_id` 参照を `High` で優先
  - v2会議中は `action=post_if` のみを投稿候補化

## ファイル構成

### v1（従来）

- `00-shared-contract.md`
- `board-orchestrator-agent.md`
- `evidence-gateway-agent.md`
- `financial-capital-orchestrator.md`
- `preboard-synthesis-agent.md`
- `inmeeting-critic-agent.md`
- `treasury-liquidity-worker.md`
- `capital-markets-intel-worker.md`
- `capital-structure-worker.md`
- `capital-allocation-worker.md`
- `investment-diligence-worker.md`
- `financial-risk-governance-worker.md`
- `peer-benchmarking-worker.md`

### v2（今回の更新）

- `meeting-prep-orchestrator-agent-v2.md`
- `internal-info-collector-agent-v2.md`
- `public-research-agent-v2.md`
- `fact-coverage-review-agent-v2.md`
- `inmeeting-orchestrator-agent-v2.md`
- `logical-jump-critic-agent-v2.md`
- `premise-question-agent-v2.md`
- `evidence-gateway-agent-v2.md`
- `finance-capital-evidence-routing-agent.md`

## 使い方（実装向け）

1. v1ベースの実装者はまず `design_philosophy_v1.md` と本ガイドを読み、v2運用時は `MeetingPrepOrchestratorAgentV2` と `InMeetingOrchestratorAgentV2` のみを主役として差し替える
2. 会議前シーンでは `meeting-prep-orchestrator-agent-v2.md` → `Internal/Public/FCR → MeetingPrepLeader` の順
3. 会議中シーンでは `inmeeting-orchestrator-agent-v2.md` → `Logical/Premise` を優先
4. 必要最小限の追加調査が必要なときのみ `evidence-gateway-agent-v2.md` と `finance-capital-evidence-routing-agent.md` を起動
