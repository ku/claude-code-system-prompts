<!--
name: 'データ: ツール使用の概念'
description: ツール定義、ツール選択、ベストプラクティスを含むClaude APIでのツール使用の概念的基礎
ccVersion: 2.1.63
-->
# ツール使用の概念

このファイルでは、Claude APIでのツール使用の概念的基礎について説明します。言語固有のコード例については、`python/`、`typescript/`、または他の言語フォルダを参照してください。

## ユーザー定義ツール

### ツール定義の構造

> **注意:** ツールランナー（ベータ版）を使用する場合、ツールスキーマは関数シグネチャ（Python）、Zodスキーマ（TypeScript）、アノテーション付きクラス（Java）、`jsonschema`構造体タグ（Go）、または`BaseTool`サブクラス（Ruby）から自動的に生成されます。以下の生のJSONスキーマ形式は、手動アプローチまたはツールランナーをサポートしていないSDK用です。

各ツールには名前、説明、および入力用のJSONスキーマが必要です：

```json
{
  "name": "get_weather",
  "description": "場所の現在の天気を取得",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "市と州、例: San Francisco, CA"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "温度単位"
      }
    },
    "required": ["location"]
  }
}
```

**ツール定義のベストプラクティス:**

- 明確で説明的な名前を使用（例: `get_weather`、`search_database`、`send_email`）
- 詳細な説明を記述 — Claudeはこれらを使用してツールを使用するタイミングを決定
- 各プロパティに説明を含める
- 固定値セットを持つパラメータには`enum`を使用
- 本当に必要なパラメータを`required`にマーク。その他はデフォルト値でオプションに

---

### ツール選択オプション

Claudeがツールを使用するタイミングを制御：

| 値                                | 動作                                      |
| --------------------------------- | ----------------------------------------- |
| `{"type": "auto"}`                | Claudeがツールを使用するかどうかを決定（デフォルト） |
| `{"type": "any"}`                 | Claudeは少なくとも1つのツールを使用する必要がある |
| `{"type": "tool", "name": "..."}` | Claudeは指定されたツールを使用する必要がある |
| `{"type": "none"}`                | Claudeはツールを使用できない              |

どの`tool_choice`値でも、`"disable_parallel_tool_use": true`を含めることで、Claudeが1回のレスポンスで最大1つのツールを使用するように強制できます。デフォルトでは、Claudeは1回のレスポンスで複数のツール呼び出しをリクエストすることがあります。

---

### ツールランナー vs 手動ループ

**ツールランナー（推奨）:** SDKのツールランナーはエージェンティックループを自動的に処理します — APIを呼び出し、ツール使用リクエストを検出し、ツール関数を実行し、結果をClaudeにフィードバックし、Claudeがツールの呼び出しを停止するまで繰り返します。Python、TypeScript、Java、Go、Ruby SDK（ベータ版）で利用可能。

**手動エージェンティックループ:** ループを細かく制御する必要がある場合（例：カスタムログ、条件付きツール実行、ヒューマンインザループ承認）に使用します。`stop_reason == "end_turn"`になるまでループし、tool_useブロックを保持するために常に完全な`response.content`を追加し、各`tool_result`に一致する`tool_use_id`を含めることを確認してください。

**サーバーサイドツールの停止理由:** サーバーサイドツール（コード実行、Web検索など）を使用する場合、APIはサーバーサイドのサンプリングループを実行します。このループがデフォルトの制限である10回の反復に達した場合、レスポンスは`stop_reason: "pause_turn"`になります。続行するには、ユーザーメッセージとアシスタントレスポンスを再送信し、別のAPIリクエストを行ってください — サーバーは中断したところから再開します。「続けてください」のような追加のユーザーメッセージを追加しないでください — APIは末尾の`server_tool_use`ブロックを検出し、自動的に再開することを認識します。

