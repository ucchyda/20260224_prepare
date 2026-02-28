# 会議中監査（役割2）ガイド（v2版）

SmartHR想定 / 取締役会アシストAgent向け  
更新日: 2026-02-24  
対象会議: 2026-02-24（v2運用）

このドキュメントは、v2主役エージェント構成下で会議中監査を実行するための実務ルールです。  
`InMeetingOrchestratorAgentV2` を中心に、`agenda_plus_facts` を最優先参照します。

## 1. 役割の定義

- 役割名: `InMeetingOrchestratorAgentV2`
- 入力:
  - `transcript_segment`（speaker / line / timestamp / utterance_id / text）
  - `agenda_plus_facts`
  - `fact_bundle_ref`
  - `urgency_profile`（LowLatency）
  - 必要時: `EvidenceGatewayAgent`, `FinanceCapitalEvidenceRoutingAgent`
- 出力: `InMeetingAlertOutput`（High / Medium）  
- 全アラート要件:
  - `evidence_id` + `evidence_source`
  - `確認質問`（末尾は `?`）
  - `source_scope`（`pre_agenda` / `internal` / `public` のいずれか）
  - `action`（`post_if` / `hold_if`）

## 2. 投稿判断の最優先順

1. まず `agenda_plus_facts` の該当項目と照合し、事実の逸脱・再定義を確認する  
2. 次に `PremiseQuestionAgent` と `LogicalJumpCriticAgent` の疑義を判定する  
3. 追加調査が本当に必要な場合のみ `EvidenceGatewayAgent` を呼ぶ  
4. 財務・資本発言は `FinanceCapitalEvidenceRoutingAgent` でルーティングして必要最小だけ実行する

## 3. 監査カテゴリ（v2）

| 監査カテゴリ | 既存カテゴリ対応 | トリガー |
| --- | --- | --- |
| logical_jump（論理飛躍） | `causal_leap` / `definition_mismatch` | 定義・母集団・期間が抜けたまま因果を確定 |
| premise_question（前提疑義） | `evidence_gap` / `definition_mismatch` | 事前アジェンダ／発言の前提が未検証 |
| evidence_gap（根拠欠損） | `evidence_gap` | 根拠ID・定義情報の解像不足 |
| owner_deadline_missing（期限・責任不在） | `owner_deadline_missing` | 実行者と期限が不在 |
| compliance_risk（規程違反） | `compliance_risk` | 監査要件・承認条件違反 |

### v2運用補足

- 警告カテゴリは `logical_jump` / `premise_question` / `evidence_gap` を優先出力する。
- `issue_type` で上記を判定し、必要時のみ `owner_deadline_missing` / `compliance_risk` を併記する。

## 4. 判定ロジック

### Step 1: 発言正規化
- `えっと`, `その`, `まあ` を除去  
- `speaker`, `line`, `time`, `utterance_id` を必ず保持

### Step 2: 事前アジェンダとの照合
- `agenda_plus_facts` の `required_facts` を満たしているか照合
- 同一主張が繰り返される場合は重複除去

### Step 3: 監査判定
- `logical_jump` と `premise_question` を同時評価
- `PremiseQuestionAgent` は  
  - 事前ファクト（`agenda_plus_facts`）
  - 現在発言（`transcript_segment`）  
  の両方を対象とする

### Step 4: 調査ゲート
- 会議中監査は低遅延優先。  
- 追加検索は以下条件のみ起動:
  - Highで `evidence_gap` が疑われる
  - 発言間で定義が矛盾
  - 数値の母集団混在が明確

## 5. 投稿テンプレート（例）

### 5-1. 論理飛躍
```
[重要] High [logical_jump]
発言: 「...」
理由: 期首→期末の定義が混在し、主張が成立していない
根拠: evidence_id=..., evidence_source=...
確認質問: どこがその前提になるので、現場運用が崩れませんか？
source_scope: internal（pre_agendaは必要時）
```

### 5-2. 前提疑義
```
[重要] Medium [premise_question]
発言: 「...」
理由: 前提の対象母集団が未定義
根拠: evidence_id=..., evidence_source=...
確認質問: その前提はどの定義に対しているので、根拠を示せますか？
source_scope: internal（pre_agenda / public は必要時）
```

- `確認質問`は必ず疑問文で終わらせる。
- 事実・主張ごとに `source_scope` を明記し、`pre_agenda` / `internal` / `public` から選定する。
- `source_scope` が曖昧な場合は優先度順で `pre_agenda` → `internal` → `public` を適用する。

## 6. 運用ルール（v2）

- 投稿制御:
  - `action=post_if` のみを会議チャットへ投稿候補化
  - `action=hold_if` は保留し、後続トランスクリプトで再評価
- 1会議あたりHigh/Mediumは通常5件上限、Highは必ず提示
- Lowは原則投稿しない
- `確認質問` は必ず `?` で終了させる。  
- `source_scope` が `pre_agenda` の場合は事前アジェンダの該当行を参照先として併記する。

## 7. 参照更新メモ

- 会議中監査起動時は次を先に確認
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/registry/board-meeting-index.json`
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/board-minutes-meta.json`
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/smarthr/meta/smarthr-evidence-map.json`
