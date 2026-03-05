<!--
name: 'データ: ライブドキュメントソース'
description: 公式ソースから現在のClaude APIおよびAgent SDKドキュメントを取得するためのWebFetch URL
ccVersion: 2.1.63
-->
# ライブドキュメントソース

このファイルには、platform.claude.comおよびAgent SDKリポジトリから現在の情報を取得するためのWebFetch URLが含まれています。キャッシュされたコンテンツが最後に更新されて以降に変更された可能性のある最新データが必要な場合に使用してください。

## WebFetchを使用するタイミング

- ユーザーが「最新」または「現在」の情報を明示的に求めた場合
- キャッシュされたデータが正しくないと思われる場合
- ユーザーがキャッシュされたコンテンツでカバーされていない機能について尋ねた場合
- ユーザーが特定のAPIの詳細や例を必要としている場合

## Claude APIドキュメントURL

### モデルと価格

| トピック        | URL                                                                   | 抽出プロンプト                                                                  |
| --------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| モデル概要      | `https://platform.claude.com/docs/en/about-claude/models/overview.md` | "すべてのClaudeモデルの現在のモデルID、コンテキストウィンドウ、価格を抽出"      |
| 価格            | `https://platform.claude.com/docs/en/pricing.md`                      | "入力と出力の100万トークンあたりの現在の価格を抽出"                             |

### コア機能

| トピック          | URL                                                                          | 抽出プロンプト                                                                         |
| ----------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| 拡張思考          | `https://platform.claude.com/docs/en/build-with-claude/extended-thinking.md` | "拡張思考パラメータ、budget_tokens要件、使用例を抽出"                                  |
| アダプティブ思考  | `https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking.md` | "アダプティブ思考の設定、エフォートレベル、{{OPUS_NAME}}の使用例を抽出"                |
| エフォートパラメータ | `https://platform.claude.com/docs/en/build-with-claude/effort.md`         | "エフォートレベル、コストと品質のトレードオフ、思考との連携を抽出"                     |
| ツール使用        | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview.md`  | "ツール定義スキーマ、tool_choiceオプション、ツール結果の処理を抽出"                    |
| ストリーミング    | `https://platform.claude.com/docs/en/build-with-claude/streaming.md`         | "ストリーミングイベントタイプ、SDK例、ベストプラクティスを抽出"                        |
| プロンプトキャッシング | `https://platform.claude.com/docs/en/build-with-claude/prompt-caching.md` | "cache_controlの使用方法、価格メリット、実装例を抽出"                                  |

### メディアとファイル

| トピック    | URL                                                                    | 抽出プロンプト                                                    |
| ----------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- |
| ビジョン    | `https://platform.claude.com/docs/en/build-with-claude/vision.md`      | "サポートされる画像形式、サイズ制限、コード例を抽出"              |
| PDFサポート | `https://platform.claude.com/docs/en/build-with-claude/pdf-support.md` | "PDF処理機能、制限、例を抽出"                                     |

### API操作

| トピック         | URL                                                                         | 抽出プロンプト                                                                                          |
| ---------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| バッチ処理       | `https://platform.claude.com/docs/en/build-with-claude/batch-processing.md` | "バッチAPIエンドポイント、リクエスト形式、結果のポーリングを抽出"                                       |
| Files API        | `https://platform.claude.com/docs/en/build-with-claude/files.md`            | "ファイルのアップロード、ダウンロード、メッセージでの参照、サポートされるタイプとベータヘッダーを抽出" |
| トークンカウント | `https://platform.claude.com/docs/en/build-with-claude/token-counting.md`   | "トークンカウントAPIの使用方法と例を抽出"                                                               |
| レート制限       | `https://platform.claude.com/docs/en/api/rate-limits.md`                    | "ティアとモデル別の現在のレート制限を抽出"                                                              |
| エラー           | `https://platform.claude.com/docs/en/api/errors.md`                         | "HTTPエラーコード、意味、リトライガイダンスを抽出"                                                      |

### ツール

| トピック       | URL                                                                                    | 抽出プロンプト                                                                           |
| -------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| コード実行     | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool.md` | "コード実行ツールの設定、ファイルアップロード、コンテナ再利用、レスポンス処理を抽出"     |
| コンピュータ使用 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use.md`      | "コンピュータ使用ツールの設定、機能、実装例を抽出"                                       |

### 高度な機能

