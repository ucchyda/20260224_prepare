# SmartHR想定 会議Agent用データカタログ（v1）

- **対象**: 2026-01〜2026-03想定の取締役会検証
- **更新日**: 2026-02-22
- **版**: smarthr-data-catalog-v1.0
- **運用原則**: 参照優先順位は `evidence_id` > ファイル名 > タイムスタンプ

## 1. 参照キー設計
- **evidence_id**: 主要データは実データ種別略称を使う（例: `BS-2026-02`, `IS-2026-02`, `TREAS-13W-2026-01`）
- **meeting_id**: `SM-YYYY-MM-DD` 形式（`2026-01-06` など）
- **evidence_source**: `data/board-agent-mock/.../` への相対パス
- **required**: 会議前後で根拠参照を必須化するため、各議事録テーマは最低1件以上参照

## 2. データ一覧（v1）

### 財務（finance）
- `balance-sheet-smarthr-2026-02.json`
  - ID: `BS-2026-02`
  - 粒度: 月次（期末 2026-02-28）
  - 用途: 流動性、レバレッジ、資本政策のベースライン
- `income-statement-smarthr-2026-02.json`
  - ID: `IS-2026-02`
  - 粒度: 2か月期（2026-01-01〜2026-02-28）
  - 用途: 利益性、粗利、販管費、収益品質
- `cash-flow-smarthr-2026-02.json`
  - ID: `CF-2026-02`
  - 粒度: 2か月期（2026-01-01〜2026-02-28）
  - 用途: 営業CF、自由CF、資金繰り再現性
- `treasury-runway-13w-2026-01.json`
  - ID: `TREAS-13W-2026-01`
  - 粒度: 週次（13週）
  - 用途: 短期資金ショートリスク、未使用コミット線
- `debt-maturity-vs-covenants-smarthr-2026-02.json`
  - ID: `DEBT-2026-02`
  - 粒度: 3年到来年先までの満期・条項
  - 用途: 満期集中、コベナンツ逸脱、再編計画
- `capital-allocation-pipeline-smarthr-2026-q1.json`
  - ID: `CAPAL-2026-Q1`
  - 粒度: 3〜12か月
  - 用途: 投資/還元/運用配分の優先度評価
- `forecast-vs-actual-q4-2025-to-q1-2026.json`
  - ID: `FVA-2025Q4-2026Q1`
  - 粒度: 四半期
  - 用途: 予実差分、是正計画、ボトルネックの特定

### 組織（org）
- `org-chart-2026-02.json`
  - ID: `ORG-2026-02`
  - 粒度: 月次スナップショット
  - 用途: RACI確認、意思決定責任者の確認
- `owner-deadline-matrix-2026-q1.json`
  - ID: `RACI-2026-Q1`
  - 粒度: 取締役会案件横断
  - 用途: 未解決項目・責任者・期限の強制突合
- `compensation-and-headcount-2026.json`
  - ID: `HR-HC-2026-02`
  - 粒度: 月次
  - 用途: 人員需給、採用/離職の論点、母集団定義検証

### 事業・顧客（sales）
- `customer-economic-metrics-2026-q1.json`
  - ID: `SALES-ECON-2026-Q1`
  - 粒度: 週次入力＋四半期集計
  - 用途: ARR/NRR/解約率、チャーンの根拠検証

### 運用（operations）
- `sla-incident-regression-smarthr-2026-q1.json`
  - ID: `OPS-SLA-2026-Q1`
  - 粒度: 週次・障害単位
  - 用途: 監査再現、SLA逸脱、夜間バッチ劣化

### プロダクト（product）
- `roadmap-v2-smarthr-2026-q1.json`
  - ID: `ROADMAP-2026-Q1`
  - 粒度: 四半期
  - 用途: 仕様前提、説明可能性、実装前提と監査観点

### 法務・監査（compliance）
- `compliance-audit-findings-smarthr-2026-q1.json`
  - ID: `COMPLY-AUDIT-2026-Q1`
  - 粒度: 四半期
  - 用途: 未完了指摘事項、是正期限、リスク評価
- `partner-contract-legal-conditions-smarthr-2026.json`
  - ID: `PARTNER-LEGAL-2026`
  - 粒度: 契約別更新時
  - 用途: 第三者監査、同席権、保存条件の整合

### 公開情報（public）
- `smarthr-news-2026-to-2025.json`
  - ID: `PUB-NEWS-SMARTHR`
  - 粒度: 年内主要イベント
  - 用途: 規制・景況・競合比較での会議前警鐘

## 3. タイムラインと参照ルール
- 取締役会前準備（準備フェーズ）: 少なくとも**当該週の議事録2回分+公開情報3件以上+財務KPI主要3種類**を参照
- 会議中監査: `evidence_id`が未指定の主張は優先警告対象
- 参照不整合時: 同名指標でも期間/母集団が一致しない場合は`definition_mismatch`

## 4. 更新方針
- 月次で新規数値を更新し、`version`を上げる
- 追加フィールドは末尾に拡張し、既存キーを変更しない
- 新規証跡は必ず `smarthr-evidence-map.json` に登録してから参照
