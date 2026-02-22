# jam-board Copilot Instructions

このワークスペースは **2台の PC 間の双方向掲示板 (jam-board)** です。
Git リポジトリで同期し、Markdown ファイルでメッセージ・ログをやり取りします。

## 端末情報

参加端末の一覧は `registry.md` で管理する（SSOT）。

**自分の hostname の特定方法**: エージェント起動時に `hostname` コマンドで取得し、`registry.md` と照合する。

## ワークスペース構造

```
jam-board/
├── GOAL.md                      # アクティブミッション一覧（ポインタ）
├── registry.md                  # 参加端末レジストリ（動的管理）
├── missions/                    # ミッション（テーマ）ごとのディレクトリ
│   ├── _template/               # 新規ミッション用テンプレート
│   │   ├── GOAL.md
│   │   ├── PLAN.md
│   │   └── SUMMARY.md
│   └── <mission-name>/          # 各ミッション
│       ├── GOAL.md              # ゴール定義
│       ├── PLAN.md              # タスク分解・進捗管理
│       ├── SUMMARY.md           # 完了サマリー（完了時に作成）
│       ├── messages/            # ボードやり取り
│       ├── scripts/             # 関連スクリプト
│       └── research/            # DR結果・調査メモ
├── README.md                    # リポジトリ説明・緊急復旧手順
├── AGENTS.md                    # エージェント一覧
├── .github/
│   ├── copilot-instructions.md  # この文書
│   ├── agents/
│   │   └── board-manager.agent.md
│   ├── instructions/
│   │   ├── pull-and-check.instructions.md  # Pull+新着確認の共通手順
│   │   └── troubleshoot-investigation.instructions.md  # 調査ガイドライン
│   └── prompts/
│       ├── mission.prompt.md    # テーマ→ミッション生成
│       ├── work.prompt.md       # PLAN.mdベースの自律タスク実行
│       ├── pull.prompt.md       # プル→新着チェック→対応→返信→push
│       ├── post.prompt.md       # メッセージ投稿
│       ├── check.prompt.md      # ミッション確認
│       ├── sync.prompt.md       # pull のエイリアス
│       └── troubleshoot.prompt.md # 新着確認→調査→結果投稿→push
└── clean_env/                   # Python 仮想環境（操作不要）
```

## メッセージ規約

- **配置場所**: メッセージは必ず該当ミッションの `messages/` ディレクトリ (`missions/<name>/messages/`) 内に作成する
- **ファイル名**: `YYYY-MM-DD_HH-MM_agent_slug.md`（agent = registry.md の `agent`。例: `vainf`, `JAM`）
  - 例: `missions/rdp-fix/messages/2026-02-22_07-00_vainf_rdp-reboot-plan.md`
- **slug**: 英数字・ハイフンのみ、内容がわかる短い名前
- **返信ルール**: 既存ファイルを編集せず、**新しいファイルを作成**して返信する。関連するメッセージは slug やタグで紐づける
- **フォーマット**: 下記の YAML フロントマター + 本文

```markdown
---
from: <自分の agent>
to: <相手の agent / all>
priority: low | normal | high | urgent
status: unread | read | done
tags: [タグ1, タグ2]
created: YYYY-MM-DDTHH:MM
---

# タイトル

本文をここに書く
```

## ステータス遷移

`unread` → `read`（相手が確認） → `done`（対応完了）

## エージェント行動指針

1. **missions/ が SSOT**: メッセージの読み書きは必ず `missions/<name>/messages/` ディレクトリで行う。スクリプトは `scripts/`、調査結果は `research/` に配置する
2. **ファイル名規約を厳守**: タイムスタンプ + `agent` + slug 形式
3. **from/to を正確に**: `hostname` で自分の端末を特定し、`registry.md` の `agent` をメッセージの `from/to` とファイル名に使用する

  - 補足: `hostname`（例: X600）と掲示板上の `agent`（例: vainf）が異なる場合がある。互換のため `to: <hostname>` も受理してよいが、基本は `agent` で運用する
4. **git push は自動実行**: コミット後は自動で push する（リジェクト時は `git pull --no-edit` → 再 push）
5. **破壊的操作の前に確認**: ファイル削除・アーカイブの前にユーザーに確認する
6. **日本語で回答**: 会話はカジュアル、成果物は構造化
7. **常に一気通貫で進行**: pull → 新着確認 → 対応 → status 更新 → 返信投稿 → コミット → push の流れは、プロンプト種別を問わず常に一気通貫で実行する。途中でユーザーに「対応する？」「チェックする？」等の確認を挟まない（破壊的操作を除く）
8. **pull 後は自動的に新着チェック**: `git pull` を実行したら、必ず pull-and-check.instructions.md に従い新着確認まで自動進行する。未読メッセージがあれば対応→返信→コミット→push まで進む。未読がなく PLAN.md に自分担当の todo タスクがあれば `/work` フローに移行する
9. **最小往復・最大自己解決**: → 下記セクション参照（最重要原則）
10. **目的を見失わない**: `GOAL.md` にアクティブミッション一覧、各ミッションの `GOAL.md` に最終ゴールが記載されている。対応・調査・返信の全てにおいて、このゴールに照らして優先順位を判断する。脇道に逸れそうなときは「これはゴール達成に直接必要か？」を自問する

