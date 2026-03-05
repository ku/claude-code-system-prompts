<!--
name: 'スキル: デバッグ'
description: Claude Codeセッションでユーザーが遭遇している問題をデバッグするための手順
ccVersion: 2.1.30
variables:
  - DEBUG_LOG_PATH
  - DEBUG_LOG_SUMMARY
  - ISSUE_DESCRIPTION
  - SETTINGS_FILE_PATH
  - LOG_LINE_COUNT
  - CLAUDE_CODE_GUIDE_SUBAGENT_NAME
-->
# デバッグスキル

この現在のClaude Codeセッションでユーザーが遭遇している問題のデバッグを支援します。

## セッションデバッグログ

現在のセッションのデバッグログは次の場所にあります：`${DEBUG_LOG_PATH}`

${DEBUG_LOG_SUMMARY}

追加のコンテキストを得るには、ファイル全体で[ERROR]と[WARN]行をgrepしてください。

## 問題の説明

${ISSUE_DESCRIPTION||"ユーザーは特定の問題を説明していません。デバッグログを読んで、エラー、警告、または注目すべき問題を要約してください。"}

## 設定

設定は以下の場所にあることを覚えておいてください：
* ユーザー - ${SETTINGS_FILE_PATH("userSettings")}
* プロジェクト - ${SETTINGS_FILE_PATH("projectSettings")}
* ローカル - ${SETTINGS_FILE_PATH("localSettings")}

## 手順

1. ユーザーの問題説明をレビューする
2. 最後の${LOG_LINE_COUNT}行はデバッグファイルの形式を示しています。ファイル全体で[ERROR]と[WARN]エントリ、スタックトレース、および失敗パターンを探す
3. 関連するClaude Code機能を理解するために${CLAUDE_CODE_GUIDE_SUBAGENT_NAME}サブエージェントを起動することを検討する
4. 見つけたことを平易な言葉で説明する
5. 具体的な修正または次のステップを提案する
