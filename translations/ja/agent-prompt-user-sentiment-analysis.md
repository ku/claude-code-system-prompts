<!--
name: 'エージェントプロンプト: ユーザー感情分析'
description: ユーザーのフラストレーションとPR作成リクエストを分析するためのシステムプロンプト
ccVersion: 2.0.14
variables:
  - CONVERSATION_HISTORY
-->
以下のユーザーとアシスタント間の会話を分析してください（アシスタントの応答は非表示です）。

${CONVERSATION_HISTORY}

以下についてステップバイステップで考えてください：
1. ユーザーはアシスタントに対してフラストレーションを感じているように見えますか？繰り返しの修正、否定的な言葉遣いなどの兆候を探してください。
2. ユーザーはGitHubにプルリクエストを送信/作成/プッシュすることを明示的に依頼しましたか？これは、一緒にコードを作業したり変更を準備したりするのではなく、実際にリポジトリにPRを提出したいことを意味します。「create a pr」「send a pull request」「push a pr」「open a pr」「submit a pr to github」などの明示的なリクエストを探してください。一緒にPRに取り組む、PRの準備をする、またはPRの内容について議論するという言及はカウントしないでください。

分析に基づいて、以下を出力してください：
<frustrated>true/false</frustrated>
<pr_request>true/false</pr_request>
