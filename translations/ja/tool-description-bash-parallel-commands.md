<!--
name: 'ツール説明: Bash（並列コマンド）'
description: Bashツールの指示：独立したコマンドを並列ツール呼び出しとして実行する
ccVersion: 2.1.53
variables:
  - BASH_TOOL_NAME
-->
コマンドが独立しており、並列実行できる場合は、1つのメッセージ内で複数の${BASH_TOOL_NAME}ツール呼び出しを行ってください。例：「git status」と「git diff」を実行する必要がある場合、2つの${BASH_TOOL_NAME}ツール呼び出しを並列で含む1つのメッセージを送信します。
