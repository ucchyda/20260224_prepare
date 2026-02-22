# 会議中監査（役割2）ガイド  
SmartHR想定 / 取締役会アシストAgent向け  

このドキュメントは、会議中の発言に対して即時に**論理の飛躍・破綻を検出して指摘する**
ための運用ルールを定義する。  
目標は「Teams 会議チャット欄へ自動投稿し、会議品質を下げることなく建設的に補強する」こと。  

## 1. 役割の定義

- 役割名: `InMeetingCriticAgent`（会議中監査）
- 入力:  
  - 発言（speaker / timestamp / raw_text / normalized_text）  
  - `current_agenda_snapshot`  
  - 参照証拠（evidence_id / evidence_source）  
  - 過去議事録（`board-minutes-meta.json` 経由）  
  - 財務・コンプラ・業務KPI・公開情報  
- 出力: `CriticAlert`（High/Medium/Low）  
- 方針:  
  - 断定・推測がある発言には「証拠の有無」を最優先で確認する  
  - 重要論点は High を優先し、Medium も含めるが Teams 投稿数は抑制する  
  - 全アラートに **evidence_id / evidence_source / 確認質問** を必須付与  

## 2. Teamsチャット投稿フォーマット

### 即時投稿テンプレート

```
[重要] (High/Medium) [監査カテゴリ]
発言: 「...」
理由: 何が論理的に不整合か（1〜2行）
根拠: evidence_id=..., evidence_source=...
確認質問: ...
必要確認: owner / data_required / 次アクション期限
```

### 投稿制御（推奨）
- 1発言あたり原則1件、必要時2件まで
- 5分以上同一カテゴリが同じ観測で重複しない場合のみ再提示
- Highは都度即時、Mediumは2件以上連続検知時のみ投稿
- 最終抑制: 1会議あたり High含めて最大5件。Highは全件表示

## 3. 監査カテゴリ（v1.2）

以下を監査カテゴリとして扱う。  
（内部では既存カテゴリへマッピング）

| 監査カテゴリ | 既存カテゴリ対応 | トリガー |
| --- | --- | --- |
| premise_drift（前提誤認） | evidence_gap / definition_mismatch | 提案の前提条件が過去資料や実績と矛盾 |
| premise_clarity（主張の中核不明瞭） | definition_mismatch / evidence_gap | 「数字はいい感じ」「前提の定義を後で見る」等 |
| naive_reality_check（即時反証） | causal_leap | 「これなら必ず〜」等、因果を飛躍して断定 |
| history_contradiction（時系列矛盾） | compliance_risk | 過去の決定と現在発言が矛盾 |
| financial_balance_breach（財務バランス崩壊） | definition_mismatch / causal_leap | BS/PL/CFの整合を無視した前提 |
| degradation_ignored（悪化指標無視） | evidence_gap / causal_leap | 指標悪化を無視して継続推進 |
| milestone_gap（3か月ルール逸脱） | owner_deadline_missing | 3か月以内の約束事項未検証 |
| external_signal_omission（外部環境無視） | evidence_gap | 規制/景況/競合を踏まえず意思決定 |
| governance_rule_violation（規程逸脱） | compliance_risk | 事前承認・監査条件・契約条項違反 |
| investment-missing-sensitivity | evidence_gap / causal_leap | 投資提案でNPV/感度/前提変更条件が欠如 |

## 4. 判定ロジック（実務的な手順）

### Step 1: 発言正規化
- あいづら、えっと、まじか等の不要トークンを除去
- 否定表現・条件付き表現を明示化  

### Step 2: 命題抽出
- 事実命題（「〜が増える」）  
- 因果命題（「〜だから〜」）  
- 期限命題（「3か月以内に〜」）  
- 責任命題（「〜がやる」）  

### Step 3: 証拠要求
- 命題ごとに以下をチェック  
  1) 参照すべき資料があるか（evidence_id 解決）  
  2) 数値定義が一致しているか（期間、母集団、単位）  
  3) 3か月ルール、承認ルール、監査ログ要件が守られているか  