```python
# エージェンティックループでpause_turnを処理
if response.stop_reason == "pause_turn":
    messages = [
        {"role": "user", "content": user_query},
        {"role": "assistant", "content": response.content},
    ]
    # 別のAPIリクエストを行う — サーバーが自動的に再開
    response = client.messages.create(
        model="{{OPUS_ID}}", messages=messages, tools=tools
    )
```

無限ループを防ぐために`max_continuations`制限（例：5）を設定してください。完全なガイドについては、`https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons`を参照してください。

> **セキュリティ:** ツールランナーは、Claudeがリクエストするたびにツール関数を自動的に実行します。副作用のあるツール（メール送信、データベース変更、金融取引）では、ツール関数内で入力を検証し、破壊的な操作には確認を要求することを検討してください。各ツール実行前にヒューマンインザループ承認が必要な場合は、手動エージェンティックループを使用してください。

---

### ツール結果の処理

Claudeがツールを使用すると、レスポンスには`tool_use`ブロックが含まれます。以下を行う必要があります：

1. 提供された入力でツールを実行
2. 結果を`tool_result`メッセージで返信
3. 会話を続行

**ツール結果のエラー処理:** ツール実行が失敗した場合、`"is_error": true`を設定し、情報的なエラーメッセージを提供してください。Claudeは通常、エラーを認識し、別のアプローチを試みるか、説明を求めます。

**複数のツール呼び出し:** Claudeは1回のレスポンスで複数のツールをリクエストすることがあります。続行する前にすべてを処理してください — すべての結果を1つの`user`メッセージで送り返してください。

---

## サーバーサイドツール: コード実行

コード実行ツールを使用すると、Claudeは安全なサンドボックス化されたコンテナ内でコードを実行できます。ユーザー定義ツールとは異なり、サーバーサイドツールはAnthropicのインフラストラクチャ上で実行されます — クライアントサイドで何も実行する必要はありません。ツール定義を含めるだけで、Claudeが残りを処理します。

### 重要事項

- 分離されたコンテナで実行（1 CPU、5 GiB RAM、5 GiBディスク）
- インターネットアクセスなし（完全にサンドボックス化）
- データサイエンスライブラリがプリインストールされたPython 3.11
- コンテナは30日間保持され、リクエスト間で再利用可能
- Web検索/Webフェッチツールと一緒に使用する場合は無料。それ以外の場合、組織あたり月1,550時間の無料時間後、$0.05/時間

### ツール定義

ツールにはスキーマは必要ありません — `tools`配列で宣言するだけです：

```json
{
  "type": "code_execution_20260120",
  "name": "code_execution"
}
```

Claudeは自動的に`bash_code_execution`（シェルコマンドの実行）と`text_editor_code_execution`（ファイルの作成/表示/編集）へのアクセスを取得します。

### プリインストールされたPythonライブラリ

- **データサイエンス**: pandas、numpy、scipy、scikit-learn、statsmodels
- **可視化**: matplotlib、seaborn
- **ファイル処理**: openpyxl、xlsxwriter、pillow、pypdf、pdfplumber、python-docx、python-pptx
- **数学**: sympy、mpmath
- **ユーティリティ**: tqdm、python-dateutil、pytz、sqlite3

追加のパッケージは`pip install`で実行時にインストールできます。

### アップロードのサポートされるファイルタイプ

| タイプ | 拡張子                             |
| ------ | ---------------------------------- |
| データ | CSV、Excel（.xlsx/.xls）、JSON、XML |
| 画像   | JPEG、PNG、GIF、WebP               |
| テキスト | .txt、.md、.py、.jsなど          |

### コンテナの再利用

リクエスト間で状態（ファイル、インストールされたパッケージ、変数）を維持するためにコンテナを再利用します。最初のレスポンスから`container_id`を抽出し、後続のリクエストに渡します。

### レスポンス構造

レスポンスには、テキストとツール結果ブロックが交互に含まれます：

- `text` — Claudeの説明
- `server_tool_use` — Claudeが行っていること
- `bash_code_execution_tool_result` — コード実行出力（成功/失敗は`return_code`を確認）
- `text_editor_code_execution_tool_result` — ファイル操作結果

