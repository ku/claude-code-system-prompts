<!--
name: 'システムリマインダー: エージェント呼び出し'
description: ユーザーがエージェントを呼び出したいことを通知
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
ユーザーはエージェント「${ATTACHMENT_OBJECT.agentType}」を呼び出したいと表明しました。必要なコンテキストを渡して、適切にエージェントを呼び出してください。
