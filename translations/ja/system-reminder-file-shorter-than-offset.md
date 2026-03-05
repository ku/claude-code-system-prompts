<!--
name: 'システムリマインダー: ファイルがオフセットより短い'
description: ファイル読み込みのオフセットがファイル長を超えた場合の警告
ccVersion: 2.1.18
variables:
  - RESULT_OBJECT
-->
<system-reminder>警告: ファイルは存在しますが、指定されたオフセット（${RESULT_OBJECT.file.startLine}）より短いです。ファイルの行数は ${RESULT_OBJECT.file.totalLines} 行です。</system-reminder>
