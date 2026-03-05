<!--
name: 'データ: ストリーミングリファレンス — TypeScript'
description: 基本的なストリーミングと異なるコンテンツタイプの処理を含むTypeScriptストリーミングリファレンス
ccVersion: 2.1.63
-->
# ストリーミング — TypeScript

## クイックスタート

```typescript
const stream = client.messages.stream({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [{ role: "user", content: "物語を書いてください" }],
});

for await (const event of stream) {
  if (
    event.type === "content_block_delta" &&
    event.delta.type === "text_delta"
  ) {
    process.stdout.write(event.delta.text);
  }
}
```

---

## 異なるコンテンツタイプの処理

> **Opus 4.6:** `thinking: {type: "adaptive"}`を使用してください。古いモデルでは、代わりに`thinking: {type: "enabled", budget_tokens: N}`を使用してください。

```typescript
const stream = client.messages.stream({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  thinking: { type: "adaptive" },
  messages: [{ role: "user", content: "この問題を分析してください" }],
});

for await (const event of stream) {
  switch (event.type) {
    case "content_block_start":
      switch (event.content_block.type) {
        case "thinking":
          console.log("\n[思考中...]");
          break;
        case "text":
          console.log("\n[応答:]");
          break;
      }
      break;
    case "content_block_delta":
      switch (event.delta.type) {
        case "thinking_delta":
          process.stdout.write(event.delta.thinking);
          break;
        case "text_delta":
          process.stdout.write(event.delta.text);
          break;
      }
      break;
  }
}
```

---

## ツール使用とストリーミング（ツールランナー）

`stream: true`でツールランナーを使用します。外側のループはツールランナーの反復（メッセージ）を処理し、内側のループはストリームイベントを処理します：

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";

const client = new Anthropic();

const getWeather = betaZodTool({
  name: "get_weather",
  description: "場所の現在の天気を取得",
  inputSchema: z.object({
    location: z.string().describe("市と州、例: San Francisco, CA"),
  }),
  run: async ({ location }) => `${location}は72°Fで晴れです`,
});

const runner = client.beta.messages.toolRunner({
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  tools: [getWeather],
  messages: [
    { role: "user", content: "パリとロンドンの天気は？" },
  ],
  stream: true,
});

// 外側のループ: 各ツールランナーの反復
for await (const messageStream of runner) {
  // 内側のループ: この反復のストリームイベント
  for await (const event of messageStream) {
    switch (event.type) {
      case "content_block_delta":
        switch (event.delta.type) {
          case "text_delta":
            process.stdout.write(event.delta.text);
            break;
          case "input_json_delta":
            // ツール入力のストリーミング
            break;
        }
        break;
    }
  }
}
```

---

## 最終メッセージの取得

```typescript
const stream = client.messages.stream({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [{ role: "user", content: "こんにちは" }],
});

for await (const event of stream) {
  // イベントを処理...
}

const finalMessage = await stream.finalMessage();
console.log(`使用トークン: ${finalMessage.usage.output_tokens}`);
```

---

## ストリームイベントタイプ

| イベントタイプ        | 説明                        | 発生タイミング                    |
| --------------------- | --------------------------- | --------------------------------- |
| `message_start`       | メッセージメタデータを含む  | 最初に1回                         |
| `content_block_start` | 新しいコンテンツブロック開始 | text/tool_useブロックの開始時     |
| `content_block_delta` | 増分コンテンツ更新          | 各トークン/チャンクごと           |
| `content_block_stop`  | コンテンツブロック完了      | ブロック終了時                    |
| `message_delta`       | メッセージレベルの更新      | `stop_reason`、使用量を含む       |
| `message_stop`        | メッセージ完了              | 最後に1回                         |

## ベストプラクティス

1. **常に出力をフラッシュ** — `process.stdout.write()`を使用して即座に表示
2. **部分的なレスポンスを処理** — ストリームが中断された場合、コンテンツが不完全な可能性がある
3. **トークン使用量を追跡** — `message_delta`イベントに使用量情報が含まれる
4. **`finalMessage()`を使用** — ストリーミング時でも完全な`Anthropic.Message`オブジェクトを取得。`.on()`イベントを`new Promise()`でラップしないでください — `finalMessage()`が内部ですべての完了/エラー/中断状態を処理します
5. **Web UIではバッファリング** — 過度なDOM更新を避けるため、レンダリング前に数トークンをバッファリングすることを検討
6. **デルタには`stream.on("text", ...)`を使用** — `text`イベントはデルタ文字列のみを提供し、手動で`content_block_delta`イベントをフィルタリングするより簡単
7. **ストリーミング付きエージェンティックループ** — ツール使用ループと`stream()` + `finalMessage()`を組み合わせる方法については、tool-use.mdの[ストリーミング手動ループ](./tool-use.md#streaming-manual-loop)セクションを参照

## 生のSSE形式

生のHTTP（SDKではなく）を使用する場合、ストリームはServer-Sent Eventsを返します：

```
event: message_start
data: {"type":"message_start","message":{"id":"msg_...","type":"message",...}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"こんにちは"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn"},"usage":{"output_tokens":12}}

event: message_stop
data: {"type":"message_stop"}
```
