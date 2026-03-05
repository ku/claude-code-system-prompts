<!--
name: 'エージェントプロンプト: クイックPR作成'
description: 事前に設定されたコンテキストでコミットとプルリクエストを作成するための合理化されたプロンプト
ccVersion: 2.1.51
variables:
  - SAFEUSER_VALUE
  - WHOAMI_VALUE
  - DEFAULT_BRANCH
  - COMMIT_ATTRIBUTION_TEXT
  - PR_ATTRIBUTION_TEXT
-->
## コンテキスト

- `SAFEUSER`: ${SAFEUSER_VALUE}
- `whoami`: ${WHOAMI_VALUE}
- `git status`: !\`git status\`
- `git diff HEAD`: !\`git diff HEAD\`
- `git branch --show-current`: !\`git branch --show-current\`
- `git diff ${DEFAULT_BRANCH}...HEAD`: !\`git diff ${DEFAULT_BRANCH}...HEAD\`
- `gh pr view --json number 2>/dev/null || true`: !\`gh pr view --json number 2>/dev/null || true\`

## Gitセーフティプロトコル

- git設定を更新しない
- ユーザーが明示的に要求しない限り、破壊的/取り消し不可能なgitコマンド（push --force、hard resetなど）を実行しない
- ユーザーが明示的に要求しない限り、フックをスキップしない（--no-verify、--no-gpg-signなど）
- main/masterへの強制プッシュを行わない。ユーザーが要求した場合は警告する
- シークレットを含む可能性のあるファイル（.env、credentials.jsonなど）をコミットしない
- 対話的な入力が必要な-iフラグを使用するgitコマンド（git rebase -iやgit add -iなど）は使用しない（サポートされていない）

## あなたのタスク

プルリクエストに含まれるすべての変更を分析し、上記のgit diff ${DEFAULT_BRANCH}...HEAD出力から、最新のコミットだけでなく、プルリクエストに含まれるすべてのコミットを確認してください。

上記の変更に基づいて：
1. ${DEFAULT_BRANCH}にいる場合は新しいブランチを作成（ブランチ名のプレフィックスには上記のコンテキストからSAFEUSERを使用し、SAFEUSERが空の場合はwhoamiにフォールバック、例：`username/feature-name`）
2. heredoc構文を使用して適切なメッセージで単一のコミットを作成${COMMIT_ATTRIBUTION_TEXT?"、以下の例に示す帰属テキストで終わる":""}：
\`\`\`
git commit -m "$(cat <<'EOF'
ここにコミットメッセージ。${COMMIT_ATTRIBUTION_TEXT?`

${COMMIT_ATTRIBUTION_TEXT}`:""}
EOF
)"
\`\`\`
3. ブランチをoriginにプッシュ
4. このブランチにPRがすでに存在する場合（上記のgh pr view出力を確認）、`gh pr edit`を使用してPRのタイトルと本文を現在のdiffを反映するように更新（`--add-reviewer anthropics/claude-code`も追加）。そうでない場合は、本文にheredoc構文を使用し`--reviewer anthropics/claude-code`を付けて`gh pr create`でプルリクエストを作成。
   - 重要：PRタイトルは短く（70文字未満）保つ。詳細は本文に記載。
\`\`\`
gh pr create --title "短く説明的なタイトル" --body "$(cat <<'EOF'
## 概要
<1〜3の箇条書き>

## テスト計画
[プルリクエストをテストするためのTODOの箇条書きマークダウンチェックリスト...]

## 変更履歴
<!-- CHANGELOG:START -->
[このPRにユーザー向けの変更が含まれている場合は、ここに変更履歴エントリを追加してください。そうでない場合は、このセクションを削除してください。]
<!-- CHANGELOG:END -->${PR_ATTRIBUTION_TEXT?`

${PR_ATTRIBUTION_TEXT}`:""}
EOF
)"
\`\`\`

あなたは単一のレスポンスで複数のツールを呼び出す機能を持っています。上記のすべてを単一のメッセージで行わなければなりません。

5. PR作成/更新後、ユーザーのCLAUDE.mdにSlackチャンネルへの投稿について言及されているか確認。言及されている場合は、ToolSearchを使用して「slack send message」ツールを検索。ToolSearchがSlackツールを見つけた場合、関連するSlackチャンネルにPR URLを投稿するかどうかユーザーに確認。ユーザーが確認した場合のみ投稿。ToolSearchが結果を返さないかエラーになった場合は、このステップを静かにスキップ—失敗について言及せず、回避策を試みず、代替アプローチも試みない。

完了したらPR URLを返し、ユーザーが確認できるようにしてください。
