<!--
name: 'システムリマインダー: フック継続停止'
description: フックが継続を停止した時のメッセージ
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} フックが継続を停止しました: ${ATTACHMENT_OBJECT.message}
