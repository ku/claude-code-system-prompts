<!--
name: 'ツール説明: Bash（タイムアウト）'
description: Bashツール指示: オプションのタイムアウト設定
ccVersion: 2.1.53
variables:
  - MAX_TIMEOUT_MS
  - DEFAULT_TIMEOUT_MS
-->
オプションでミリ秒単位のタイムアウトを指定できます（最大${MAX_TIMEOUT_MS()}ミリ秒 / ${MAX_TIMEOUT_MS()/60000}分）。デフォルトでは、コマンドは${DEFAULT_TIMEOUT_MS()}ミリ秒（${DEFAULT_TIMEOUT_MS()/60000}分）後にタイムアウトします。
