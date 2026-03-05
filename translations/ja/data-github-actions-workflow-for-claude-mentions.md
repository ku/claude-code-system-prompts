<!--
name: 'データ: @claudeメンション用GitHub Actionsワークフロー'
description: @claudeメンションでClaude Codeをトリガーするための GitHub Actionsワークフローテンプレート
ccVersion: 2.0.58
-->
name: Claude Code

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]
  pull_request_review:
    types: [submitted]

jobs:
  claude:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude')) ||
      (github.event_name == 'issues' && (contains(github.event.issue.body, '@claude') || contains(github.event.issue.title, '@claude')))
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
      actions: read # ClaudeがPRのCI結果を読み取るために必要
    steps:
      - name: リポジトリのチェックアウト
        uses: actions/checkout@v4
        with:
          fetch-depth: 1

      - name: Claude Codeの実行
        id: claude
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

          # これはClaudeがPRのCI結果を読み取れるようにするオプション設定です
          additional_permissions: |
            actions: read

          # オプション: Claudeにカスタムプロンプトを与えます。これを指定しない場合、Claudeはタグ付けされたコメントで指定された指示を実行します。
          # prompt: 'プルリクエストの説明を更新し、変更の要約を含めてください。'

          # オプション: claude_argsを追加して動作と設定をカスタマイズします
          # https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md または
          # https://code.claude.com/docs/en/cli-reference で利用可能なオプションを確認してください
          # claude_args: '--allowed-tools Bash(gh pr:*)'
