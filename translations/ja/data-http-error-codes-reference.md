<!--
name: 'データ: HTTPエラーコードリファレンス'
description: Claude APIが返すHTTPエラーコード、一般的な原因、対処方法のリファレンス
ccVersion: 2.1.63
-->
# HTTPエラーコードリファレンス

このファイルでは、Claude APIが返すHTTPエラーコード、一般的な原因、対処方法について説明します。言語固有のエラー処理の例については、`python/`または`typescript/`フォルダを参照してください。

## エラーコード一覧

| コード | エラータイプ              | リトライ可能 | 一般的な原因                         |
| ---- | ----------------------- | --------- | ------------------------------------ |
| 400  | `invalid_request_error` | いいえ    | 無効なリクエスト形式またはパラメータ |
| 401  | `authentication_error`  | いいえ    | 無効または欠落しているAPIキー        |
| 403  | `permission_error`      | いいえ    | APIキーに権限がない                  |
| 404  | `not_found_error`       | いいえ    | 無効なエンドポイントまたはモデルID   |
| 413  | `request_too_large`     | いいえ    | リクエストがサイズ制限を超過         |
| 429  | `rate_limit_error`      | はい      | リクエスト過多                       |
| 500  | `api_error`             | はい      | Anthropicサービスの問題              |
| 529  | `overloaded_error`      | はい      | APIが一時的に過負荷                  |

## 詳細なエラー情報

### 400 Bad Request

**原因:**

- リクエストボディのJSON形式が不正
- 必須パラメータの欠落（`model`、`max_tokens`、`messages`）
- 無効なパラメータ型（例：整数が期待される場所に文字列）
- 空のmessages配列
- messagesがuser/assistantで交互になっていない

**エラー例:**

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate between \"user\" and \"assistant\""
  }
}
```

**対処方法:** 送信前にリクエスト構造を検証してください。以下を確認：

- `model`が有効なモデルIDである
- `max_tokens`が正の整数である
- `messages`配列が空でなく、正しく交互になっている

---

### 401 Unauthorized

**原因:**

- `x-api-key`ヘッダーまたは`Authorization`ヘッダーの欠落
- 無効なAPIキー形式
- 失効または削除されたAPIキー

**対処方法:** `ANTHROPIC_API_KEY`環境変数が正しく設定されていることを確認してください。

---

### 403 Forbidden

**原因:**

- APIキーがリクエストされたモデルへのアクセス権を持っていない
- 組織レベルの制限
- ベータアクセスなしでベータ機能にアクセスしようとしている

**対処方法:** コンソールでAPIキーの権限を確認してください。異なるAPIキーが必要な場合や、特定の機能へのアクセスをリクエストする必要がある場合があります。

---

### 404 Not Found

**原因:**

- モデルIDの入力ミス（例：`claude-sonnet-4.6`ではなく`claude-sonnet-4-6`）
- 非推奨のモデルIDの使用
- 無効なAPIエンドポイント

**対処方法:** モデルドキュメントの正確なモデルIDを使用してください。エイリアス（例：`{{OPUS_ID}}`）を使用することもできます。

---

### 413 Request Too Large

**原因:**

- リクエストボディが最大サイズを超過
- 入力のトークン数が多すぎる
- 画像データが大きすぎる

**対処方法:** 入力サイズを削減してください — 会話履歴を切り詰める、画像を圧縮/リサイズする、大きなドキュメントをチャンクに分割するなど。

---

### 400 バリデーションエラー

一部の400エラーは、パラメータ検証に関連しています：

- `max_tokens`がモデルの制限を超過
- 無効な`temperature`値（0.0〜1.0である必要）
- 拡張思考で`budget_tokens` >= `max_tokens`
- 無効なツール定義スキーマ

**拡張思考でよくあるミス:**

```
# 間違い: budget_tokensはmax_tokens未満である必要がある
thinking: budget_tokens=10000, max_tokens=1000  → エラー！

