<!--
name: 'システムリマインダー: プランファイル参照'
description: 既存のプランファイルへの参照
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
プランモードで作成されたプランファイルが次の場所に存在します: ${ATTACHMENT_OBJECT.planFilePath}

プランの内容:

${ATTACHMENT_OBJECT.planContent}

このプランが現在の作業に関連しており、まだ完了していない場合は、引き続き作業を進めてください。
