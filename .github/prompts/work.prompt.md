---
description: "PLAN.md に基づいて自分担当のタスクを自律実行する"
agent: board-manager
---

# Work — PLAN ベースの自律タスク実行

アクティブミッションの PLAN.md を読み、自分担当のタスクを依存順に実行する。

## ワークフロー

### Step 1: Pull & Goal & Registry

> Pull 前後の共通手順（Preflight / Pull / Heartbeat / Goal）は
> [pull-and-check.instructions.md](../instructions/pull-and-check.instructions.md) を参照。

1. `GOAL.md` を読んでアクティブミッション一覧を確認し、表示する
2. `registry.md` を読んで参加端末一覧を確認する
3. `hostname` で自分の端末名を取得し、registry の `last-seen` を更新する

### Step 2: PLAN.md 読み込み

各アクティブミッションの PLAN.md を読み込み、以下を抽出:

- **自分担当 (`@自分のhostname`) + 状態が `todo`** のタスク
- **`@any` で未引き取り + 状態が `todo`** のタスク → 自分が引き取る（担当を `@自分` に書き換え）
- 依存関係（依存タスクが完了していないタスクはスキップ）

表形式で表示:

| ミッション | Task # | タスク | 依存 | 実行可能 |
| ---------- | ------ | ------ | ---- | -------- |

### Step 3: 未読メッセージ確認

各アクティブミッションの `messages/` ディレクトリ内の未読メッセージ（`status: unread` かつ自分宛）を確認。

未読メッセージがあれば:

1. 内容を読んで PLAN.md の状態に反映（相手の作業結果等）
2. ルーティング判定（→ pull.prompt.md の Step 2.5 と同じ）
3. 対応・返信

### Step 4: タスク実行

実行可能なタスクを順番に:

1. PLAN.md のタスク状態を `🔲 todo` → `🔄 doing` に更新
2. PLAN.md の「タスク詳細」セクションに従ってタスクを実行
3. 実行結果を PLAN.md の「結果」列に記録:
   - 成功: `✅ <概要>`
   - 失敗: `❌ <概要>`
   - 部分成功: `⚠️ <概要>`
4. 状態を `✅ done` or `❌ failed` に更新

### Step 5: 結果評価（Evaluator）

全タスク実行後:

1. **完了条件チェック** — PLAN.md の完了条件を1つずつ検証
2. **全条件 OK** → ミッション完了:
   - GOAL.md のステータスを ✅ に更新
   - `GOAL.md` のアクティブミッションを完了済みに移動
   - 完了メッセージを相手に投稿
3. **条件未達** → 継続:
   - 失敗タスクの原因を分析
   - 代替タスクを PLAN.md に追加（具体的なコマンド付き）
   - 相手側の作業が必要ならメッセージで依頼

### Step 6: 結果メッセージ投稿

実行結果をミッションの `messages/` ディレクトリにメッセージとして投稿:

```markdown
---
from: <このPCのagent（registry.md の agent）>
to: <相手のagent（registry.md の agent）>
priority: normal
status: unread
tags: [<ミッション名>, work-result]
created: <現在時刻>
---

# Task 実行結果

## 実行したタスク

| Task # | タスク | 結果 |
| ------ | ------ | ---- |

## 次のアクション

[相手側に必要な作業があれば記載]
```

### Step 7: Commit & Push

```
git add missions/ GOAL.md registry.md
git commit -m "feat: work on <mission> tasks <N1,N2,...>"
git push origin master
```

## 注意事項

- **破壊的タスク**（設定変更、再起動等）は実行前にユーザーに確認する
- タスクが失敗しても止まらない — 代替案を考えて PLAN.md に追加する
- 相手担当のタスクは実行しない — メッセージで依頼するのみ
- 全タスク完了 or 自分側の実行可能タスクが無くなったらフローを終了する
