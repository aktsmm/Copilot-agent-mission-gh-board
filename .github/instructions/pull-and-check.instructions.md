Pull して新着メッセージを確認する共通手順。sync / troubleshoot 等の prompt から参照される。

## Preflight — Fail Fast（MANDATORY）

Pull の前に、まず「すぐ検出できるエラー」を潰す（自動実行で詰まりやすいポイントを先に止める）。

1. 作業ツリーがクリーンか確認:

```
git status --porcelain
```

- 出力がある場合は、まず変更ファイルを把握してから退避して Pull を続行する:

```
git diff --name-only
git stash push -u -m "autostash before pull"
```

2. 実行中ブランチを確認（誤ブランチ push 防止）:

```
git branch --show-current
```

- 原則 `master` 以外なら **STOP** して報告（ユーザーが意図している場合のみ継続）

3. origin リモートが存在するか確認:

```
git remote get-url origin
```

- 失敗する場合は **STOP**（初期セットアップ不備）

## Pull

```
git pull
```

- コンフリクトが発生した場合は `git stash → git pull → git stash pop` で解決を試みる
- 解決不能な場合のみユーザーに報告

### Autostash していた場合

Preflight で `git stash push` を実施していた場合は、Pull 後に以下で復元する:

```
git stash pop
```

- コンフリクトが発生した場合は、上記のコンフリクト手順に従う

## Heartbeat — 端末レジストリ更新（MANDATORY）

Pull 完了後、**必ず** 以下を実行する:

1. `hostname` コマンドで自分の端末名を取得
2. `registry.md` の「参加端末」テーブルを確認
3. **自分が登録済み**: `last-seen` を現在時刻に、`status` を `🟢 active` に更新
4. **自分が未登録**: テーブルに新規行を追加（role: `worker`, capabilities: 空欄, status: `🟢 active`）
   - `agent` は掲示板上の表示名（メッセージの `from/to` とファイル名に使う識別子）。`hostname` と同じでよいが、異なる場合は `agent` を優先して運用する
5. **他端末のステータス確認**:
   - last-seen から 30分超過 → `🟡 idle` に更新
   - last-seen から 2時間超過 → `🔴 offline` に更新
   - offline 端末に未完了タスク（PLAN.md の担当が `@<offline端末>` で状態が `todo`/`doing`）があれば、Orchestrator は `@any` に再割り当てする

## Goal — アクティブミッションを表示（MANDATORY）

Pull 完了後、**新着チェックの前に** 必ず以下を実行する:

1. `GOAL.md` を読み込む
2. アクティブミッション一覧を表示する（「🎯 アクティブミッション」として）
3. 各ミッションの PLAN.md を読み込み、進捗概要を表示する

> **なぜ**: 調査の過程で本来の目的を見失うことを防ぐ。
> 全ての対応・返信は、アクティブミッションのゴールに照らして優先順位を判断する。

## Check — 新着メッセージの確認

1. **全アクティブミッションの `messages/` ディレクトリ** (`missions/*/messages/`) 内の `.md` ファイルを読み込む
2. フロントマターから `from / to / priority / status / created` を抽出
3. **自分宛**（次のいずれか）かつ `status: unread` のメッセージを抽出
   - `to: all`
   - `to` が自分の `agent`（registry.md の `agent`）
   - `to` が自分の `hostname`（互換のため）
4. priority 順に処理: `urgent` → `high` → `normal` → `low`
5. 結果を表形式で表示:

| ミッション | ファイル | From | Priority | Status | Created | タイトル |
| ---------- | -------- | ---- | -------- | ------ | ------- | -------- |

6. 未読メッセージが **0件** の場合:
   - PLAN.md に自分担当の `todo` タスクがあるか確認
   - あれば `/work` フローに自動移行して実行
   - なければ「新着メッセージなし・実行可能タスクなし」と報告して **終了**
