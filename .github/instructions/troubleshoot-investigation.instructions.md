<!-- Referenced from troubleshoot.prompt.md -->

# トラブルシューティング調査ガイドライン

## 調査の進め方

> copilot-instructions.md の「最小往復・最大自己解決の原則」に従うこと。

1. **問題の特定**: メッセージから依頼・問題点を抽出
2. **基本確認から順に深掘り**: レイヤーの低い方から確認（疎通 → ポート → サービス → アプリ）
3. **全仮説を並行検証**: 複数の原因候補がある場合、1つずつではなく **全て同時に** 調査する
4. **各ステップの結果を記録**: 何を実行して何がわかったかを逐一記録
5. **見つけた問題は即修正**: 報告だけで終わらず、自分側で直せるものは **修正してから報告する**
6. **根本原因の特定**: 推定ではなく確定を目指す。確定できない場合は残る仮説と検証方法を明示する
7. **代替手段の事前検証**: メインの解決策がダメだった場合に備え、代替手段を **自分側で実際にテストしておく**
8. **相手への依頼事項を整理**: 自分側で対応不可能な項目のみ、コピペ可能なコマンド付きでまとめる

## 調査の典型パターン

| カテゴリ         | 確認コマンド例                        |
| ---------------- | ------------------------------------- |
| 疎通             | `ping`, `Test-NetConnection`          |
| ポート           | `Test-NetConnection -Port <n>`        |
| DNS/名前解決     | `nslookup`, `nbtstat -A`              |
| SMB共有          | `net view`, `net use`, `Get-SmbShare` |
| サービス         | `Get-Service`, `sc.exe query`         |
| ファイアウォール | `Get-NetFirewallRule`                 |
| ログ             | `Get-EventLog`, `Get-WinEvent`        |

## 調査深度の基準

**表面的な確認では不十分**。問題が特定できない場合は、より深いレイヤーまで掘り下げる:

1. **第1層**: 高レベルAPI・コマンド (`Get-Service`, `Get-SmbShare`, `Test-NetConnection`)
2. **第2層**: レジストリ・設定ファイル (`Get-ItemProperty`, `Get-SmbServerConfiguration`)
3. **第3層**: ドライバー・カーネルサービス (`sc.exe query`, `.sys` ドライバー状態)
4. **第4層**: プロトコルレベル検証 (RAW TCP socket, パケット送受信)

**例: SMB接続問題**

- ❌ 表面的: `net view \\server` → エラー → 「SMBがダメです」で終了
- ✅ 徹底的:
  1. `Test-NetConnection -Port 445` → TCP疎通確認
  2. `Get-SmbServerConfiguration` → 設定確認
  3. `sc.exe query srv2`, `sc.exe query srvnet` → ドライバー状態
  4. RAW TCP socket + SMB negotiate packet送信 → プロトコル応答確認
  5. `netstat -an | Select-String ':445'` → リスニング状態
  6. `net view \\127.0.0.1` → ローカルループバック経由テスト

→ これにより「JAM側は100%正常、問題はvainf側」と確定できた

## 非管理者権限での調査制約

一部のログ・設定は管理者権限なしでは読み取れない。エラーが出たら代替手段を試す:

| アクセス失敗                                            | エラー                                         | 代替手段                                                            |
| ------------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------- |
| `Get-WinEvent -LogName 'Microsoft-Windows-SMBServer/*'` | Attempted to perform an unauthorized operation | `Get-WinEvent -LogName System` で srv/SMB 関連を検索                |
| SMBServer イベントログ                                  | 同上                                           | `wevtutil el \| Select-String 'SMB'` でログ名一覧 →読めるログを確認 |
| FW詳細設定                                              | Access denied                                  | `netsh advfirewall show allprofiles` で最小限確認                   |

**同じコマンドを何度も繰り返さない** — 1回目でエラーなら代替手段に切り替える

## 管理者権限が必要なコマンド

権限不足（exit code 1）の場合は `Start-Process -Verb RunAs` で昇格実行し、結果を一時ファイルに出力する:

```powershell
# 1. 出力先ディレクトリを確保
if (-not (Test-Path C:\temp)) { New-Item -ItemType Directory -Path C:\temp }
# 2. 昇格実行 + 結果をファイルに書き出し
Start-Process powershell -Verb RunAs -ArgumentList '
  -NoProfile -Command "<コマンド> | Out-File C:\temp\result.txt"' -Wait
# 3. 結果を読み取り
Get-Content C:\temp\result.txt
```

- タイムアウトするコマンドは適切にタイムアウト処理する（SMB アクセス等）
  - SMB/UNC パスへのアクセスは `net use` + `2>&1` でエラーコードを取得する（`Test-Path \\...` はハングするので避ける）
  - タイムアウト付きコマンドは必ず timeout パラメータを設定し、ターミナルが詰まらないようにする
- 調査結果が長くなっても省略しない — 相手が再現・判断できる情報量を維持する
