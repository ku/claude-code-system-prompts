<!--
name: 'データ: Claude APIリファレンス — TypeScript'
description: インストール、クライアント初期化、基本リクエスト、思考、マルチターン会話を含むTypeScript SDKリファレンス
ccVersion: 2.1.63
-->
# Claude API — TypeScript

## インストール

\`\`\`bash
npm install @anthropic-ai/sdk
\`\`\`

## クライアント初期化

\`\`\`typescript
import Anthropic from "@anthropic-ai/sdk";

// デフォルト（ANTHROPIC_API_KEY環境変数を使用）
const client = new Anthropic();

// 明示的なAPIキー
const client = new Anthropic({ apiKey: "your-api-key" });
\`\`\`

---

## 基本的なメッセージリクエスト

\`\`\`typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [{ role: "user", content: "What is the capital of France?" }],
});
console.log(response.content[0].text);
\`\`\`

---

## システムプロンプト

\`\`\`typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  system:
    "You are a helpful coding assistant. Always provide examples in Python.",
  messages: [{ role: "user", content: "How do I read a JSON file?" }],
});
\`\`\`

---

## ビジョン（画像）

### URL

\`\`\`typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "url", url: "https://example.com/image.png" },
        },
        { type: "text", text: "Describe this image" },
      ],
    },
  ],
});
\`\`\`

### Base64

\`\`\`typescript
import fs from "fs";

const imageData = fs.readFileSync("image.png").toString("base64");

const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "base64", media_type: "image/png", data: imageData },
        },
        { type: "text", text: "What's in this image?" },
      ],
    },
  ],
});
\`\`\`

---

## プロンプトキャッシング

### 自動キャッシング（推奨）

トップレベルの`cache_control`を使用して、リクエスト内の最後のキャッシュ可能なブロックを自動的にキャッシュ：

\`\`\`typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  cache_control: { type: "ephemeral" }, // 最後のキャッシュ可能なブロックを自動キャッシュ
  system: "You are an expert on this large document...",
  messages: [{ role: "user", content: "Summarize the key points" }],
});
\`\`\`

### 手動キャッシュ制御

きめ細かい制御のために、特定のコンテンツブロックに`cache_control`を追加：

\`\`\`typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  system: [
    {
      type: "text",
      text: "You are an expert on this large document...",
      cache_control: { type: "ephemeral" }, // デフォルトTTLは5分
    },
  ],
  messages: [{ role: "user", content: "Summarize the key points" }],
});

// 明示的なTTL（有効期限）付き
const response2 = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  system: [
    {
      type: "text",
      text: "You are an expert on this large document...",
      cache_control: { type: "ephemeral", ttl: "1h" }, // 1時間のTTL
    },
  ],
  messages: [{ role: "user", content: "Summarize the key points" }],
});
\`\`\`

---

## 拡張思考

> **Opus 4.6とSonnet 4.6:** アダプティブ思考を使用。`budget_tokens`はOpus 4.6とSonnet 4.6の両方で非推奨。
> **古いモデル:** `thinking: {type: "enabled", budget_tokens: N}`を使用（`max_tokens`より小さく、最小1024）。

\`\`\`typescript
// Opus 4.6: アダプティブ思考（推奨）
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  thinking: { type: "adaptive" },
  output_config: { effort: "high" }, // low | medium | high | max
  messages: [
    { role: "user", content: "Solve this math problem step by step..." },
  ],
});

for (const block of response.content) {
  if (block.type === "thinking") {
    console.log("Thinking:", block.thinking);
  } else if (block.type === "text") {
    console.log("Response:", block.text);
  }
}
\`\`\`

---

## エラー処理

SDKの型付き例外クラスを使用 — 文字列マッチングでエラーメッセージをチェックしないこと：

\`\`\`typescript
import Anthropic from "@anthropic-ai/sdk";

try {
  const response = await client.messages.create({...});
} catch (error) {
  if (error instanceof Anthropic.BadRequestError) {
    console.error("Bad request:", error.message);
  } else if (error instanceof Anthropic.AuthenticationError) {
    console.error("Invalid API key");
  } else if (error instanceof Anthropic.RateLimitError) {
    console.error("Rate limited - retry later");
  } else if (error instanceof Anthropic.APIError) {
    console.error(\`API error \${error.status}:\`, error.message);
  }
}
\`\`\`

すべてのクラスは型付き`status`フィールドを持つ`Anthropic.APIError`を拡張します。最も具体的なものから最も一般的なものへチェックしてください。完全なエラーコードリファレンスについては[shared/error-codes.md](../../shared/error-codes.md)を参照してください。

---

## マルチターン会話

APIはステートレス — 毎回完全な会話履歴を送信。messages配列の型付けには`Anthropic.MessageParam[]`を使用：

\`\`\`typescript
const messages: Anthropic.MessageParam[] = [
  { role: "user", content: "My name is Alice." },
  { role: "assistant", content: "Hello Alice! Nice to meet you." },
  { role: "user", content: "What's my name?" },
];

const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: messages,
});
\`\`\`

**ルール：**

- メッセージは`user`と`assistant`の間で交互に送信する必要がある
- 最初のメッセージは`user`でなければならない
- すべてのAPIデータ構造にSDK型（`Anthropic.MessageParam`、`Anthropic.Message`、`Anthropic.Tool`など）を使用 — 同等のインターフェースを再定義しない

---

### コンパクション（長い会話）

> **ベータ、Opus 4.6のみ。** 会話が200Kコンテキストウィンドウに近づくと、コンパクションはサーバー側で以前のコンテキストを自動的に要約します。APIは`compaction`ブロックを返します。後続のリクエストで渡す必要があります — テキストだけでなく`response.content`を追加してください。

\`\`\`typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const messages: Anthropic.Beta.BetaMessageParam[] = [];

async function chat(userMessage: string): Promise<string> {
  messages.push({ role: "user", content: userMessage });

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "{{OPUS_ID}}",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }],
    },
  });

  // 完全なコンテンツを追加 — コンパクションブロックを保持する必要がある
  messages.push({ role: "assistant", content: response.content });

  const textBlock = response.content.find((block) => block.type === "text");
  return textBlock?.text ?? "";
}

// コンテキストが大きくなると自動的にコンパクションがトリガーされる
console.log(await chat("Help me build a Python web scraper"));
console.log(await chat("Add support for JavaScript-rendered pages"));
console.log(await chat("Now add rate limiting and error handling"));
\`\`\`

---

## 停止理由

レスポンスの`stop_reason`フィールドは、モデルが生成を停止した理由を示します：

| 値           | 意味                                                         |
| --------------- | --------------------------------------------------------------- |
| `end_turn`      | Claudeは自然にレスポンスを終了した                          |
| `max_tokens`    | `max_tokens`の制限に達した — 増やすかストリーミングを使用       |
| `stop_sequence` | カスタム停止シーケンスに達した                                      |
| `tool_use`      | Claudeはツールを呼び出したい — 実行して続行           |
| `pause_turn`    | モデルは一時停止し、再開可能（エージェントフロー）                 |
| `refusal`       | Claudeは安全上の理由で拒否 — 出力がスキーマと一致しない可能性 |

---

## コスト最適化戦略

### 1. 繰り返しコンテキストにプロンプトキャッシングを使用

\`\`\`typescript
// 自動キャッシング（最もシンプル — 最後のキャッシュ可能なブロックをキャッシュ）
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  cache_control: { type: "ephemeral" },
  system: largeDocumentText, // 例：50KBのコンテキスト
  messages: [{ role: "user", content: "Summarize the key points" }],
});

// 最初のリクエスト：フルコスト
// 後続のリクエスト：キャッシュ部分は約90%安い
\`\`\`

### 2. リクエスト前にトークンカウントを使用

\`\`\`typescript
const countResponse = await client.messages.countTokens({
  model: "{{OPUS_ID}}",
  messages: messages,
  system: system,
});

const estimatedInputCost = countResponse.input_tokens * 0.000005; // 1Mトークンあたり$5
console.log(\`Estimated input cost: $\${estimatedInputCost.toFixed(4)}\`);
\`\`\`
