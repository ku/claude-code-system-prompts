<!--
name: 'データ: Claude APIリファレンス — Python'
description: インストール、クライアント初期化、基本リクエスト、思考、マルチターン会話を含むPython SDKリファレンス
ccVersion: 2.1.63
-->
# Claude API — Python

## インストール

\`\`\`bash
pip install anthropic
\`\`\`

## クライアント初期化

\`\`\`python
import anthropic

# デフォルト（ANTHROPIC_API_KEY環境変数を使用）
client = anthropic.Anthropic()

# 明示的なAPIキー
client = anthropic.Anthropic(api_key="your-api-key")

# 非同期クライアント
async_client = anthropic.AsyncAnthropic()
\`\`\`

---

## 基本的なメッセージリクエスト

\`\`\`python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "What is the capital of France?"}
    ]
)
print(response.content[0].text)
\`\`\`

---

## システムプロンプト

\`\`\`python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    system="You are a helpful coding assistant. Always provide examples in Python.",
    messages=[{"role": "user", "content": "How do I read a JSON file?"}]
)
\`\`\`

---

## ビジョン（画像）

### Base64

\`\`\`python
import base64

with open("image.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": image_data
                }
            },
            {"type": "text", "text": "What's in this image?"}
        ]
    }]
)
\`\`\`

### URL

\`\`\`python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "url",
                    "url": "https://example.com/image.png"
                }
            },
            {"type": "text", "text": "Describe this image"}
        ]
    }]
)
\`\`\`

---

## プロンプトキャッシング

大きなコンテキストをキャッシュしてコストを削減（最大90%の節約）。

### 自動キャッシング（推奨）

トップレベルの`cache_control`を使用して、リクエスト内の最後のキャッシュ可能なブロックを自動的にキャッシュ — 個々のコンテンツブロックにアノテーションを付ける必要はありません：

\`\`\`python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},  # 最後のキャッシュ可能なブロックを自動キャッシュ
    system="You are an expert on this large document...",
    messages=[{"role": "user", "content": "Summarize the key points"}]
)
\`\`\`

### 手動キャッシュ制御

きめ細かい制御のために、特定のコンテンツブロックに`cache_control`を追加：

\`\`\`python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    system=[{
        "type": "text",
        "text": "You are an expert on this large document...",
        "cache_control": {"type": "ephemeral"}  # デフォルトTTLは5分
    }],
    messages=[{"role": "user", "content": "Summarize the key points"}]
)

# 明示的なTTL（有効期限）付き
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    system=[{
        "type": "text",
        "text": "You are an expert on this large document...",
        "cache_control": {"type": "ephemeral", "ttl": "1h"}  # 1時間のTTL
    }],
    messages=[{"role": "user", "content": "Summarize the key points"}]
)
\`\`\`

---

## 拡張思考

> **Opus 4.6とSonnet 4.6:** アダプティブ思考を使用。`budget_tokens`はOpus 4.6とSonnet 4.6の両方で非推奨。
> **古いモデル:** `thinking: {type: "enabled", budget_tokens: N}`を使用（`max_tokens`より小さく、最小1024）。

\`\`\`python
# Opus 4.6: アダプティブ思考（推奨）
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # low | medium | high | max
    messages=[{"role": "user", "content": "Solve this step by step..."}]
)

# 思考とレスポンスにアクセス
for block in response.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking}")
    elif block.type == "text":
        print(f"Response: {block.text}")
\`\`\`

---

## エラー処理

\`\`\`python
import anthropic

try:
    response = client.messages.create(...)
except anthropic.BadRequestError as e:
    print(f"Bad request: {e.message}")
except anthropic.AuthenticationError:
    print("Invalid API key")
except anthropic.PermissionDeniedError:
    print("API key lacks required permissions")
except anthropic.NotFoundError:
    print("Invalid model or endpoint")
except anthropic.RateLimitError as e:
    retry_after = int(e.response.headers.get("retry-after", "60"))
    print(f"Rate limited. Retry after {retry_after}s.")
except anthropic.APIStatusError as e:
    if e.status_code >= 500:
        print(f"Server error ({e.status_code}). Retry later.")
    else:
        print(f"API error: {e.message}")
except anthropic.APIConnectionError:
    print("Network error. Check internet connection.")
\`\`\`

---

## マルチターン会話

APIはステートレス — 毎回完全な会話履歴を送信。

\`\`\`python
class ConversationManager:
    """Claude APIでマルチターン会話を管理。"""

    def __init__(self, client: anthropic.Anthropic, model: str, system: str = None):
        self.client = client
        self.model = model
        self.system = system
        self.messages = []

    def send(self, user_message: str, **kwargs) -> str:
        """メッセージを送信してレスポンスを取得。"""
        self.messages.append({"role": "user", "content": user_message})

        response = self.client.messages.create(
            model=self.model,
            max_tokens=kwargs.get("max_tokens", 1024),
            system=self.system,
            messages=self.messages,
            **kwargs
        )

        assistant_message = response.content[0].text
        self.messages.append({"role": "assistant", "content": assistant_message})

        return assistant_message

# 使用方法
conversation = ConversationManager(
    client=anthropic.Anthropic(),
    model="{{OPUS_ID}}",
    system="You are a helpful assistant."
)

response1 = conversation.send("My name is Alice.")
response2 = conversation.send("What's my name?")  # Claudeは「Alice」を覚えている
\`\`\`

**ルール：**

- メッセージは`user`と`assistant`の間で交互に送信する必要がある
- 最初のメッセージは`user`でなければならない

---

### コンパクション（長い会話）

> **ベータ、Opus 4.6のみ。** 会話が200Kコンテキストウィンドウに近づくと、コンパクションはサーバー側で以前のコンテキストを自動的に要約します。APIは`compaction`ブロックを返します。後続のリクエストで渡す必要があります — テキストだけでなく`response.content`を追加してください。

\`\`\`python
import anthropic

client = anthropic.Anthropic()
messages = []

def chat(user_message: str) -> str:
    messages.append({"role": "user", "content": user_message})

    response = client.beta.messages.create(
        betas=["compact-2026-01-12"],
        model="{{OPUS_ID}}",
        max_tokens=4096,
        messages=messages,
        context_management={
            "edits": [{"type": "compact_20260112"}]
        }
    )

    # 完全なコンテンツを追加 — コンパクションブロックを保持する必要がある
    messages.append({"role": "assistant", "content": response.content})

    return next(block.text for block in response.content if block.type == "text")

# コンテキストが大きくなると自動的にコンパクションがトリガーされる
print(chat("Help me build a Python web scraper"))
print(chat("Add support for JavaScript-rendered pages"))
print(chat("Now add rate limiting and error handling"))
\`\`\`

---

## 停止理由

レスポンスの`stop_reason`フィールドは、モデルが生成を停止した理由を示します：

| 値 | 意味 |
|-------|---------|
| `end_turn` | Claudeは自然にレスポンスを終了した |
| `max_tokens` | `max_tokens`の制限に達した — 増やすかストリーミングを使用 |
| `stop_sequence` | カスタム停止シーケンスに達した |
| `tool_use` | Claudeはツールを呼び出したい — 実行して続行 |
| `pause_turn` | モデルは一時停止し、再開可能（エージェントフロー） |
| `refusal` | Claudeは安全上の理由で拒否 — 出力がスキーマと一致しない可能性 |

---

## コスト最適化戦略

### 1. 繰り返しコンテキストにプロンプトキャッシングを使用

\`\`\`python
# 自動キャッシング（最もシンプル — 最後のキャッシュ可能なブロックをキャッシュ）
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},
    system=large_document_text,  # 例：50KBのコンテキスト
    messages=[{"role": "user", "content": "Summarize the key points"}]
)

# 最初のリクエスト：フルコスト
# 後続のリクエスト：キャッシュ部分は約90%安い
\`\`\`

### 2. 適切なモデルを選択

\`\`\`python
# ほとんどのタスクにはOpusをデフォルトで使用
response = client.messages.create(
    model="{{OPUS_ID}}",  # 1Mトークンあたり$5.00/$25.00
    max_tokens=1024,
    messages=[{"role": "user", "content": "Explain quantum computing"}]
)

# 大量の本番ワークロードにはSonnetを使用
standard_response = client.messages.create(
    model="{{SONNET_ID}}",  # 1Mトークンあたり$3.00/$15.00
    max_tokens=1024,
    messages=[{"role": "user", "content": "Summarize this document"}]
)

# シンプルで速度が重要なタスクにのみHaikuを使用
simple_response = client.messages.create(
    model="{{HAIKU_ID}}",  # 1Mトークンあたり$1.00/$5.00
    max_tokens=256,
    messages=[{"role": "user", "content": "Classify this as positive or negative"}]
)
\`\`\`

### 3. リクエスト前にトークンカウントを使用

\`\`\`python
count_response = client.messages.count_tokens(
    model="{{OPUS_ID}}",
    messages=messages,
    system=system
)

estimated_input_cost = count_response.input_tokens * 0.000005  # 1Mトークンあたり$5
print(f"Estimated input cost: \${estimated_input_cost:.4f}")
\`\`\`

---

## 指数バックオフでリトライ

> **注意:** Anthropic SDKはレート制限（429）とサーバーエラー（5xx）を指数バックオフで自動的にリトライします。`max_retries`（デフォルト：2）で設定できます。SDKが提供する動作を超えて必要な場合にのみカスタムリトライロジックを実装してください。

\`\`\`python
import time
import random
import anthropic

def call_with_retry(
    client: anthropic.Anthropic,
    max_retries: int = 5,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    **kwargs
):
    """指数バックオフリトライでAPIを呼び出す。"""
    last_exception = None

    for attempt in range(max_retries):
        try:
            return client.messages.create(**kwargs)
        except anthropic.RateLimitError as e:
            last_exception = e
        except anthropic.APIStatusError as e:
            if e.status_code >= 500:
                last_exception = e
            else:
                raise  # クライアントエラー（429以外の4xx）はリトライすべきではない

        delay = min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
        print(f"Retry {attempt + 1}/{max_retries} after {delay:.1f}s")
        time.sleep(delay)

    raise last_exception
\`\`\`
