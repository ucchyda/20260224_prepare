# 2026-02-24 SmartHR取締役会 事前準備アジェンダ

- **会議ID（仮）**: SM-2026-02-24
- **日付**: 2026-02-24
- **時間（案）**: 10:00-11:00
- **前提**: 2026-02-22 議事録（`20260222_議事録_SmartHR.md`）のアクションを受けて作成
- **作成日**: 2026-02-22

## 1. 本日の議事要点（2026-02-22）

### 決定
1. 価格施策は「粗利率低下0.5%以内」「解約率悪化なし」を継続条件として維持
2. 監査ログ検索時間KPIは10分以内を継続目標
3. 提携先契約に「データ保存先分離」「監査同席権」「インシデント報告期限」を条項化
4. 3月第1週に予算再配分素案を再提案

### 未解決 + 期限付きアクション（次回反映）
| 期限 | 議題 | 責任者 | 参照RACI | 根拠 |
| --- | --- | --- | --- | --- |
| 2026-02-24 | 2件顧客の監査再現時間短縮施策完了報告 | CISO | RACI-05 | `COMPLY-AUDIT-2026-Q1#AUDIT-01`, `OPS-SLA-2026-Q1` |
| 2026-02-24 | A社保存分離監査ログ実装可否の確認 | CISO | RACI-08 | `PARTNER-LEGAL-2026#A`, `COMPLY-AUDIT-2026-Q1#AUDIT-01` |
| 2026-02-26 | 提携契約改訂版の法務最終承認 | 法務 | RACI-06 | `PARTNER-LEGAL-2026`, `RACI-2026-Q1` |
| 2026-02-27 | 営業・CS配置見直し案の提出 | COO | RACI-08 | `HR-HC-2026-02`, `RACI-2026-Q1` |
| 2026-02-28 | 3月予算配分素案提出 | CFO | RACI-07 | `CAPAL-2026-Q1`, `CF-2026-02`, `DEBT-2026-02` |

## 2. 次回アジェンダ（優先順）

### 1) 監査再現性2件の改善完了確認（高）
- 目的: 2件の監査再現性遅延が10分以内へ収束したかを最終確認
- 対象: `COMPLY-AUDIT-2026-Q1#AUDIT-01`, `OPS-SLA-2026-Q1`, `TREAS-13W-2026-01`（夜間工数影響）
- 成果物: 5〜10分で読める再現フロー画面1枚 + 根拠ログ

### 2) 提携契約改訂の最終承認（高）
- 目的: 監査同席権、保存先分離、インシデント報告SLAの運用可否を最終固定
- 対象: `PARTNER-LEGAL-2026`, `COMPLY-AUDIT-2026-Q1`, `TREAS-13W-2026-01`（監査対応コストの影響）
- 成果物: 最終承認記録、未確定条項の是正有無

### 3) 3月予算配分素案の承認可否（高）
- 目的: 採用支援・労務コア・監査強化の配分判断（資金制約を反映）
- 対象: `CAPAL-2026-Q1`, `BS-2026-02`, `IS-2026-02`, `CF-2026-02`, `DEBT-2026-02`
- 成果物: 予算配分案の暫定可決/保留理由

### 4) 営業・CSの人員配置見直し（中）
- 目的: 稼働率・地域別退職率・リードタイムへの接続を確認
- 対象: `HR-HC-2026-02`, `RACI-2026-Q1`, `kpi_attrition_rate`, `kpi_headcount_trend`
- 成果物: 再編案と実施スケジュール

### 5) 価格施策の最終実行条件確認（中）
- 目的: 価格維持/停止の境界値が営業現場で説明可能かを確認
- 対象: `SALES-ECON-2026-Q1`, `CAPAL-2026-Q1`、価格条件共有資料
- 成果物: 2ページ（継続可否シナリオ）

### 6) KPI整合レビュー（中）
- 目的: 監査KPIと財務KPIの連動（監査改善→資金繰り悪化の有無）
- 対象: `kpi_arr`, `kpi_nrr`, `kpi_attrition_rate`, `OPS-SLA-2026-Q1`, `CF-2026-02`, `TREAS-13W-2026-01`
- 成果物: アラート件数、未解決件数、次回アクション

### 7) 外部環境レビュー（低）
- 目的: 規制・競合・資金状況の会議判断バイアスを排除
- 対象: `PUB-NEWS-SMARTHR`（`PUB-2025-11-01`, `PUB-2026-01-28`, `PUB-2026-02-20`）
- 成果物: 外部要因の意思決定影響度サマリ

## 3. 参照・提出資料（最低準備条件）

### 必須（高）
- `data/board-agent-mock/smarthr/finance/cash-flow-smarthr-2026-02.json`（CF-2026-02）
- `data/board-agent-mock/smarthr/finance/debt-maturity-vs-covenants-smarthr-2026-02.json`（DEBT-2026-02）
- `data/board-agent-mock/smarthr/compliance/compliance-audit-findings-smarthr-2026-q1.json`（COMPLY-AUDIT-2026-Q1）
- `data/board-agent-mock/smarthr/operations/sla-incident-regression-smarthr-2026-q1.json`（OPS-SLA-2026-Q1）
- `data/board-agent-mock/smarthr/compliance/partner-contract-legal-conditions-smarthr-2026.json`（PARTNER-LEGAL-2026）
- `data/board-agent-mock/smarthr/finance/treasury-runway-13w-2026-01.json`（TREAS-13W-2026-01）
- `data/board-agent-mock/smarthr/sales/customer-economic-metrics-2026-q1.json`（SALES-ECON-2026-Q1）
- `data/board-agent-mock/smarthr/org/owner-deadline-matrix-2026-q1.json`（RACI-2026-Q1）

### 推奨（中）
- `data/board-agent-mock/smarthr/finance/capital-allocation-pipeline-smarthr-2026-q1.json`（CAPAL-2026-Q1）
- `data/board-agent-mock/smarthr/finance/balance-sheet-smarthr-2026-02.json`（BS-2026-02）
- `data/board-agent-mock/smarthr/finance/income-statement-smarthr-2026-02.json`（IS-2026-02）
- `data/board-agent-mock/public/news-feed-2026-02.json`（N-REG-2026-02-01, N-MARKET-2026-02-01, N-COMP-2026-02-01）
- `data/board-agent-mock/smarthr/public/smarthr-news-2026-to-2025.json`（PUB-NEWS-SMARTHR）

## 4. 進行上の確認事項（チェック）
- [ ] 直近2回（SM-2026-02-17, SM-2026-02-22）の未解決事項を全項目クロスチェック済み
- [ ] 財務KPI（kpi_arr, kpi_nrr, kpi_revenue）を本日値更新として整備済み
- [ ] 提携契約の最終条項（監査同席・保存分離・インシデント報告SLA）を次回会議開始前に固定
- [ ] RACI未完了3件（RACI-05, RACI-06, RACI-07, RACI-08）の進捗を短報で更新

## 5. 参考
- 今後の監査観点に必要: `data/board-agent-mock/runbook/test-scenarios-smarthr.md`（準備品質: 2回議事録＋公開情報3件＋KPI3種）