## 最小往復・最大自己解決の原則（MANDATORY — 最重要）

**1回の返信で問題を解決する** ことを最優先目標とする。
メッセージの往復は「コスト」であり、各往復に **数時間かかる** と想定して行動すること。

### 受信時の行動規範

メッセージを受け取ったら、**返信する前に以下を全て実施する**:

1. **依頼されたことをやる** — 当然。ここで止まらない
2. **依頼されていないがやるべきことをやる** — 依頼内容の周辺で、明らかに必要な調査・確認・修正を全て実施する
3. **修正可能なものは即座に修正する** — 問題を見つけたら報告だけでなく、自分側で直せるなら直す
4. **次に聞かれそうなことを先回りで調べる** — 相手が結果を見て次に確認したくなることを予測し、事前に回答を用意する
5. **代替手段も先に調査・検証する** — 提案した方法がダメだった場合に備え、代替案を自分側でできる範囲で実際に検証しておく
6. **自分側でできることを全て完了してから返信する** — 「〜してみてください」と投げる前に、自分側で確認可能・実行可能なことが残っていないか確認する

### 返信前チェックリスト（MUST）

返信メッセージを投稿する **前に** 以下を全て自問する。1つでも No なら追加作業を行う:

- [ ] 依頼された作業は全て実施したか？
- [ ] 依頼されていないが関連する調査・修正は全て行ったか？
- [ ] 自分側で試せる修正・回避策を全て試したか？
- [ ] 見つけた問題は「報告」だけでなく「修正」まで行ったか？（可能な範囲で）
- [ ] 提案する方法がダメだった場合の代替案も調査・検証したか？
- [ ] 相手に依頼する内容には、コピペ可能な具体的コマンドを添えたか？
- [ ] 相手が次に確認しそうな情報を先回りで含めたか？
- [ ] **この返信を受け取った相手が、追加の質問なしに作業を完了できるか？**

### 悪い例と良い例

**❌ 悪い例（往復が増える）:**

1. vainf: 「SMBが繋がらない、調べて」
2. JAM: 「RejectUnencryptedAccess を直した、試して」
3. vainf: 「まだダメ」
4. JAM: 「LanmanServer 再起動した、試して」
5. vainf: 「まだダメ」
6. JAM: 「こっちは正常、そっちの FW では？」

→ **6往復**。毎回1つしか確認せず、部分的な対応を繰り返している

**✅ 良い例（1-2往復で解決）:**

1. vainf: 「SMBが繋がらない、調べて」
2. JAM: 以下を **全て実施した上で** 1回で返信:
   - RejectUnencryptedAccess → False に修正済み
   - LanmanServer 再起動済み・動作確認済み
   - FW ルール全数確認済み（受信OK、20件以上 Allow）
   - ローカル SMB テスト正常確認済み
   - JAM→vainf 方向の TCP 445 疎通も確認済み
   - ∴ JAM側は100%正常。**vainf側の確認コマンド**（コピペで実行可能）:
     ```powershell
     Get-NetConnectionProfile
     Get-NetFirewallRule | Where-Object { $_.Direction -eq 'Outbound' -and ... }
     Get-MpPreference | Select-Object EnableNetworkProtection
     ```
   - Network Protection が有効だった場合の代替手段も **検証済み**:
     - (A) FWルール追加: `New-NetFirewallRule ...`（コマンド添付）
     - (B) HTTP経由: JAM側で `python -m http.server 8080` **起動テスト済み**
     - (C) SSH/SCP: JAM側の sshd 状態確認済み（未インストール→インストール手順添付）

→ **2往復で全パターン網羅**。相手はどのパターンでも即座に対応できる

### クリティカルな問題の扱い

RDP接続不可、ファイル共有不可など **業務に直接影響する問題** は最高優先度で対応する:

- 調査は **表面的な確認で絶対に終わらせない** — 根本原因が特定できるまで全レイヤーを深掘りする
- 複数の仮説がある場合は **全て検証する**（1つ試してダメなら次、ではなく並行で全部）
- 修正可能なものは **報告前に修正する**（「これが原因かも」ではなく「これが原因だったので直した」）
- 代替手段を **必ず用意する**（メインの方法がダメだった場合も相手が困らないように）
- 相手に依頼する作業は **全てコピペ可能なコマンドと期待される結果** を添える

## コミット規約

- Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`
- 例: `feat: post network-fix instructions to JAM`

## `/export-session-log` ルール

`/export-session-log` を実行する際は、出力の冒頭に以下の情報を必ず明記すること：

- **ホスト名**: `hostname` コマンド等で取得したマシン名
- **IPアドレス**: 主要な NIC の IPv4 アドレス（複数ある場合はすべて記載）

これにより、どの環境で実施したセッションかを後から特定できるようにする。

## Skills (Optional)

This template does not include .github/skills/. Add them later if needed.