> **セキュリティ:** ダウンロードしたファイルをディスクに書き込む前に、パストラバーサル攻撃を防ぐために`os.path.basename()` / `path.basename()`でファイル名をサニタイズしてください。専用の出力ディレクトリにファイルを書き込んでください。

---

## サーバーサイドツール: Web検索とWebフェッチ

Web検索とWebフェッチを使用すると、ClaudeはWebを検索し、ページコンテンツを取得できます。サーバーサイドで実行されます — ツール定義を含めるだけで、Claudeがクエリ、フェッチ、結果処理を自動的に処理します。

### ツール定義

```json
[
  { "type": "web_search_20260209", "name": "web_search" },
  { "type": "web_fetch_20260209", "name": "web_fetch" }
]
```

### 動的フィルタリング（Opus 4.6 / Sonnet 4.6）

`web_search_20260209`と`web_fetch_20260209`バージョンは**動的フィルタリング**をサポートしています — Claudeは検索結果がコンテキストウィンドウに到達する前にフィルタリングするコードを書いて実行し、精度とトークン効率を向上させます。動的フィルタリングはこれらのツールバージョンに組み込まれており、自動的に有効になります。`code_execution`ツールを別途宣言したり、ベータヘッダーを渡す必要はありません。

```json
{
  "tools": [
    { "type": "web_search_20260209", "name": "web_search" },
    { "type": "web_fetch_20260209", "name": "web_fetch" }
  ]
}
```

動的フィルタリングなしの場合、以前の`web_search_20250305`バージョンも利用可能です。

> **注意:** アプリケーションがWeb検索とは独立してコード実行（データ分析、ファイル処理、可視化）を必要とする場合にのみ、スタンドアロンの`code_execution`ツールを含めてください。`_20260209` Webツールと一緒に含めると、モデルを混乱させる可能性のある2つ目の実行環境が作成されます。

---

## サーバーサイドツール: プログラマティックツール呼び出し

プログラマティックツール呼び出しを使用すると、Claudeはコード内で複雑なマルチツールワークフローを実行し、中間結果をコンテキストウィンドウから除外できます。Claudeはツールを直接呼び出すコードを書き、マルチステップ操作のトークン使用量を削減します。

完全なドキュメントについては、WebFetchを使用してください：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling`

---

## サーバーサイドツール: ツール検索

ツール検索ツールを使用すると、Claudeはすべての定義をコンテキストウィンドウに読み込むことなく、大規模なライブラリからツールを動的に発見できます。多くのツールがあるが、特定のクエリに関連するのはわずかな場合に便利です。

完全なドキュメントについては、WebFetchを使用してください：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool`

---

## ツール使用の例

使用パターンを示し、パラメータエラーを減らすために、ツール定義に直接サンプルツール呼び出しを提供できます。これにより、Claudeが特に複雑なスキーマを持つツールの入力を正しくフォーマットする方法を理解しやすくなります。

完全なドキュメントについては、WebFetchを使用してください：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use`

---

## サーバーサイドツール: コンピュータ使用

コンピュータ使用を使用すると、Claudeはデスクトップ環境（スクリーンショット、マウス、キーボード）と対話できます。Anthropicがホスト（コード実行のようにサーバーサイド）することも、セルフホスト（環境を提供し、クライアントサイドでアクションを実行）することもできます。

完全なドキュメントについては、WebFetchを使用してください：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/computer-use/overview`

---

## クライアントサイドツール: メモリ

メモリツールを使用すると、Claudeはメモリファイルディレクトリを通じて会話間で情報を保存および取得できます。Claudeはセッション間で保持されるファイルを作成、読み取り、更新、削除できます。

### 重要事項

- クライアントサイドツール — 実装を通じてストレージを制御
- サポートされるコマンド: `view`、`create`、`str_replace`、`insert`、`delete`、`rename`
- `/memories`ディレクトリ内のファイルを操作
- SDKはメモリバックエンドを実装するためのヘルパークラス/関数を提供