| トピック           | URL                                                                           | 抽出プロンプト                                              |
| ------------------ | ----------------------------------------------------------------------------- | ----------------------------------------------------------- |
| 構造化出力         | `https://platform.claude.com/docs/en/build-with-claude/structured-outputs.md` | "output_config.formatの使用方法とスキーマの強制を抽出"      |
| コンパクション     | `https://platform.claude.com/docs/en/build-with-claude/compaction.md`         | "コンパクションの設定、トリガー設定、ストリーミングとの組み合わせを抽出" |
| 引用               | `https://platform.claude.com/docs/en/build-with-claude/citations.md`          | "引用形式と実装を抽出"                                      |
| コンテキストウィンドウ | `https://platform.claude.com/docs/en/build-with-claude/context-windows.md` | "コンテキストウィンドウサイズとトークン管理を抽出"          |

---

## Claude API SDKリポジトリ

| SDK        | URL                                                       | 説明                           |
| ---------- | --------------------------------------------------------- | ------------------------------ |
| Python     | `https://github.com/anthropics/anthropic-sdk-python`     | `anthropic` pipパッケージソース |
| TypeScript | `https://github.com/anthropics/anthropic-sdk-typescript` | `@anthropic-ai/sdk` npmソース   |
| Java       | `https://github.com/anthropics/anthropic-sdk-java`       | `anthropic-java` Mavenソース    |
| Go         | `https://github.com/anthropics/anthropic-sdk-go`         | Goモジュールソース              |
| Ruby       | `https://github.com/anthropics/anthropic-sdk-ruby`       | `anthropic` gemソース           |
| C#         | `https://github.com/anthropics/anthropic-sdk-csharp`     | NuGetパッケージソース           |
| PHP        | `https://github.com/anthropics/anthropic-sdk-php`        | Composerパッケージソース        |

---

## Agent SDKドキュメントURL

### コアドキュメント

| トピック             | URL                                                         | 抽出プロンプト                                                  |
| -------------------- | ----------------------------------------------------------- | --------------------------------------------------------------- |
| Agent SDK概要        | `https://platform.claude.com/docs/en/agent-sdk.md`          | "Agent SDKの概要、主要機能、ユースケースを抽出"                 |
| Agent SDK Python     | `https://github.com/anthropics/claude-agent-sdk-python`     | "Python SDKのインストール、インポート、基本的な使用方法を抽出"  |
| Agent SDK TypeScript | `https://github.com/anthropics/claude-agent-sdk-typescript` | "TypeScript SDKのインストール、インポート、基本的な使用方法を抽出" |

### SDKリファレンス（GitHub README）

| トピック       | URL                                                                                       | 抽出プロンプト                                               |
| -------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Python SDK     | `https://raw.githubusercontent.com/anthropics/claude-agent-sdk-python/main/README.md`     | "Python SDK APIリファレンス、クラス、メソッドを抽出"         |
| TypeScript SDK | `https://raw.githubusercontent.com/anthropics/claude-agent-sdk-typescript/main/README.md` | "TypeScript SDK APIリファレンス、型、関数を抽出"             |

### npm/PyPIパッケージ

| パッケージ                          | URL                                                            | 説明                      |
| ----------------------------------- | -------------------------------------------------------------- | ------------------------- |
| claude-agent-sdk (Python)           | `https://pypi.org/project/claude-agent-sdk/`                   | PyPI上のPythonパッケージ  |
| @anthropic-ai/claude-agent-sdk (TS) | `https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk` | npm上のTypeScriptパッケージ |

### GitHubリポジトリ

| リソース       | URL                                                         | 説明                            |
| -------------- | ----------------------------------------------------------- | ------------------------------- |
| Python SDK     | `https://github.com/anthropics/claude-agent-sdk-python`     | Pythonパッケージソース          |
| TypeScript SDK | `https://github.com/anthropics/claude-agent-sdk-typescript` | TypeScript/Node.jsパッケージソース |
| MCPサーバー    | `https://github.com/modelcontextprotocol`                   | 公式MCPサーバー実装             |

---

## フォールバック戦略

WebFetchが失敗した場合（ネットワークの問題、URLが変更された場合）：

1. 言語固有のファイルからキャッシュされたコンテンツを使用（キャッシュ日付に注意）
2. データが古い可能性があることをユーザーに通知
3. platform.claude.comまたはGitHubリポジトリを直接確認するよう提案
