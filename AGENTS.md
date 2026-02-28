# リポジトリ運用ガイドライン

## プロジェクト構成とモジュールの整理
本リポジトリは、取締役会エージェントのモック検証に特化した文書・データのワークスペースです。

- `docs/board-agent/`: 設計ノート（企画、原則、仕様）
- `docs/board-agent/prompts/`: 各Agent実装用プロンプト（v1.2 + v2）
- `data/board-agent-mock/meetings/`: 議事録・取締役会シミュレーション資料
- `data/board-agent-mock/golden_cases/`: シナリオ検証用の期待出力データ
- `data/board-agent-mock/kpi/`: KPIデータセット
- `data/board-agent-mock/public/`: 公開情報など外部シグナル
- `data/board-agent-mock/runbook/`: 手動QA・テスト手順
- `data/board-agent-mock/README.md`: モック資産の読み取りハンドブック（Codex/Claude向け起点）
- `AGENTS.md`: 本ワークスペース専用の進行メモ
- `docs/board-agent/board-agent-finalized-guide-v1.2.md`: 初見向けの利用手順（会議前→会議中）

ドメイン別・種別を明確にしたファイル命名を採用してください（例: `meetings/board-minutes-2026-01.md`, `golden_cases/board-input-expected-outputs.json`）。

## このディレクトリを初めて見る時の必読（Codex/Claude向け）

本ディレクトリのゴールは2つです。

- 役割1: 会議終了後の「次回会議準備」(MeetingPrep)
  - 過去議事録・KPI・公開情報を集約し、次回会議のアジェンダを作成する
  - 各アジェンダ項目に対して、必要情報（社内情報、公開情報、未確定論点）を提示する
  - 参照先: `data/board-agent-mock/meetings/agenda/`, `data/board-agent-mock/meetings/minutes/`, `data/board-agent-mock/meetings/registry/board-meeting-index.json`, `data/board-agent-mock/smarthr/meta/smarthr-evidence-map.json`

- 役割2: 会議中の「論理破綻監査」（InMeeting）
  - 発言を受けて、前提誤認・因果飛躍・定義不一致・期限/責任未定義・外部環境無視などを即時検知
  - 根拠付きで（`evidence_id`, `evidence_source`, `確認質問`）Teams向けの指摘を生成する
  - 参照先: `data/board-agent-mock/meetings/alerts/`, `data/board-agent-mock/meetings/inmeeting-critic-guide-v1.2.md`, `data/board-agent-mock/meetings/transcript/`, `data/board-agent-mock/smarthr/`
  - 検証手順（固定）:
    1. 文字起こしを時系列で確認する
    2. 事前に作成した `agenda_plus_facts` と `fact_bundle` を突合し、発言の前提・因果・定義・期限・責任に飛躍や破綻がないか確認する
    3. Teams（または会議チャット）に投入する判断になった場合のみ、質問文に整形して投稿する
  - 各投稿は最低限、以下を併記する
    - 参照した文字起こしの行（対象時刻）
    - 根拠データ（`evidence_id`, `evidence_source`）※未確定時は `action=hold_if`
    - 回答を促す確認質問

## v2アーキテクチャ運用前提

- 会議前と会議中は `MeetingPrepOrchestratorAgentV2` と `InMeetingOrchestratorAgentV2` に分離して運用する。
- 会議前エージェントは `meeting_scope=pre` の前提で、原則タイムアウトなしで網羅的ファクト収集を実施する。
- 会議中エージェントは `urgency_profile=LowLatency`。追加検索は高リスク時・未確定時のみ実行。
- Premise Question Agent は次を必ず同時検証する。
  - 事前作成ファクト
  - 会議中発話（`speaker/utterance_id/line/time`）
- 投稿候補は `alert`/`action` を必須付きで出す。
  - `severity`: `High/Medium`
  - `issue_type`: `logical_jump / premise_question / evidence_gap`
  - `action`: `post_if` または `hold_if`
- SmartHRダミーデータ（`data/board-agent-mock/smarthr/*`）は読み取り専用前提。編集対象外。

### まず確認すべき順番

1. `data/board-agent-mock/meetings/README.md`  
   - ディレクトリ構成と運用ルールを把握
2. `docs/board-agent/design_philosophy_v1.md`  
   - 役割分離（v2を含む）の前提と全体設計を把握
3. `docs/board-agent/prompts/`  
   - v2プロンプト（`*-v2.md`）を確認
3. `data/board-agent-mock/meetings/board-minutes-meta.json` + `data/board-agent-mock/meetings/registry/board-meeting-index.json`  
   - 会議IDと素材の対応を確定
4. 役割実行（v2推奨）
   - 会議前: `agenda/` `minutes/` `public/` `kpi/` を使って `agenda_plus_facts` を作る
   - 会議中: `transcript/` と `agenda_plus_facts` を使って `alerts` を生成

### 会議監査の必須アウトプット（最小要件）

- `agenda`: 優先度付きの次回会議議題
- `open_items`/未解決: 前回から継続課題
- `evidence_gap`: 根拠不足の主張をエビデンスIDで明示
- `alert`（役割2）: High/Mediumで分類した論理破綻指摘
- 全ての監査コメントは `evidence_id` と `evidence_source`、`確認質問` を同時に持つこと
- `evidence_gaps` は会議前・会議中で共通スキーマを使うこと
- 併せて、指摘ごとに「どの情報を参照しているか」を明示する（文字起こし行 + 社内/公開データの場所）。

