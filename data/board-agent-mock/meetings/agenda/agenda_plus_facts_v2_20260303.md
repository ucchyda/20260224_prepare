# 2026-03-03 会議前アジェンダプラスファクト（v2）

- **会議ID**: SM-2026-03-03
- **作成エージェント**: MeetingPrepOrchestratorAgentV2
- **生成日**: 2026-03-10
- **対象最新議事録**: `minutes/20260227_議事録_SmartHR.md`
- **方針**: 網羅優先。2/27で残った未完了は期限優先で先頭固定

## 1) 事前サマリ（2/27の意思決定引継ぎ）

- 監査パイロットは中央値12分まで改善し、目標15分未満を達成したが、局所的な18分超過1件が残る（要是正）。
- 提携契約の改訂版は法務承認済みで、  
  ①データ保存先分離、②監査同席権（事前通知48時間）、③インシデント報告期限24時間以内を正式契約化。
- 3月予算は**暫定**として監査強化40%、採用支援30%、営業施策20%、AI実験10%で合意済み（確定版は3/3）。
- 営業・CS重複業務削減案は3/10（第2週開始）実施開始を想定し、2週間後計測を想定。
- KPI母集団定義は「契約継続6か月以上かつ月次アクティブユーザー」で統一へ更新。

## 2) アジェンダ（優先順）

### AF-01 High: 監査再現性の再確認と最終是正報告
- **目的**: 2/27の未残存1件（インデックス断片化）を3/3で是正済みか確認し、再測定結果を取締役会記録に残す
- **必要ファクト**: `COMPLY-AUDIT-2026-Q1`（`AUDIT-01`）, `OPS-SLA-2026-Q1`, `RACI-2026-Q1`
- **検討資料**:
  - `data/board-agent-mock/smarthr/operations/sla-incident-regression-smarthr-2026-q1.json`
  - `data/board-agent-mock/smarthr/compliance/compliance-audit-findings-smarthr-2026-q1.json`
  - `data/board-agent-mock/meetings/agenda/agenda_plus_facts_v2_20260224.json`
  - 2/27議事録 監査パイロット関連
- **成果物**: 再現時間（p95と局所最大値）を会議前後比較し、閾値逸脱が残る場合は是正計画再申請
- **依存開示**: `OpenItem-01`, `OpenItem-03`

### AF-02 High: 3月予算配分（暫定比率）から確定版への移行可否
- **目的**: 監査強化40% / 採用支援30% / 営業施策20% / AI実験10%の暫定比率を、財務健全性前提で正式確定する
- **必要ファクト**: `CF-2026-02`, `BS-2026-02`, `DEBT-2026-02`, `CAPAL-2026-Q1`, `TREAS-13W-2026-01`, `RACI-2026-Q1`
- **検討資料**:
  - `data/board-agent-mock/smarthr/finance/cash-flow-smarthr-2026-02.json`
  - `data/board-agent-mock/smarthr/finance/balance-sheet-smarthr-2026-02.json`
  - `data/board-agent-mock/smarthr/finance/debt-maturity-vs-covenants-smarthr-2026-02.json`
  - `data/board-agent-mock/smarthr/finance/capital-allocation-pipeline-smarthr-2026-q1.json`
  - `data/board-agent-mock/smarthr/finance/treasury-runway-13w-2026-01.json`
  - 2/27議事録: 予算配分の暫定比率と確定期限
- **成果物**: 3月配分確定版の承認／保留条件（キャッシュ・DSCR・月次実行見通し）
- **依存開示**: `OpenItem-03`, `OpenItem-04`

### AF-03 High: AI改善施策（誤通知）実験設計審議
- **目的**: 価格実験・AI改善施策の母集団混同を避ける前提で、対照群・評価指標・期間を確定
- **必要ファクト**: `SALES-ECON-2026-Q1`, `COMPLY-AUDIT-2026-Q1`, `RACI-2026-Q1`, `kpi_arr`, `kpi_nrr`
- **検討資料**:
  - `data/board-agent-mock/smarthr/sales/customer-economic-metrics-2026-q1.json`
  - `data/board-agent-mock/smarthr/compliance/compliance-audit-findings-smarthr-2026-q1.json`
  - 2/27議事録: AI改善施策未確定の再確認
- **成果物**: 実験設計書承認（or 保留理由）と翌週以降の効果観測設計

