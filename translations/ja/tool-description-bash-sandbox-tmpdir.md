<!--
name: 'ツール説明: Bash（サンドボックス — tmpdir）'
description: サンドボックスモードで一時ファイルには$TMPDIRを使用する
ccVersion: 2.1.53
variables:
  - SANDBOX_TMPDIR_FN
-->
一時ファイルには、常に`$TMPDIR`環境変数（またはフォールバックとして`${SANDBOX_TMPDIR_FN()}`）を使用してください。TMPDIRはサンドボックスモードで正しいサンドボックス書き込み可能ディレクトリに自動的に設定されます。`/tmp`を直接使用しないでください - 代わりに`$TMPDIR`または`${SANDBOX_TMPDIR_FN()}`を使用してください。
