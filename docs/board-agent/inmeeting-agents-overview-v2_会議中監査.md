# 会議中監査エージェント構成（v2）  

本書は、`data/board-agent-mock` の実運用を前提に、会議中監査（InMeeting）で使用するエージェントを日本語で整理したものです。  
英語名はそのまま採番し、隣接列で日本語の役割名を明記しています。  

## 全体像（会議中監査）

### 1) 全体の主役

| 区分 | 英語名 | 日本語名（監査での呼称） | 役割 |
| --- | --- | --- | --- |
| 主役エージェント | `InMeetingOrchestratorAgentV2` | **会議中監査司令Agent（v2）** | 文字起こしを時系列で受け取り、疑義抽出の全体統制を担当。会議中は `LowLatency` で最小限の投稿候補を作成する。 |

### 2) 会議中監査のサブエージェント（汎用）

| 英語名 | 日本語名 | 役割 | 監査観点 | 代表的な出力 |
| --- | --- | --- | --- | --- |
| `LogicalJumpCriticAgent` | **論理飛躍チェックAgent** | 断定・因果関係・定義をまたいだ主張の飛躍を検出 | `logical_jump` | `question_text`（例: 「ここはどの前提でつながりますか？」） |
| `PremiseQuestionAgent` | **前提確認質問Agent** | 参加者の発言や事前ファクトに対して、前提そのものの妥当性を確認する | `premise_question` | `question_text`（例: 「この前提は最新議事録のどこを根拠にしていますか？」） |
| `EvidenceGatewayAgent` | **根拠照合ゲートウェイ** | 監査時に必要な `evidence_id` / `evidence_source` を解決。未確定なら補足確認へ移す | `evidence_gap` の入口 | `evidence_id`, `evidence_source`, `action`（`post_if` / `hold_if`） |

### 3) 財務・資本領域のルータ + Worker

| 英語名 | 日本語名 | 役割 | 監査トリガー | 代表的な監査テーマ |
| --- | --- | --- | --- | --- |
| `FinanceCapitalEvidenceRoutingAgent` | **財務資本ルータAgent** | 財務・資本に関係する主張を適切なWorkerへ振り分ける | 資金・借入・投資・配分・規制・外部比較など | `treasury-*`, `investment-*` 系を含む金融資本領域全体 |
| `treasury-liquidity-worker` | **資金繰り監査Worker** | キャッシュ、流動性、支払い能力、短期資金制約を監査 | `liquidity`, `owner_deadline_missing` | 現金枯渇、コミット枠、短期予算余力 |
| `capital-markets-intel-worker` | **資本市場監査Worker** | 借入・金利・調達環境の前提更新有無を監査 | `definition_mismatch`, `evidence_gap` | 金利上振れ時に仮定を更新しているか |
| `capital-structure-worker` | **資本構成監査Worker** | レバレッジ、満期、格付、契約条項の整合を監査 | `definition_mismatch` | 満期管理・コベナンツ・規律違反リスク |
| `capital-allocation-worker` | **資本配分監査Worker** | 投資・還元・維持支出の配分順序や資金余力整合を監査 | `owner_deadline_missing`, `evidence_gap` | 成長投資と防御施策の優先順位衝突 |
| `investment-diligence-worker` | **投資デューデリ監査Worker** | NPV/IRR/回収期間・感度分析の妥当性を監査 | `evidence_gap`, `definition_mismatch` | 投資判断の前提、感度条件、母集団定義 |
| `financial-risk-governance-worker` | **金融リスク・統制監査Worker** | 監査条件・契約・規制・承認順守を監査 | `compliance_risk`, `owner_deadline_missing` | 監査同席、承認権限、規制遵守 |
| `peer-benchmarking-worker` | **外部比較乖離監査Worker** | 競合比較・公開情報との差分を監査 | `definition_mismatch`, `evidence_gap` | 外部比較有無、乖離が大きい主張の根拠 |

## 監査フロー（要点）

1. `InMeetingOrchestratorAgentV2` が文字起こしを受領  
2. まず `LogicalJumpCriticAgent` と `PremiseQuestionAgent` を先行実行  
3. 疑義が高リスク/曖昧なら `EvidenceGatewayAgent` で根拠解決  
4. 財務・資本主張なら `FinanceCapitalEvidenceRoutingAgent` で該当 Worker へ分岐（必要時のみ）  
5. `action=post_if` のものを監査コメント候補として最終提示  

## 期待される投稿フォーマット（最小要件）

- `severity`: `High / Medium`
- `issue_type`: `logical_jump / premise_question / evidence_gap`
- `evidence_id` と `evidence_source`
- `確認質問`（日本語で疑義が明確に読める形式）

## 備考（顧客向け説明用途）

- 本構成は、会議前との分離運用（`MeetingPrepOrchestratorAgentV2`）と合わせて v2 の設計思想に沿う。  
- 英語名は実装ID、説明は上記の日本語名で運用すると、監査対象者（経営層含む）の理解が上がる。  
