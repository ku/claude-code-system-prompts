<!--
name: 'システムリマインダー: USD予算'
description: 現在のUSD予算統計
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
USD予算: $${ATTACHMENT_OBJECT.used}/$${ATTACHMENT_OBJECT.total}; 残り $${ATTACHMENT_OBJECT.remaining}
