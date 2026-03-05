<!--
name: 'データ: ツール使用リファレンス — TypeScript'
description: ツールランナー、手動エージェンティックループ、コード実行、構造化出力を含むTypeScriptツール使用リファレンス
ccVersion: 2.1.63
-->
# ツール使用 — TypeScript

概念的な概要（ツール定義、ツール選択、ヒント）については、[shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)を参照してください。

## ツールランナー（推奨）

**ベータ版:** ツールランナーはTypeScript SDKでベータ版です。

`betaZodTool`をZodスキーマと`run`関数でツールを定義し、`client.beta.messages.toolRunner()`に渡します：

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
    unit: z.enum(["celsius", "fahrenheit"]).optional(),
  }),
  run: async (input) => {
    // ここに実装
    return `${input.location}は72°Fで晴れです`;
  },
});

// ツールランナーはエージェンティックループを処理し、最終メッセージを返す
const finalMessage = await client.beta.messages.toolRunner({
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  tools: [getWeather],
  messages: [{ role: "user", content: "パリの天気は？" }],
});

console.log(finalMessage.content);
```

**ツールランナーの主な利点:**

- 手動ループなし — SDKがツールの呼び出しと結果のフィードバックを処理
- Zodスキーマによる型安全なツール入力
- Zod定義からツールスキーマを自動生成
- Claudeがツール呼び出しをしなくなると自動的に反復停止

---

## 手動エージェンティックループ

ループを細かく制御する必要がある場合（カスタムログ、条件付きツール実行、個々の反復のストリーミング、ヒューマンインザループ承認）に使用：

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const tools: Anthropic.Tool[] = [...]; // ツール定義
let messages: Anthropic.MessageParam[] = [{ role: "user", content: userInput }];

while (true) {
  const response = await client.messages.create({
    model: "{{OPUS_ID}}",
    max_tokens: 4096,
    tools: tools,
    messages: messages,
  });

  if (response.stop_reason === "end_turn") break;

  // サーバーサイドツールが反復制限に達した場合、再送信して続行
  if (response.stop_reason === "pause_turn") {
    messages = [
      { role: "user", content: userInput },
      { role: "assistant", content: response.content },
    ];
    continue;
  }

  const toolUseBlocks = response.content.filter(
    (b): b is Anthropic.ToolUseBlock => b.type === "tool_use",
  );

  messages.push({ role: "assistant", content: response.content });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const tool of toolUseBlocks) {
    const result = await executeTool(tool.name, tool.input);
    toolResults.push({
      type: "tool_result",
      tool_use_id: tool.id,
      content: result,
    });
  }

  messages.push({ role: "user", content: toolResults });
}
```

### ストリーミング手動ループ

手動ループ内でストリーミングが必要な場合は、`.create()`の代わりに`client.messages.stream()` + `finalMessage()`を使用します。テキストデルタは各反復でストリームされ、`finalMessage()`は`stop_reason`を検査してtool-useブロックを抽出できる完全な`Message`を収集します：

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const tools: Anthropic.Tool[] = [...];
let messages: Anthropic.MessageParam[] = [{ role: "user", content: userInput }];

while (true) {
  const stream = client.messages.stream({
    model: "{{OPUS_ID}}",
    max_tokens: 4096,
    tools,
    messages,
  });

  // 各反復でテキストデルタをストリーム
  stream.on("text", (delta) => {
    process.stdout.write(delta);
  });

  // finalMessage()は完全なMessageで解決 —
  // .on("message") / .on("error") / .on("abort")を手動で接続する必要なし
  const message = await stream.finalMessage();

  if (message.stop_reason === "end_turn") break;

  // サーバーサイドツールが反復制限に達した場合、再送信して続行
  if (message.stop_reason === "pause_turn") {
    messages = [
      { role: "user", content: userInput },
      { role: "assistant", content: message.content },
    ];
    continue;
  }

  const toolUseBlocks = message.content.filter(
    (b): b is Anthropic.ToolUseBlock => b.type === "tool_use",
  );

  messages.push({ role: "assistant", content: message.content });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const tool of toolUseBlocks) {
    const result = await executeTool(tool.name, tool.input);
    toolResults.push({
      type: "tool_result",
      tool_use_id: tool.id,
      content: result,
    });
  }

  messages.push({ role: "user", content: toolResults });
}
```

> **重要:** 最終メッセージを収集するために`.on()`イベントを`new Promise()`でラップしないでください — 代わりに`stream.finalMessage()`を使用してください。SDKが内部ですべてのエラー/中断/完了状態を処理します。

> **ループ内のエラー処理:** SDKの型付き例外（例：`Anthropic.RateLimitError`、`Anthropic.APIError`）を使用してください — 例については[エラー処理](./README.md#error-handling)を参照。文字列マッチングでエラーメッセージをチェックしないでください。

> **SDK型:** すべてのAPI関連データ構造に`Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.ToolUseBlock`、`Anthropic.ToolResultBlockParam`、`Anthropic.Message`などを使用してください。同等のインターフェースを再定義しないでください。

---

## ツール結果の処理

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  tools: tools,
  messages: [{ role: "user", content: "パリの天気は？" }],
});

for (const block of response.content) {
  if (block.type === "tool_use") {
    const result = await executeTool(block.name, block.input);

    const followup = await client.messages.create({
      model: "{{OPUS_ID}}",
      max_tokens: 1024,
      tools: tools,
      messages: [
        { role: "user", content: "パリの天気は？" },
        { role: "assistant", content: response.content },
        {
          role: "user",
          content: [
            { type: "tool_result", tool_use_id: block.id, content: result },
          ],
        },
      ],
    });
  }
}
```

