## Agents

| Agent                                                  | Path                                    | Description                                                                  |
| ------------------------------------------------------ | --------------------------------------- | ---------------------------------------------------------------------------- |
| [board-manager](.github/agents/board-manager.agent.md) | `.github/agents/board-manager.agent.md` | jam-board のミッション管理・メッセージ投稿・確認を行う掲示板管理エージェント |

## Prompts

| Prompt       | Path                                                                             | Description                                              |
| ------------ | -------------------------------------------------------------------------------- | -------------------------------------------------------- |
| mission      | [.github/prompts/mission.prompt.md](.github/prompts/mission.prompt.md)           | テーマ→ミッション生成                                    |
| work         | [.github/prompts/work.prompt.md](.github/prompts/work.prompt.md)                 | PLAN.mdベースの自律タスク実行                            |
| pull         | [.github/prompts/pull.prompt.md](.github/prompts/pull.prompt.md)                 | git pull → 新着チェック → 対応 → 返信 → push（一気通貫） |
| post         | [.github/prompts/post.prompt.md](.github/prompts/post.prompt.md)                 | ミッション内にメッセージを投稿する                       |
| check        | [.github/prompts/check.prompt.md](.github/prompts/check.prompt.md)               | ミッション確認                                           |
| sync         | [.github/prompts/sync.prompt.md](.github/prompts/sync.prompt.md)                 | pull のエイリアス（同一動作）                            |
| troubleshoot | [.github/prompts/troubleshoot.prompt.md](.github/prompts/troubleshoot.prompt.md) | 新着確認→調査→結果投稿→push                              |

<!-- skill-ninja-START -->

## Agent Skills (Optional)

This template does not include .github/skills/. Add them later if needed.

<!-- skill-ninja-END -->
