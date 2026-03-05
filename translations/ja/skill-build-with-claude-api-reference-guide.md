<!--
name: 'スキル: Claude APIを使った開発（リファレンスガイド）'
description: クイックタスクナビゲーション付きの言語別リファレンスドキュメントを提示するためのテンプレート
ccVersion: 2.1.47
-->
## リファレンスドキュメント

検出された言語に関連するドキュメントが以下の`<doc>`タグ内に含まれています。各タグには元のファイルパスを示す`path`属性があります。これを使用して適切なセクションを見つけてください：

### クイックタスクリファレンス

**単一のテキスト分類/要約/抽出/Q&A：**
→ `{lang}/claude-api/README.md`を参照

**チャットUIまたはリアルタイムレスポンス表示：**
→ `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`を参照

**長時間の会話（コンテキストウィンドウを超える可能性あり）：**
→ `{lang}/claude-api/README.md`を参照 — コンパクションセクションを確認

**関数呼び出し / ツール使用 / エージェント：**
→ `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`を参照

**バッチ処理（レイテンシ非重視）：**
→ `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`を参照

**複数リクエストにまたがるファイルアップロード：**
→ `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`を参照

**組み込みツール（ファイル/ウェブ/ターミナル）を持つエージェント（PythonとTypeScriptのみ）：**
→ `{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`を参照

**エラーハンドリング：**
→ `shared/error-codes.md`を参照

**WebFetch経由の最新ドキュメント：**
→ URLについては`shared/live-sources.md`を参照
