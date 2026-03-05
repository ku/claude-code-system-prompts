<!--
name: 'データ: GitHub Appインストール用PR説明'
description: Claude Code GitHub App統合をインストールする際のPR説明テンプレート
ccVersion: 2.0.14
-->
## Claude Code GitHub Appのインストール

このPRは、リポジトリでClaude Code統合を有効にするGitHub Actionsワークフローを追加します。

### Claude Codeとは？

[Claude Code](https://claude.com/claude-code)は、以下を支援できるAIコーディングエージェントです：
- バグ修正と改善
- ドキュメントの更新
- 新機能の実装
- コードレビューと提案
- テストの作成
- その他多数！

### 仕組み

このPRがマージされると、プルリクエストまたはIssueコメントで@claudeをメンションすることでClaudeと対話できるようになります。
ワークフローがトリガーされると、Claudeはコメントと周囲のコンテキストを分析し、GitHub Actionでリクエストを実行します。

### 重要な注意事項

- **このワークフローはこのPRがマージされるまで有効になりません**
- **@claudeメンションはマージが完了するまで機能しません**
- ワークフローはPRまたはIssueコメントでClaudeがメンションされるたびに自動的に実行されます
- Claudeはファイル、差分、以前のコメントを含むPRまたはIssue全体のコンテキストにアクセスできます

### セキュリティ

- Anthropic APIキーはGitHub Actionsシークレットとして安全に保存されています
- リポジトリへの書き込みアクセス権を持つユーザーのみがワークフローをトリガーできます
- すべてのClaude実行はGitHub Actions実行履歴に保存されます
- Claudeのデフォルトツールは、ファイルの読み書き、コメント・ブランチ・コミットの作成によるリポジトリとの対話に限定されています
- 以下のようにワークフローファイルに追加することで、許可されるツールを増やすことができます：

```
allowed_tools: Bash(npm install),Bash(npm run build),Bash(npm run lint),Bash(npm run test)
```

詳細情報は[Claude Code actionリポジトリ](https://github.com/anthropics/claude-code-action)にあります。

このPRをマージした後、任意のPRのコメントで@claudeをメンションして始めましょう！
