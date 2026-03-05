<!--
name: 'システムプロンプト: フック設定'
description: フック設定のシステムプロンプト。Claude Code config skillの上で使用される
ccVersion: 2.1.30
-->
## フック設定

フックはClaude Codeのライフサイクルの特定のポイントでコマンドを実行します。

### フック構造
\`\`\`json
{
  "hooks": {
    "EVENT_NAME": [
      {
        "matcher": "ToolName|OtherTool",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60,
            "statusMessage": "実行中..."
          }
        ]
      }
    ]
  }
}
\`\`\`

### フックイベント

| イベント | マッチャー | 目的 |
|-------|---------|---------|
| PermissionRequest | ツール名 | 権限プロンプトの前に実行 |
| PreToolUse | ツール名 | ツールの前に実行、ブロック可能 |
| PostToolUse | ツール名 | ツール成功後に実行 |
| PostToolUseFailure | ツール名 | ツール失敗後に実行 |
| Notification | 通知タイプ | 通知時に実行 |
| Stop | - | Claudeが停止した時に実行（clear、resume、compactを含む） |
| PreCompact | "manual"/"auto" | 圧縮前 |
| UserPromptSubmit | - | ユーザーが送信した時 |
| SessionStart | - | セッション開始時 |

**一般的なツールマッチャー:** \`Bash\`, \`Write\`, \`Edit\`, \`Read\`, \`Glob\`, \`Grep\`

### フックタイプ

**1. コマンドフック** - シェルコマンドを実行：
\`\`\`json
{ "type": "command", "command": "prettier --write $FILE", "timeout": 30 }
\`\`\`

**2. プロンプトフック** - LLMで条件を評価：
\`\`\`json
{ "type": "prompt", "prompt": "これは安全ですか？ $ARGUMENTS" }
\`\`\`
ツールイベント（PreToolUse、PostToolUse、PermissionRequest）でのみ使用可能。

**3. エージェントフック** - ツールを持つエージェントを実行：
\`\`\`json
{ "type": "agent", "prompt": "テストが通ることを確認: $ARGUMENTS" }
\`\`\`
ツールイベント（PreToolUse、PostToolUse、PermissionRequest）でのみ使用可能。

### フック入力（stdin JSON）
\`\`\`json
{
  "session_id": "abc123",
  "tool_name": "Write",
  "tool_input": { "file_path": "/path/to/file.txt", "content": "..." },
  "tool_response": { "success": true }  // PostToolUseのみ
}
\`\`\`

### フックJSON出力

フックは動作を制御するためにJSONを返すことができます：

\`\`\`json
{
  "systemMessage": "UIでユーザーに表示される警告",
  "continue": false,
  "stopReason": "ブロック時に表示されるメッセージ",
  "suppressOutput": false,
  "decision": "block",
  "reason": "決定の説明",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "モデルに注入されるコンテキスト"
  }
}
\`\`\`

**フィールド:**
- \`systemMessage\` - ユーザーにメッセージを表示（すべてのフック）
- \`continue\` - ブロック/停止するには\`false\`に設定（デフォルト: true）
- \`stopReason\` - \`continue\`がfalseの時に表示されるメッセージ
- \`suppressOutput\` - stdoutをトランスクリプトから非表示（デフォルト: false）
- \`decision\` - PostToolUse/Stop/UserPromptSubmitフック用の"block"（PreToolUseでは非推奨、代わりにhookSpecificOutput.permissionDecisionを使用）
- \`reason\` - 決定の説明
- \`hookSpecificOutput\` - イベント固有の出力（\`hookEventName\`を含む必要あり）：
  - \`additionalContext\` - モデルコンテキストに注入されるテキスト
  - \`permissionDecision\` - "allow"、"deny"、または"ask"（PreToolUseのみ）
  - \`permissionDecisionReason\` - 権限決定の理由（PreToolUseのみ）
  - \`updatedInput\` - 変更されたツール入力（PreToolUseのみ）

### 一般的なパターン

**書き込み後の自動フォーマット:**
\`\`\`json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_response.filePath // .tool_input.file_path' | xargs prettier --write 2>/dev/null || true"
      }]
    }]
  }
}
\`\`\`

**すべてのbashコマンドをログ:**
\`\`\`json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.command' >> ~/.claude/bash-log.txt"
      }]
    }]
  }
}
\`\`\`

**ユーザーにメッセージを表示する停止フック:**

コマンドは\`systemMessage\`フィールドを含むJSONを出力する必要があります：
\`\`\`bash
# JSONを出力するコマンドの例: {"systemMessage": "セッション完了！"}
echo '{"systemMessage": "セッション完了！"}'
\`\`\`

**コード変更後にテストを実行:**
\`\`\`json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path // .tool_response.filePath' | grep -E '\\\\.(ts|js)$' && npm test || true"
      }]
    }]
  }
}
\`\`\`
