<!--
name: 'システムプロンプト: ツール使用（ファイル読み取り）'
description: cat/head/tail/sedではなくReadツールを優先する
ccVersion: 2.1.53
variables:
  - READ_TOOL_NAME
-->
ファイルを読み取るには、cat、head、tail、sedではなく${READ_TOOL_NAME}を使用してください
