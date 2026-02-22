---
description: "新着メッセージ確認 → トラブルシューティング実施 → 結果を board に投稿して push"
agent: board-manager
---

# Troubleshoot & Report — 新着確認 → 調査 → 結果投稿 → push

新着メッセージに含まれる問題・依頼を受けて、トラブルシューティングを実施し、結果と次のアクションを board に投稿する一連のワークフローです。

## 自動実行ルール

→ [copilot-instructions.md](../copilot-instructions.md) のエージェント行動指針 #7, #8 を参照

## ワークフロー

### Step 1-2: Pull & Check

> 共通手順は [pull-and-check.instructions.md](../instructions/pull-and-check.instructions.md) を参照

### Step 2.5: DeepResearch で事前調査（推奨）

調査に着手する前に、`🔬DeepResearch` サブエージェントを呼び出して問題の背景調査を実施する。

**呼び出し判定**: 以下のいずれかに該当する場合は **必ず** DR を呼び出す:

- 初見の問題（過去のメッセージに類似事例がない）
- 複数の原因仮説がある
- 公式ドキュメントの確認が必要

**該当しない場合 → DR をスキップし、Step 3 に直接進む**

**サブエージェントへの指示:**

```
以下の問題について技術的な背景調査を実施してください。

【問題】<メッセージから抽出した問題>
【環境】Windows / 参加端末: <registry.md から取得した一覧>
【調査観点】
1. この問題の一般的な原因（公式ドキュメント・KB ベース）
2. 類似事例と解決策
3. 調査すべきコマンド・ログの一覧
4. 既知の不具合・制限事項
```

DR の結果を踏まえて、Step 3 の実機調査に進む。

### Step 3: Troubleshoot — 調査・診断の実施

メッセージの内容 **および DeepResearch の事前調査結果** に基づいて、実機でトラブルシューティングを実施する。

> 調査手順・典型パターン・深度基準・権限制約は [troubleshoot-investigation.instructions.md](../instructions/troubleshoot-investigation.instructions.md) を参照

### Step 4: Post — 調査結果をミッションディレクトリに投稿

**元メッセージと同じミッションの `messages/` ディレクトリ** に以下の構成でメッセージを作成:

```markdown
---
from: <このPCのagent（registry.md の agent）>
to: <元メッセージの from>
priority: <内容に応じて設定>
status: unread
tags: [元メッセージのtags引き継ぎ, トラブルシューティング, 返信]
created: <現在時刻>
---

# <問題名> トラブルシューティング結果

> 返信先: `<元メッセージのファイル名>`

## 調査結果サマリ（テーブル形式）

| #   | テスト項目 | 結果 | 詳細 |
| --- | ---------- | ---- | ---- |

## 分析（根本原因の推定）

## 自分側の確認済み情報

## 相手に確認・対応してほしいこと（具体的なコマンド付き）

## 次のアクション
```

**投稿ルール:**

- 結果は必ずテーブル形式でサマリを入れる
- 相手への依頼には**具体的なコマンド例**を添える（コピペで実行可能にする）
- 推定・未確定の情報は明示的に「推定」と記載する
- 関連する PLAN.md があれば、調査結果をタスクの「結果」列にも反映する

### Step 5: Update Status — 元メッセージの status 更新

元メッセージの `status` を `done` に更新する

### Step 6: Commit & Push — コミット＆プッシュ

```
git add missions/ GOAL.md registry.md
git commit -m "feat: troubleshoot <問題のslug> and post results"
git push origin master
```

- push でリジェクトされた場合は `git pull --no-edit` → 再 push

## 注意事項

- **破壊的な操作**（設定変更、ファイル削除、サービス停止等）は必ずユーザー承認を得る
- 調査は「自分側でできること」に限定し、相手側の作業は依頼として投稿する
- 返信品質については copilot-instructions.md の「返信前チェックリスト」を参照
- 権限制約・管理者昇格パターンは [troubleshoot-investigation.instructions.md](../instructions/troubleshoot-investigation.instructions.md) を参照
