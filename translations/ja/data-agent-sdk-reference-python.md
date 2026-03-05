<!--
name: 'データ: Agent SDKリファレンス — Python'
description: インストール、クイックスタート、MCP経由のカスタムツール、フックを含むPython Agent SDKリファレンス
ccVersion: 2.1.63
-->
# Agent SDK — Python

Claude Agent SDKは、組み込みツール、安全機能、エージェント機能を備えたAIエージェントを構築するための高レベルインターフェースを提供します。

## インストール

\`\`\`bash
pip install claude-agent-sdk
\`\`\`

---

## クイックスタート

\`\`\`python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Explain this codebase",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`

---

## 組み込みツール

| ツール      | 説明                          |
| --------- | ------------------------------------ |
| Read      | ワークスペース内のファイルを読み取る          |
| Write     | 新しいファイルを作成                     |
| Edit      | 既存ファイルに精密な編集を加える |
| Bash      | シェルコマンドを実行               |
| Glob      | パターンでファイルを検索                |
| Grep      | コンテンツでファイルを検索              |
| WebSearch | Web上の情報を検索       |
| WebFetch        | Webページを取得して分析          |
| AskUserQuestion | ユーザーに明確化のための質問を行う         |
| Agent           | サブエージェントをスポーン                      |

---

## 主要インターフェース

### `query()` — シンプルなワンショット使用

`query()`関数は、エージェントを実行する最もシンプルな方法です。メッセージの非同期イテレータを返します。

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Explain this codebase",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

### `ClaudeSDKClient` — 完全な制御

`ClaudeSDKClient`は、エージェントのライフサイクルを完全に制御します。カスタムツール、フック、ストリーミング、または実行を中断する機能が必要な場合に使用します。

\`\`\`python
import anyio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, AssistantMessage, TextBlock

async def main():
    options = ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
    async with ClaudeSDKClient(options=options) as client:
        await client.query("Explain this codebase")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)

anyio.run(main)
\`\`\`

`ClaudeSDKClient`は以下をサポート：

- **コンテキストマネージャー** (`async with`) による自動リソースクリーンアップ
- **`client.query(prompt)`** でエージェントにプロンプトを送信
- **`receive_response()`** で完了までメッセージをストリーミング
- **`interrupt()`** でタスク中のエージェント実行を停止
- **カスタムツールに必須** (SDK MCPサーバー経由)

---

## 権限システム

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Refactor the authentication module",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Write"],
        permission_mode="acceptEdits"  # ファイル編集を自動承認
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

権限モード：

- `"default"`: 危険な操作の前にプロンプト
- `"plan"`: 計画のみ、実行なし
- `"acceptEdits"`: ファイル編集を自動承認
- `"dontAsk"`: プロンプトしない（CI/CDに便利）
- `"bypassPermissions"`: すべてのプロンプトをスキップ（オプションで`allow_dangerously_skip_permissions=True`が必要）

---

## MCP（Model Context Protocol）サポート

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Open example.com and describe what you see",
    options=ClaudeAgentOptions(
        mcp_servers={
            "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

---

## フック

コールバック関数を使用してエージェントの動作をカスタマイズ：

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage

async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get('tool_input', {}).get('file_path', 'unknown')
    print(f"Modified: {file_path}")
    return {}

async for message in query(
    prompt="Refactor utils.py",
    options=ClaudeAgentOptions(
        permission_mode="acceptEdits",
        hooks={
            "PostToolUse": [HookMatcher(matcher="Edit|Write", hooks=[log_file_change])]
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

利用可能なフックイベント: `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `Notification`, `UserPromptSubmit`, `SessionStart`, `SessionEnd`, `Stop`, `SubagentStart`, `SubagentStop`, `PreCompact`, `PermissionRequest`, `Setup`, `TeammateIdle`, `TaskCompleted`, `ConfigChange`

---

## 一般的なオプション

`query()`はトップレベルの`prompt`（文字列）と`options`オブジェクト（`ClaudeAgentOptions`）を受け取ります：

\`\`\`python
async for message in query(prompt="...", options=ClaudeAgentOptions(...)):
\`\`\`

| オプション                              | 型   | 説明                                                                |
| ----------------------------------- | ------ | -------------------------------------------------------------------------- |
| `cwd`                               | string | ファイル操作の作業ディレクトリ                                      |
| `allowed_tools`                     | list   | エージェントが使用できるツール（例：`["Read", "Edit", "Bash"]`）                |
| `tools`                             | list   | 利用可能にする組み込みツール（デフォルトセットを制限）               |
| `disallowed_tools`                  | list   | 明示的に禁止するツール                                               |
| `permission_mode`                   | string | 権限プロンプトの処理方法                                           |
| `allow_dangerously_skip_permissions`| bool   | `permission_mode="bypassPermissions"`を使用するには`True`が必要                |
| `mcp_servers`                       | dict   | 接続するMCPサーバー                                                  |
| `hooks`                             | dict   | 動作をカスタマイズするフック                                             |
| `system_prompt`                     | string | カスタムシステムプロンプト                                                       |
| `max_turns`                         | int    | 停止前の最大エージェントターン数                                        |
| `max_budget_usd`                    | float  | クエリの最大予算（USD）                                        |
| `model`                             | string | モデルID（デフォルト：CLIによって決定）                                      |
| `agents`                            | dict   | サブエージェント定義（`dict[str, AgentDefinition]`）                        |
| `output_format`                     | dict   | 構造化出力スキーマ                                                   |
| `thinking`                          | dict   | 思考/推論制御                                                 |
| `betas`                             | list   | 有効にするベータ機能（例：`["context-1m-2025-08-07"]`）               |
| `setting_sources`                   | list   | 読み込む設定（例：`["project"]`）。デフォルト：なし（CLAUDE.mdファイルなし） |
| `env`                               | dict   | セッションに設定する環境変数                               |

---

## メッセージタイプ

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage

async for message in query(
    prompt="Find TODO comments",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    if isinstance(message, ResultMessage):
        print(message.result)
    elif isinstance(message, SystemMessage) and message.subtype == "init":
        session_id = message.session_id  # 後で再開するためにキャプチャ
\`\`\`

---

## サブエージェント

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

async for message in query(
    prompt="Use the code-reviewer agent to review this codebase",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Glob", "Grep", "Agent"],
        agents={
            "code-reviewer": AgentDefinition(
                description="Expert code reviewer for quality and security reviews.",
                prompt="Analyze code quality and suggest improvements.",
                tools=["Read", "Glob", "Grep"]
            )
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

---

## エラー処理

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, CLINotFoundError, CLIConnectionError, ResultMessage

try:
    async for message in query(
        prompt="...",
        options=ClaudeAgentOptions(allowed_tools=["Read"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)
except CLINotFoundError:
    print("Claude Code CLIが見つかりません。インストール: pip install claude-agent-sdk")
except CLIConnectionError as e:
    print(f"接続エラー: {e}")
\`\`\`

---

## ベストプラクティス

1. **常にallowed_toolsを指定** — エージェントが使用できるツールを明示的にリスト
2. **作業ディレクトリを設定** — ファイル操作には常に`cwd`を指定
3. **適切な権限モードを使用** — `"default"`から始めて、必要な場合にのみエスカレート
4. **すべてのメッセージタイプを処理** — エージェント出力を取得するには`ResultMessage`をチェック
5. **max_turnsを制限** — 妥当な制限で暴走エージェントを防止
