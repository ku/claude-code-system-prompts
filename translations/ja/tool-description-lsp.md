<!--
name: 'ツール説明: LSP'
description: LSPツールの説明
ccVersion: 2.0.73
-->
Language Server Protocol（LSP）サーバーと対話して、コードインテリジェンス機能を取得します。

サポートされる操作:
- goToDefinition: シンボルが定義されている場所を見つける
- findReferences: シンボルへのすべての参照を見つける
- hover: シンボルのホバー情報（ドキュメント、型情報）を取得
- documentSymbol: ドキュメント内のすべてのシンボル（関数、クラス、変数）を取得
- workspaceSymbol: ワークスペース全体でシンボルを検索
- goToImplementation: インターフェースまたは抽象メソッドの実装を見つける
- prepareCallHierarchy: 位置にあるコール階層アイテムを取得（関数/メソッド）
- incomingCalls: 指定位置の関数を呼び出しているすべての関数/メソッドを見つける
- outgoingCalls: 指定位置の関数が呼び出しているすべての関数/メソッドを見つける

すべての操作に必要:
- filePath: 操作対象のファイル
- line: 行番号（1ベース、エディタで表示される通り）
- character: 文字オフセット（1ベース、エディタで表示される通り）

注意: LSPサーバーはファイルタイプに対して設定されている必要があります。利用可能なサーバーがない場合、エラーが返されます。
