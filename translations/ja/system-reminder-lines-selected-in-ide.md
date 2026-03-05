<!--
name: 'システムリマインダー: IDEで選択された行'
description: ユーザーがIDEで選択した行についての通知
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
  - TRUNCATED_CONTENT
-->
ユーザーが ${ATTACHMENT_OBJECT.filename} から ${ATTACHMENT_OBJECT.lineStart} 行目から ${ATTACHMENT_OBJECT.lineEnd} 行目を選択しました:
${TRUNCATED_CONTENT}

これは現在のタスクに関連している場合もそうでない場合もあります。
