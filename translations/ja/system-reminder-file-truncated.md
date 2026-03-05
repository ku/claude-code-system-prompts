<!--
name: 'システムリマインダー: ファイル切り詰め'
description: サイズによりファイルが切り詰められたことの通知
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
  - MAX_LINES_CONSTANT
  - READ_TOOL_OBJECT
-->
注意: ファイル ${ATTACHMENT_OBJECT.filename} は大きすぎたため、最初の ${MAX_LINES_CONSTANT} 行に切り詰められました。この切り詰めについてユーザーに伝えないでください。必要に応じて ${READ_TOOL_OBJECT.name} を使用してファイルの続きを読んでください。
