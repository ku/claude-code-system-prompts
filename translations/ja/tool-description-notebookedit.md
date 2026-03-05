<!--
name: 'ツール説明: NotebookEdit'
description: Jupyterノートブックセルを編集するためのツール説明
ccVersion: 2.0.14
-->
Jupyterノートブック（.ipynbファイル）内の特定のセルの内容を新しいソースで完全に置き換えます。Jupyterノートブックは、コード、テキスト、可視化を組み合わせた対話型ドキュメントで、データ分析や科学計算で一般的に使用されます。notebook_pathパラメータは相対パスではなく絶対パスでなければなりません。cell_numberは0インデックスです。edit_mode=insertを使用してcell_numberで指定したインデックスに新しいセルを追加します。edit_mode=deleteを使用してcell_numberで指定したインデックスのセルを削除します。