### AF-04 Medium: 人員再配置の試験運用開始準備と2週間後レビュー設計
- **目的**: 営業・CS重複業務削減計画を運用開始条件と測定ルールで合意する
- **必要ファクト**: `HR-HC-2026-02`, `RACI-2026-Q1`
- **検討資料**:
  - `data/board-agent-mock/smarthr/org/compensation-and-headcount-2026.json`
  - 2/27議事録: 3月第2週から試験開始/2週間観測
- **成果物**: 試験条件（対象範囲・除外顧客・効果指標）と再計画トリガー
- **依存開示**: `OpenItem-04`

### AF-05 Medium: KPI母集団定義と地域離職率フォーマットの統合
- **目的**: 採用KPIと顧客KPI、地域別離職率データの母集団を分離・統一し、数値比較の前提を固定する
- **必要ファクト**: `HR-HC-2026-02`, `kpi_attrition_rate`, `kpi_headcount_trend`, `kpi_arr`, `kpi_nrr`
- **検討資料**:
  - `data/board-agent-mock/smarthr/org/compensation-and-headcount-2026.json`
  - `data/board-agent-mock/kpi/kpi-series.json`
  - 2/27議事録: 地域別離職率フォーマット整備期限（3/6）
- **成果物**: 定義表（採用/顧客/地域）と初回再開版
- **依存開示**: `OpenItem-05`

### AF-06 Low: 外部環境更新の意思決定インパクト再確認
- **目的**: 競合・規制・AI説明責任論点が次回意思決定条件に与える影響を更新
- **必要ファクト**: `N-REG-2026-02-01`, `N-COMP-2026-02-01`, `N-TECH-2026-02-02`
- **検討資料**:
  - `data/board-agent-mock/public/news-feed-2026-02.json`
  - 2026年度監査資料の監査トレース要件
- **成果物**: 条件付き意思決定のリスクメモ（価格/採用/監査）

## 3) Open Items（次回までにクロスチェック）

1. `OpenItem-01` 夜間バッチ局所断片化の是正（`2026-03-03` / CTO）
2. `OpenItem-02` AI改善施策実験設計書提出（`2026-03-03` / CPO）
3. `OpenItem-03` 3月予算配分確定版作成（`2026-03-03` / CFO）
4. `OpenItem-04` 人員再配置試験運用開始準備（`2026-03-10` / COO）
5. `OpenItem-05` 地域別離職率データの統一フォーマット整備（`2026-03-06` / COO）
6. `OpenItem-06` 同意履歴版管理（`2026-02-28` 起点の進行中項目）完了報告（監査準拠）
7. `OpenItem-07` 提携契約 B社の遡及的監査同席明示要否（継続確認）

## 4) evidence gap（追加確認必須）

- `DEBT-2026-02`: DSCRストレス10%下方シナリオ（1.08）での事業資金余力と月次配分の両立確認
- `COMPLY-AUDIT-2026-Q1#AUDIT-02`: 同意文面変更履歴の版管理実装状況
- `SALES-ECON-2026-Q1` と `kpi_series` の母集団定義差分（採用母集団・顧客母集団の分離）
- 地域別離職率のフォーマット（CSV/システム連携、除外ルール、更新日）

## 5) 次回会議運営上の提出順（推奨）

1. AF-01（20分）  
2. AF-02（20分）  
3. AF-03（15分）  
4. AF-04（10分）  
5. AF-05（10分）  
6. AF-06（5分）  

**総所要**: 80分（監査上の緊急度を加味して、5分超過バッファを取る）

## 6) 事前提出必須（会議開始前）

- `OpenItem-01` 是正結果と再計測画面
- 3/3 budget確定版（CF/BS/DEBT整合表つき）
- `OpenItem-02` CPO実験設計書（対照群、評価指標、観測期間、停止条件）
- 人員再配置の試験設計（対象チーム・除外顧客・対応時間算定式）
- 地域別離職率フォーマット v1（監査追跡ID付き）

## 7) ハンドオフ情報

- `agenda_plus_facts_path`: `agenda/agenda_plus_facts_v2_20260303.json`
- `evidence_ref_common`: `COMPLY-AUDIT-2026-Q1`, `OPS-SLA-2026-Q1`, `PARTNER-LEGAL-2026`, `CF-2026-02`, `DEBT-2026-02`, `CAPAL-2026-Q1`, `HR-HC-2026-02`, `RACI-2026-Q1`
