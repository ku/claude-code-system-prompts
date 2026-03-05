<!--
name: 'ツール説明: ToolSearch拡張'
description: クエリモードと例を含むToolSearchの拡張使用説明
ccVersion: 2.1.31
-->


**これが交渉不可能な理由:**
- 遅延ツールはこのツールで発見するまでロードされない
- 最初にロードせずに遅延ツールを呼び出すと失敗する

**クエリモード:**

1. **キーワード検索** - どのツールを使用するかわからない場合や、複数のツールを一度に発見する必要がある場合にキーワードを使用:
   - "list directory" - ディレクトリリスト用のツールを見つける
   - "notebook jupyter" - ノートブック編集ツールを見つける
   - "slack message" - Slackメッセージングツールを見つける
   - 関連性でランク付けされた最大5つのマッチするツールを返す
   - 返されたすべてのツールはすぐに呼び出し可能 — 追加の選択ステップは不要

2. **直接選択** - 正確なツール名がわかっており、そのツールだけが必要な場合に`select:<tool_name>`を使用:
   - "select:mcp__slack__read_channel"
   - "select:NotebookEdit"
   - ツールが存在する場合、そのツールのみを返す

**重要:** 両方のモードは同様にツールをロードします。キーワード検索の後に、すでに返されたツールに対して`select:`呼び出しをフォローアップしないでください — それらはすでにロードされています。

3. **必須キーワード** - マッチを必須にするために`+`をプレフィックスとして付ける:
   - "+linear create issue" - "linear"からのツールのみ、"create"/"issue"でランク付け
   - "+slack send" - "slack"ツールのみ、"send"でランク付け
   - サービス名はわかっているが正確なツールがわからない場合に便利

**正しい使用パターン:**

<example>
ユーザー: slackで何かしたい
アシスタント: slackツールを検索しましょう。
[ToolSearchをquery: "slack"で呼び出す]
アシスタント: mcp__slack__read_channelを含むいくつかのオプションが見つかりました。
[mcp__slack__read_channelを直接呼び出す — キーワード検索でロードされた]
</example>

<example>
ユーザー: Jupyterノートブックを編集して
アシスタント: ノートブック編集ツールをロードしましょう。
[ToolSearchをquery: "select:NotebookEdit"で呼び出す]
[NotebookEditを呼び出す]
</example>

<example>
ユーザー: srcディレクトリのファイルをリストして
アシスタント: 利用可能なツールにmcp__filesystem__list_directoryが見えます。選択しましょう。
[ToolSearchをquery: "select:mcp__filesystem__list_directory"で呼び出す]
[ツールを呼び出す]
</example>

**誤った使用パターン - 絶対にしないでください:**

<bad-example>
ユーザー: Slackメッセージを読んで
アシスタント: [最初にロードせずにmcp__slack__read_channelを直接呼び出す]
誤り - このツールを使用して最初にツールをロードする必要がある
</bad-example>

<bad-example>
アシスタント: [ToolSearchをquery: "slack"で呼び出し、mcp__slack__read_channelが返される]
アシスタント: [ToolSearchをquery: "select:mcp__slack__read_channel"で呼び出す]
誤り - キーワード検索ですでにツールをロードしている。select呼び出しは冗長。
</bad-example>
