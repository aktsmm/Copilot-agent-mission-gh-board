---
description: ミッション内に新しいメッセージを投稿する
agent: board-manager
---

# メッセージ投稿

アクティブミッション内に新しいメッセージを投稿します。

## 手順

1. 以下の情報を確認してください:
   - **ミッション**: 対象ミッション（省略時はアクティブミッションから推定）
   - **宛先** (to): registry.md の `agent` / all（互換: `hostname` も受理）
   - **優先度** (priority): low / normal / high / urgent
   - **タグ** (tags): 任意
   - **本文**: メッセージ内容

2. `GOAL.md` からアクティブミッションを確認
3. 該当ミッションの `messages/` ディレクトリ (`missions/<name>/messages/`) にメッセージを作成
4. 現在時刻でファイル名を生成
5. git commit & push まで自動実行

## 入力例

> JAM宛に、RDP接続テスト完了の報告を投稿して。priorityはnormalで。

## 注意

- from は自動でこのPCの `agent`（registry.md の `agent`）を設定します
- slug は内容から英数字・ハイフンで自動生成します
- ミッションが1つだけの場合は自動選択します
