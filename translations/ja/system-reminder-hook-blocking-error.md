<!--
name: 'システムリマインダー: フックブロッキングエラー'
description: ブロッキングフックコマンドからのエラー
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} フックブロッキングエラー（コマンド: "${ATTACHMENT_OBJECT.blockingError.command}"）: ${ATTACHMENT_OBJECT.blockingError.blockingError}