> **セキュリティ:** メモリファイルにAPIキー、パスワード、トークン、その他のシークレットを保存しないでください。個人識別情報（PII）には注意してください — ユーザーデータを永続化する前にデータプライバシー規制（GDPR、CCPA）を確認してください。リファレンス実装には組み込みのアクセス制御はありません。マルチユーザーシステムでは、ツールハンドラーにユーザーごとのメモリディレクトリと認証を実装してください。

完全な実装例については、WebFetchを使用してください：

- ドキュメント: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md`

---

## 構造化出力

構造化出力は、Claudeのレスポンスを特定のJSONスキーマに従うように制約し、有効でパース可能な出力を保証します。これは別のツールではありません — Messages APIのレスポンス形式および/またはツールパラメータ検証を強化します。

2つの機能が利用可能です：

- **JSON出力** (`output_config.format`): Claudeのレスポンス形式を制御
- **厳密なツール使用** (`strict: true`): 有効なツールパラメータスキーマを保証

**サポートされるモデル:** {{OPUS_NAME}}、{{SONNET_NAME}}、および{{HAIKU_NAME}}。レガシーモデル（Claude Opus 4.5、Claude Opus 4.1）も構造化出力をサポートしています。

> **推奨:** スキーマに対するレスポンスを自動的に検証する`client.messages.parse()`を使用してください。`messages.create()`を直接使用する場合は、`output_config: {format: {...}}`を使用してください。`output_format`便利パラメータも一部のSDKメソッド（例：`.parse()`）で受け入れられますが、`output_config.format`が正規のAPIレベルのパラメータです。

### JSONスキーマの制限

**サポートされる:**

- 基本型: object、array、string、integer、number、boolean、null
- `enum`、`const`、`anyOf`、`allOf`、`$ref`/`$def`
- 文字列形式: `date-time`、`time`、`date`、`duration`、`email`、`hostname`、`uri`、`ipv4`、`ipv6`、`uuid`
- `additionalProperties: false`（すべてのオブジェクトで必須）

**サポートされない:**

- 再帰的スキーマ
- 数値制約（`minimum`、`maximum`、`multipleOf`）
- 文字列制約（`minLength`、`maxLength`）
- 複雑な配列制約
- `false`以外に設定された`additionalProperties`

PythonおよびTypeScript SDKは、サポートされていない制約をスキーマから削除してAPIに送信し、クライアントサイドで検証することで自動的に処理します。

### 重要な注意事項

- **最初のリクエストのレイテンシ**: 新しいスキーマは1回限りのコンパイルコストが発生します。同じスキーマを使用した後続のリクエストは24時間のキャッシュを使用します。
- **拒否**: Claudeが安全上の理由で拒否した場合（`stop_reason: "refusal"`）、出力がスキーマに一致しない可能性があります。
- **トークン制限**: `stop_reason: "max_tokens"`の場合、出力が不完全な可能性があります。`max_tokens`を増やしてください。
- **互換性なし**: 引用（400エラーを返す）、メッセージプレフィリング。
- **互換性あり**: Batches API、ストリーミング、トークンカウント、拡張思考。

---

## 効果的なツール使用のためのヒント

1. **詳細な説明を提供**: Claudeはツールを使用するタイミングと方法を理解するために説明に大きく依存しています
2. **具体的なツール名を使用**: `weather`より`get_current_weather`の方が良い
3. **入力を検証**: 実行前にツール入力を常に検証
4. **エラーを適切に処理**: Claudeが適応できるように情報的なエラーメッセージを返す
5. **ツール数を制限**: ツールが多すぎるとモデルが混乱する可能性がある — セットを集中させる
6. **ツールの相互作用をテスト**: Claudeがさまざまなシナリオでツールを正しく使用することを確認

詳細なツール使用ドキュメントについては、WebFetchを使用してください：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview`
