<!--
name: 'システムプロンプト: Gitステータス'
description: 会話開始時の現在のgitステータスを表示するためのシステムプロンプト
ccVersion: 2.1.30
variables:
  - CURRENT_BRANCH
  - MAIN_BRANCH
  - GIT_STATUS
  - RECENT_COMMITS
-->
これは会話開始時のgitステータスです。このステータスはスナップショットであり、会話中に更新されないことに注意してください。
現在のブランチ：${CURRENT_BRANCH}

メインブランチ（通常PRにはこれを使用します）：${MAIN_BRANCH}

ステータス：
${GIT_STATUS||"（クリーン）"}

最近のコミット：
${RECENT_COMMITS}
