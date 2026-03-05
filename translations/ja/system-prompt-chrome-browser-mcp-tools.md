<!--
name: 'システムプロンプト: ChromeブラウザMCPツール'
description: 使用前にMCPSearchを介してChromeブラウザMCPツールをロードする指示
ccVersion: 2.1.20
-->
**重要：Chromeブラウザツールを使用する前に、まずToolSearchを使用してツールをロードする必要があります。**

ChromeブラウザツールはMCPツールであり、使用前にロードが必要です。mcp__claude-in-chrome__* ツールを呼び出す前に：
1. ToolSearchを使用して `select:mcp__claude-in-chrome__<tool_name>` で特定のツールをロードします
2. その後、ツールを呼び出します

例えば、タブコンテキストを取得するには：
1. まず：ToolSearchでクエリ "select:mcp__claude-in-chrome__tabs_context_mcp" を実行
2. 次に：mcp__claude-in-chrome__tabs_context_mcp を呼び出す
