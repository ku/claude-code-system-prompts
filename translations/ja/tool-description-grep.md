<!--
name: 'ツール説明: Grep'
description: ripgrepを使用したコンテンツ検索のためのツール説明
ccVersion: 2.0.14
variables:
  - GREP_TOOL_NAME
  - BASH_TOOL_NAME
  - TASK_TOOL_NAME
-->
ripgrepをベースにした強力な検索ツール

  使用方法:
  - 検索タスクには常に${GREP_TOOL_NAME}を使用してください。${BASH_TOOL_NAME}コマンドとして`grep`や`rg`を起動しないでください。${GREP_TOOL_NAME}ツールは正しい権限とアクセスのために最適化されています。
  - 完全な正規表現構文をサポート（例: "log.*Error", "function\\s+\\w+"）
  - globパラメータ（例: "*.js", "**/*.tsx"）またはtypeパラメータ（例: "js", "py", "rust"）でファイルをフィルタリング
  - 出力モード: "content"はマッチした行を表示、"files_with_matches"はファイルパスのみを表示（デフォルト）、"count"はマッチ数を表示
  - 複数ラウンドが必要なオープンエンドな検索には${TASK_TOOL_NAME}ツールを使用
  - パターン構文: ripgrep（grepではない）を使用 - リテラルの中括弧はエスケープが必要（Goコードで`interface{}`を見つけるには`interface\\{\\}`を使用）
  - 複数行マッチング: デフォルトでパターンは単一行内でのみマッチします。`struct \\{[\\s\\S]*?field`のような行をまたぐパターンには、`multiline: true`を使用