---

## ツール選択

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  tools: tools,
  tool_choice: { type: "tool", name: "get_weather" },
  messages: [{ role: "user", content: "パリの天気は？" }],
});
```

---

## コード実行

### 基本的な使用方法

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  messages: [
    {
      role: "user",
      content:
        "[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]の平均と標準偏差を計算してください",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});
```

### 分析用のファイルアップロード

```typescript
import Anthropic, { toFile } from "@anthropic-ai/sdk";
import { createReadStream } from "fs";

const client = new Anthropic();

// 1. ファイルをアップロード
const uploaded = await client.beta.files.upload({
  file: await toFile(createReadStream("sales_data.csv"), undefined, {
    type: "text/csv",
  }),
  betas: ["files-api-2025-04-14"],
});

// 2. コード実行に渡す
// コード実行はGA。Files APIはまだベータ版（RequestOptions経由で渡す）
const response = await client.messages.create(
  {
    model: "{{OPUS_ID}}",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "text",
            text: "この売上データを分析してください。トレンドを表示し、可視化を作成してください。",
          },
          { type: "container_upload", file_id: uploaded.id },
        ],
      },
    ],
    tools: [{ type: "code_execution_20260120", name: "code_execution" }],
  },
  { headers: { "anthropic-beta": "files-api-2025-04-14" } },
);
```

### 生成されたファイルの取得

```typescript
import path from "path";
import fs from "fs";

const OUTPUT_DIR = "./claude_outputs";
await fs.promises.mkdir(OUTPUT_DIR, { recursive: true });

for (const block of response.content) {
  if (block.type === "bash_code_execution_tool_result") {
    const result = block.content;
    if (result.type === "bash_code_execution_result" && result.content) {
      for (const fileRef of result.content) {
        if (fileRef.type === "bash_code_execution_output") {
          const metadata = await client.beta.files.retrieveMetadata(
            fileRef.file_id,
          );
          const response = await client.beta.files.download(fileRef.file_id);
          const fileBytes = Buffer.from(await response.arrayBuffer());
          const safeName = path.basename(metadata.filename);
          if (!safeName || safeName === "." || safeName === "..") {
            console.warn(`無効なファイル名をスキップ: ${metadata.filename}`);
            continue;
          }
          const outputPath = path.join(OUTPUT_DIR, safeName);
          await fs.promises.writeFile(outputPath, fileBytes);
          console.log(`保存: ${outputPath}`);
        }
      }
    }
  }
}
```

### コンテナの再利用

```typescript
// 最初のリクエスト: 環境をセットアップ
const response1 = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  messages: [
    {
      role: "user",
      content: "tabulateをインストールし、サンプルユーザーデータでdata.jsonを作成してください",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});

// コンテナを再利用
const containerId = response1.container.id;

const response2 = await client.messages.create({
  container: containerId,
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  messages: [
    {
      role: "user",
      content: "data.jsonを読み込み、整形されたテーブルとして表示してください",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});
```

---

## メモリツール

### 基本的な使用方法

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 2048,
  messages: [
    {
      role: "user",
      content: "私の好みの言語がTypeScriptであることを覚えておいてください。",
    },
  ],
  tools: [{ type: "memory_20250818", name: "memory" }],
});
```

### SDKメモリヘルパー

`betaMemoryTool`を`MemoryToolHandlers`実装で使用：

```typescript
import {
  betaMemoryTool,
  type MemoryToolHandlers,
} from "@anthropic-ai/sdk/helpers/beta/memory";

const handlers: MemoryToolHandlers = {
  async view(command) { ... },
  async create(command) { ... },
  async str_replace(command) { ... },
  async insert(command) { ... },
  async delete(command) { ... },
  async rename(command) { ... },
};

const memory = betaMemoryTool(handlers);

const runner = client.beta.messages.toolRunner({
  model: "{{OPUS_ID}}",
  max_tokens: 2048,
  tools: [memory],
  messages: [{ role: "user", content: "私の好みを覚えておいてください" }],
});

for await (const message of runner) {
  console.log(message);
}
```

完全な実装例については、WebFetchを使用してください：

- `https://github.com/anthropics/anthropic-sdk-typescript/blob/main/examples/tools-helpers-memory.ts`

---

## 構造化出力

### JSON出力（Zod — 推奨）

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { z } from "zod";
import { zodOutputFormat } from "@anthropic-ai/sdk/helpers/zod";

const ContactInfoSchema = z.object({
  name: z.string(),
  email: z.string(),
  plan: z.string(),
  interests: z.array(z.string()),
  demo_requested: z.boolean(),
});

const client = new Anthropic();

const response = await client.messages.parse({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content:
        "抽出: Jane Doe (jane@co.com)はEnterprise希望、APIとSDKに興味あり、デモ希望。",
    },
  ],
  output_config: {
    format: zodOutputFormat(ContactInfoSchema),
  },
});

console.log(response.parsed_output.name); // "Jane Doe"
```

### 厳密なツール使用

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: "3月15日に2名で東京へのフライトを予約してください",
    },
  ],
  tools: [
    {
      name: "book_flight",
      description: "目的地へのフライトを予約",
      strict: true,
      input_schema: {
        type: "object",
        properties: {
          destination: { type: "string" },
          date: { type: "string", format: "date" },
          passengers: {
            type: "integer",
            enum: [1, 2, 3, 4, 5, 6, 7, 8],
          },
        },
        required: ["destination", "date", "passengers"],
        additionalProperties: false,
      },
    },
  ],
});
```
