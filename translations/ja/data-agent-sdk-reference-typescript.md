<!--
name: 'データ: Agent SDKリファレンス — TypeScript'
description: インストール、クイックスタート、カスタムツール、フックを含むTypeScript Agent SDKリファレンス
ccVersion: 2.1.63
-->
# Agent SDK — TypeScript

Claude Agent SDKは、組み込みツール、安全機能、エージェント機能を備えたAIエージェントを構築するための高レベルインターフェースを提供します。

## インストール

\`\`\`bash
npm install @anthropic-ai/claude-agent-sdk
\`\`\`

---

## クイックスタート

\`\`\`typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Explain this codebase",
  options: { allowedTools: ["Read", "Glob", "Grep"] },
})) {
  if ("result" in message) {
    console.log(message.result);
  }
}
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

## 権限システム

\`\`\`typescript
for await (const message of query({
  prompt: "Refactor the authentication module",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits",
  },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

権限モード：

- `"default"`: 危険な操作の前にプロンプト
- `"plan"`: 計画のみ、実行なし
- `"acceptEdits"`: ファイル編集を自動承認
- `"dontAsk"`: プロンプトしない（CI/CDに便利）
- `"bypassPermissions"`: すべてのプロンプトをスキップ（オプションで`allowDangerouslySkipPermissions: true`が必要）

---

## MCP（Model Context Protocol）サポート

\`\`\`typescript
for await (const message of query({
  prompt: "Open example.com and describe what you see",
  options: {
    mcpServers: {
      playwright: { command: "npx", args: ["@playwright/mcp@latest"] },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

### インプロセスMCPツール

`tool()`と`createSdkMcpServer`を使用して、インプロセスで実行するカスタムツールを定義できます：

\`\`\`typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const myTool = tool("my-tool", "Description", { input: z.string() }, async (args) => {
  return { content: [{ type: "text", text: "result" }] };
});

const server = createSdkMcpServer({ name: "my-server", tools: [myTool] });

// queryに渡す
for await (const message of query({
  prompt: "Use my-tool to do something",
  options: { mcpServers: { myServer: server } },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

---

## フック

\`\`\`typescript
import { query, HookCallback } from "@anthropic-ai/claude-agent-sdk";
import { appendFileSync } from "fs";

const logFileChange: HookCallback = async (input) => {
  const filePath = (input as any).tool_input?.file_path ?? "unknown";
  appendFileSync(
    "./audit.log",
    \`\${new Date().toISOString()}: modified \${filePath}\\n\`,
  );
  return {};
};

for await (const message of query({
  prompt: "Refactor utils.py to improve readability",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits",
    hooks: {
      PostToolUse: [{ matcher: "Edit|Write", hooks: [logFileChange] }],
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

利用可能なフックイベント: `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `Notification`, `UserPromptSubmit`, `SessionStart`, `SessionEnd`, `Stop`, `SubagentStart`, `SubagentStop`, `PreCompact`, `PermissionRequest`, `Setup`, `TeammateIdle`, `TaskCompleted`, `ConfigChange`

---

## 一般的なオプション

`query()`はトップレベルの`prompt`（文字列）と`options`オブジェクトを受け取ります：

\`\`\`typescript
query({ prompt: "...", options: { ... } })
\`\`\`

| オプション                              | 型   | 説明                                                                |
| ----------------------------------- | ------ | -------------------------------------------------------------------------- |
| `cwd`                               | string | ファイル操作の作業ディレクトリ                                      |
| `allowedTools`                      | array  | エージェントが使用できるツール（例：`["Read", "Edit", "Bash"]`）                |
| `tools`                             | array  | 利用可能にする組み込みツール（デフォルトセットを制限）               |
| `disallowedTools`                   | array  | 明示的に禁止するツール                                               |
| `permissionMode`                    | string | 権限プロンプトの処理方法                                           |
| `allowDangerouslySkipPermissions`   | bool   | `permissionMode: "bypassPermissions"`を使用するには`true`が必要                |
| `mcpServers`                        | object | 接続するMCPサーバー                                                  |
| `hooks`                             | object | 動作をカスタマイズするフック                                             |
| `systemPrompt`                      | string | カスタムシステムプロンプト                                                       |
| `maxTurns`                          | number | 停止前の最大エージェントターン数                                        |
| `maxBudgetUsd`                      | number | クエリの最大予算（USD）                                        |
| `model`                             | string | モデルID（デフォルト：CLIによって決定）                                      |
| `agents`                            | object | サブエージェント定義（`Record<string, AgentDefinition>`）                   |
| `outputFormat`                      | object | 構造化出力スキーマ                                                   |
| `thinking`                          | object | 思考/推論制御                                                 |
| `betas`                             | array  | 有効にするベータ機能（例：`["context-1m-2025-08-07"]`）               |
| `settingSources`                    | array  | 読み込む設定（例：`["project"]`）。デフォルト：なし（CLAUDE.mdファイルなし） |
| `env`                               | object | セッションに設定する環境変数                               |

---

## サブエージェント

\`\`\`typescript
for await (const message of query({
  prompt: "Use the code-reviewer agent to review this codebase",
  options: {
    allowedTools: ["Read", "Glob", "Grep", "Agent"],
    agents: {
      "code-reviewer": {
        description: "Expert code reviewer for quality and security reviews.",
        prompt: "Analyze code quality and suggest improvements.",
        tools: ["Read", "Glob", "Grep"],
      },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

---

## メッセージタイプ

\`\`\`typescript
for await (const message of query({
  prompt: "Find TODO comments",
  options: { allowedTools: ["Read", "Glob", "Grep"] },
})) {
  if ("result" in message) {
    console.log(message.result);
  } else if (message.type === "system" && message.subtype === "init") {
    const sessionId = message.session_id; // 後で再開するためにキャプチャ
  }
}
\`\`\`

---

## ベストプラクティス

1. **常にallowedToolsを指定** — エージェントが使用できるツールを明示的にリスト
2. **作業ディレクトリを設定** — ファイル操作には常に`cwd`を指定
3. **適切な権限モードを使用** — `"default"`から始めて、必要な場合にのみエスカレート
4. **すべてのメッセージタイプを処理** — エージェント出力を取得するには`result`プロパティをチェック
5. **maxTurnsを制限** — 妥当な制限で暴走エージェントを防止
