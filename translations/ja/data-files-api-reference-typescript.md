<!--
name: 'データ: Files APIリファレンス — TypeScript'
description: ファイルアップロード、一覧表示、削除、メッセージでの使用を含むTypeScript Files APIリファレンス
ccVersion: 2.1.63
-->
# Files API — TypeScript

Files APIは、Messages APIリクエストで使用するファイルをアップロードします。コンテンツブロックで`file_id`を参照することで、複数のAPI呼び出しにわたる再アップロードを回避できます。

**ベータ版:** API呼び出しで`betas: ["files-api-2025-04-14"]`を渡してください（SDKは必要なヘッダーを自動的に設定します）。

## 重要事項

- 最大ファイルサイズ: 500 MB
- 総ストレージ: 組織あたり100 GB
- ファイルは削除されるまで保持されます
- ファイル操作（アップロード、一覧表示、削除）は無料です。メッセージで使用されるコンテンツは入力トークンとして課金されます
- Amazon BedrockおよびGoogle Vertex AIでは利用できません

---

## ファイルのアップロード

```typescript
import Anthropic, { toFile } from "@anthropic-ai/sdk";
import fs from "fs";

const client = new Anthropic();

const uploaded = await client.beta.files.upload({
  file: await toFile(fs.createReadStream("report.pdf"), undefined, {
    type: "application/pdf",
  }),
  betas: ["files-api-2025-04-14"],
});

console.log(`ファイルID: ${uploaded.id}`);
console.log(`サイズ: ${uploaded.size_bytes} バイト`);
```

---

## メッセージでのファイル使用

### PDF / テキストドキュメント

```typescript
const response = await client.beta.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "このレポートの重要な発見を要約してください。" },
        {
          type: "document",
          source: { type: "file", file_id: uploaded.id },
          title: "Q4レポート",
          citations: { enabled: true },
        },
      ],
    },
  ],
  betas: ["files-api-2025-04-14"],
});

console.log(response.content[0].text);
```

---

## ファイルの管理

### ファイル一覧の取得

```typescript
const files = await client.beta.files.list({
  betas: ["files-api-2025-04-14"],
});
for (const f of files.data) {
  console.log(`${f.id}: ${f.filename} (${f.size_bytes} バイト)`);
}
```

### ファイルの削除

```typescript
await client.beta.files.delete("file_011CNha8iCJcU1wXNR6q4V8w", {
  betas: ["files-api-2025-04-14"],
});
```

### ファイルのダウンロード

```typescript
const response = await client.beta.files.download(
  "file_011CNha8iCJcU1wXNR6q4V8w",
  { betas: ["files-api-2025-04-14"] },
);
const content = Buffer.from(await response.arrayBuffer());
await fs.promises.writeFile("output.txt", content);
```
