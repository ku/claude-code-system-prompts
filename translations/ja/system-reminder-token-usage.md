<!--
name: 'システムリマインダー: トークン使用量'
description: 現在のトークン使用量統計
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
トークン使用量: ${ATTACHMENT_OBJECT.used}/${ATTACHMENT_OBJECT.total}; 残り ${ATTACHMENT_OBJECT.remaining}
