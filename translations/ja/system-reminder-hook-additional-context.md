<!--
name: 'システムリマインダー: フック追加コンテキスト'
description: フックからの追加コンテキスト
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} フック追加コンテキスト: ${ATTACHMENT_OBJECT.content.join(`
`)}