### Step 4: アラート発行
- 欠けていれば High/Medium を決定  
  - 影響が資金・法務・事業継続に直結: High  
  - 追加検証必要だが影響限定: Medium  
  - 補助的な補足: Low（Teams投稿は原則抑制）  

## 5. 投稿テンプレ（SmartHR向け）

### A. 前提誤認（premise_drift）
- 例: 「来月から売上が毎月30%伸びる見込み」
- 投稿文:  
  - 「その前提は実績根拠が不足しています。`evidence_id=KPI-ARR-2025Q4-2026Q1` の定義（母集団：新規導入+更新）と整合が取れますか？」

### B. 財務整合崩れ（financial_balance_breach）
- 例: 「ARR改善が見込めるので採用数を5倍拡大」
- 投稿文:  
  - 「BS/CFと整合する資金実行能力が未確認です。`evidence_id=TREAS-13W-2026-01` に対し、現金増分と増員コストの同期間照合が必要です。」

### C. 定義不一致（definition_mismatch）
- 例: 「チャーンが改善したので解約率が下がっている」
- 投稿文:  
  - 「`degradation` と `attrition` の定義母集団が混在している可能性があります。`evidence_id=KPI-HEADCOUNT/ATTRITION` と `SALES-ECON-2026-Q1` の期間定義を再確認してください。」

### D. 期限欠落（owner_deadline_missing）
- 例: 「次回までに改善施策を回す」
- 投稿文:  
  - 「実行者と期限が未定義です。`RACI-2026-Q1` と連動して担当者/期限の明確化が必要です。」

### E. 外部環境無視（external_signal_omission）
- 例: 「競合の値下げは一時的」
- 投稿文:  
  - 「外部シグナル（`PUB-2026-02-20`）との照合が未了です。価格・解約率・SLA比較の更新が必要です。」

### F. 規程逸脱（governance_rule_violation）
- 例: 「今ここで提携条件を大枠決める」
- 投稿文:  
  - 「監査同席条件・承認フローの要件が未確認です。`COMPLY-AUDIT-2026-Q1` を参照し、実施条件を明記してください。」

## 6. Teams投稿の具体例（1分スクリプト）

1) 参加者発言: 「今月のARR改善で借入返済を見送る」  
   - 検知: financial_balance_breach（High）  
   - 投稿:
   - 「返済猶予はCF実績との突合が未確認です。`CF-2026-02` の営業CF残高と `DEBT-2026-02` の元本条件をまず提示してください。資金繰りが成立しない場合、施策前提が崩れます。」

2) 参加者発言: 「3か月前に決めた顧客接触率は来期どうでもいい」  
   - 検知: milestone_gap（High）  
   - 投稿:
   - 「前回議事録（`SM-2026-01-20`）で3か月フォロー確認が課題化されています。`meeting agenda` 上で未解決になっていないかを確認し、期限更新を確定してください。」

3) 参加者発言: 「監査ログは後で上げるので議論を進める」  
   - 検知: governance_rule_violation（High）  
   - 投稿:
   - 「監査観点は監督責任上、意思決定前の確認必須です。`COMPLY-AUDIT-2026-Q1` で定義される同席条件と証跡保全条件を先に提示ください。」

## 7. 運用開始時の最小要件（v1）

- このファイルを参照して、最低5件のHigh/Mediumアラートが再現する状態  
- いずれのアラートにも  
  - `evidence_id`  
  - `evidence_source`  
  - `確認質問`  
  が付与されること  
- Highは漏れなく提示、Mediumは実務重要度が高い順に絞り込み  

## 8. 参照更新メモ

- 会議中監査を行う場合、まず下記2点を確認してから処理開始  
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/board-minutes-meta.json`
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/smarthr/meta/smarthr-evidence-map.json`

