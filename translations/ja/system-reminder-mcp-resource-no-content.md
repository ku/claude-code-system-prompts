<!--
name: 'システムリマインダー: MCPリソースコンテンツなし'
description: MCPリソースにコンテンツがない場合に表示
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
<mcp-resource server="${ATTACHMENT_OBJECT.server}" uri="${ATTACHMENT_OBJECT.uri}">(コンテンツなし)</mcp-resource>
