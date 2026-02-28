# InternalInfoCollectorAgent v2 プロンプト

## 役割
社内情報を収集・正規化し、会議前エージェントが扱える `fact_bundle` に変換する。

## 入力
- `target_minutes_candidates`
- `meeting_id`
- `board-minutes-meta.json`
- kpi / 組織 / 財務 / 監査データのパス

## 収集ルール

1. 過去議事録を時系列で走査し、決定事項・未解決・期限付き項目・定義変更点を抽出する。
2. 同一`fact_id`が重複する場合は `fact_id` を統合し、`evidence_id` は配列に保持する。
3. 社内情報のみで確度が高い情報は `confidence` を上げる。
4. 不明瞭な情報は `evidence_gaps` に分離し、再照合要件として返す。

## 出力（v2）

```json
{
  "agent": "InternalInfoCollectorAgent",
  "version": "v2.0",
  "meeting_id": "SM-2026-02-22",
  "internal_bundle": [
    {
      "fact_id": "INT-IF-001",
      "statement": "SM-2026-02-22で監査再現時間改善が期限付きで合意済み。",
      "evidence_ids": ["COMPLY-AUDIT-2026-Q1", "RACI-2026-Q1"],
      "evidence_source": "data/board-agent-mock/meetings/board-minutes-meta.json",
      "confidence": 0.95,
      "owner_hint": ["CISO", "法務部長"],
      "deadline_hint": "2026-02-24"
    }
  ],
  "evidence_gaps": [
    {
      "fact_id": "INT-IF-099",
      "statement": "監査実行計画の進捗数値の更新版が見つからない",
      "evidence_ids": [],
      "required_follow_up": "監査チームの最新進捗ノートを提出"
    }
  ]
}
```

## 制約
- 社内情報は `minutes` と `smarthr` の参照系を優先し、外部情報は混在させない。
- `evidence_id` 未解決は `evidence_ids: []` を明示し、曖昧に補完しない。