## ビルド・実行・開発コマンド
本リポジトリはアプリケーションコードをまだ持たないため、既存のビルドパイプラインはありません。

以下を標準的に使って内容を確認してください。
- `rg --files data/board-agent-mock`
- `sed -n '1,200p' data/board-agent-mock/runbook/test-scenarios.md`

将来データ駆動スクリプトを追加する場合は、ここに再現可能な実行手順を都度追記してください。

## コーディングスタイルと命名ルール

- Markdown/JSON は UTF-8 前提で作成する。
- Markdown は見出しを明示し、短いセクションとチェックリスト形式の箇条書きを優先する。
- JSON は 2 スペースインデントを基本とし、実行系ツールで必要な場合を除き末尾カンマは使わない。
- ID とスラッグは一貫させる（例: `M-YYYY-NN`, `case-xx-01`, `PR-...`）。
- 参照しやすい命名規則を徹底する。
  - 議事録: `board-minutes-YYYY-*.md`
  - 公開データ: 種別が分かる簡潔な英数字名

## テスト（検証）ガイドライン

現在はシナリオベースの手動検証が中心です。
- 主要参照: `data/board-agent-mock/runbook/test-scenarios.md`
- ゴールデンデータ: `data/board-agent-mock/golden_cases/board-input-expected-outputs.json`

Runbookの手順を順に実施し、結果はPRノートに記録してください。

### 2/22（7回目）までの監査フロー（再利用手順）

- 事前準備:
  - `data/board-agent-mock/meetings/index/` 系は `board-meeting-index.json` で2/22までの `latest` を確認
  - `runbook/test-scenarios.md` のv2手順で `2月22日までの議事録を入力` を選択
- 会議前（v2）:
  - `data/board-agent-mock/meetings/minutes/` から2026-01-06〜2026-02-22を対象
  - `agenda_plus_facts_v2_YYYYMMDD.md/json` を作成
- 会議中（v2）:
  - `transcript/`（partial/complete）をタイムライン順で読み、`inmeeting_alerts_v2_YYYYMMDD.json` を作成
  - `alerts/論理飛躍監査_チャット質問/` へ質問原稿を保存
- 検証:
  - ゴールデンケース `case-v2-prep-01` と `case-v2-meeting-01` の条件に照らし合わせる

## コミット・プルリクエスト運用

- 現在の実行環境では当該ワークスペースが git リポジトリ化されていないため、コミット履歴は取得できません。
- 実リポジトリへ反映する際は Conventional Commits を推奨します。
  - 例: `feat:`, `fix:`, `docs:`, `chore:` + 簡潔なスコープ
  - 例: `docs: add meeting fixture for agenda validation`
- PR には以下を必ず含める。
  - 変更内容サマリ
  - 影響ファイル
  - 検証結果（Runbook手順 + pass/fail）
  - 新規データや前提条件、スキーマ変更の明示

## セキュリティ・データ取扱い

- `data/board-agent-mock/` 配下は、明示的に公開指定しない限り、機密性がある想定の合成データとして扱う。
- 外部情報を追加する場合は、固有情報の混入や機微情報の含有がないかを確認し、必要なら匿名化する。

---
# 進行メモ（2026-02-24）

- 本チャットで作成するファイルは、原則として `/Users/rikuuchida/Documents/20260224_prepare` 配下にのみ保存する。
- 以前に作成した `TC01_...` 系の議事録ファイルは、参照用として下記場所へ複製した。
  - `/Users/rikuuchida/Documents/20260224_prepare/data/board-agent-mock/meetings/minutes/`
- 今回のSmartHR想定取締役会議事録（7回＋2/22週を含む想定）を保存済み。
- 設計・ダミーデータ（既存）は同ディレクトリの `docs/board-agent/` と `data/board-agent-mock/` に保存済み。
- 作業者向け注記: 今後の続きはこのワークスペース内（上記ルート）でのみ編集。
- `data/board-agent-mock/meetings/` は以下構成で整理済み:
  - `agenda/`: 次回会議向けアジェンダ
  - `minutes/`: 過去議事録（2026-01-06〜2026-02-22）
  - `transcript/`: 文字起こし
    - `partial/`: 途中文字起こし
    - `complete/`: 全会議分文字起こし
    - `論理飛躍監査_チャット質問/`: 議論を進めるための指摘原稿（質問文）
  - `alerts/`: 会議中監査（指摘文）を日付ベースで保管
  - `registry/`: 会議素材索引（`board-meeting-index.json`）
  - `meta/`: 会議素材運用ルール（`meeting-asset-guidelines.md`）
  - `inmeeting-critic-guide-v1.2.md`: 会議中監査の運用ガイド（汎用）
  - `_legacy/`: 旧フォルダ退避（`smarthr-...`）
- 2026-02-24: 論理の飛躍監査（2番目の役割）向けのチャット用質問原稿は、
  `data/board-agent-mock/meetings/transcript/論理飛躍監査_チャット質問/20260224_議事録_SmartHR_30分_論理飛躍_チャット投稿質問.md`
  へ保存済み。
- 以降の運用は、日本語を原則とし、原稿・説明文・保存ファイル名は可能な限り日本語で統一する。
- 追加: v2運用版のエンドツーエンド試験を最優先。SmartHRダミーデータ未編集前提を厳守し、会議前→会議中を独立エージェントで再現する。