# 正しい
thinking: budget_tokens=10000, max_tokens=16000
```

---

### 429 Rate Limited

**原因:**

- 1分あたりのリクエスト数（RPM）を超過
- 1分あたりのトークン数（TPM）を超過
- 1日あたりのトークン数（TPD）を超過

**確認すべきヘッダー:**

- `retry-after`: リトライまでの待機秒数
- `x-ratelimit-limit-*`: 制限値
- `x-ratelimit-remaining-*`: 残りのクォータ

**対処方法:** Anthropic SDKは429および5xxエラーを指数バックオフで自動的にリトライします（デフォルト: `max_retries=2`）。カスタムリトライ動作については、言語固有のエラー処理の例を参照してください。

---

### 500 Internal Server Error

**原因:**

- 一時的なAnthropicサービスの問題
- API処理のバグ

**対処方法:** 指数バックオフでリトライしてください。継続する場合は、[status.anthropic.com](https://status.anthropic.com)を確認してください。

---

### 529 Overloaded

**原因:**

- API需要が高い
- サービス容量に達した

**対処方法:** 指数バックオフでリトライしてください。別のモデルの使用を検討（Haikuは通常負荷が低い）、リクエストを時間分散、またはリクエストキューイングの実装を検討してください。

---

## よくあるミスと対処方法

| ミス                            | エラー           | 対処方法                                                    |
| ------------------------------- | ---------------- | ----------------------------------------------------------- |
| `budget_tokens` >= `max_tokens` | 400              | `budget_tokens` < `max_tokens`を確保する                    |
| モデルIDの入力ミス              | 404              | `{{OPUS_ID}}`のような有効なモデルIDを使用する               |
| 最初のメッセージが`assistant`   | 400              | 最初のメッセージは`user`である必要がある                    |
| 同じロールのメッセージが連続    | 400              | `user`と`assistant`を交互にする                             |
| コード内にAPIキー               | 401（漏洩キー）  | 環境変数を使用する                                          |
| カスタムリトライが必要          | 429/5xx          | SDKは自動的にリトライする。`max_retries`でカスタマイズ      |

## SDKの型付き例外

エラーメッセージを文字列マッチングでチェックする代わりに、**常にSDKの型付き例外クラスを使用**してください。各HTTPエラーコードは特定の例外クラスにマップされます：

| HTTPコード | TypeScriptクラス                  | Pythonクラス                      |
| --------- | --------------------------------- | --------------------------------- |
| 400       | `Anthropic.BadRequestError`       | `anthropic.BadRequestError`       |
| 401       | `Anthropic.AuthenticationError`   | `anthropic.AuthenticationError`   |
| 403       | `Anthropic.PermissionDeniedError` | `anthropic.PermissionDeniedError` |
| 404       | `Anthropic.NotFoundError`         | `anthropic.NotFoundError`         |
| 429       | `Anthropic.RateLimitError`        | `anthropic.RateLimitError`        |
| 500+      | `Anthropic.InternalServerError`   | `anthropic.InternalServerError`   |
| 任意      | `Anthropic.APIError`              | `anthropic.APIError`              |

```typescript
// 正しい: 型付き例外を使用
try {
  const response = await client.messages.create({...});
} catch (error) {
  if (error instanceof Anthropic.RateLimitError) {
    // レート制限を処理
  } else if (error instanceof Anthropic.APIError) {
    console.error(`APIエラー ${error.status}:`, error.message);
  }
}

// 間違い: 文字列マッチングでエラーメッセージをチェックしない
try {
  const response = await client.messages.create({...});
} catch (error) {
  const msg = error instanceof Error ? error.message : String(error);
  if (msg.includes("429") || msg.includes("rate_limit")) { ... }
}
```

すべての例外クラスは`Anthropic.APIError`を継承しており、`status`プロパティを持っています。`instanceof`チェックは最も具体的なものから最も一般的なものへ（例：`RateLimitError`を`APIError`より先にチェック）行ってください。
