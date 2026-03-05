<!--
name: 'データ: Claude APIリファレンス — PHP'
description: PHP SDKリファレンス
ccVersion: 2.1.63
-->
# Claude API — PHP

> **注意:** PHP SDKはAnthropic公式のPHP SDKです。ツールランナーとAgent SDKは利用できません。Bedrock、Vertex AI、Foundryクライアントはサポートされています。

## インストール

\`\`\`bash
composer require "anthropic-ai/sdk"
\`\`\`

## クライアント初期化

\`\`\`php
use Anthropic\\Client;

// 環境変数からのAPIキーを使用
$client = new Client(apiKey: getenv("ANTHROPIC_API_KEY"));
\`\`\`

### Amazon Bedrock

\`\`\`php
use Anthropic\\BedrockClient;

$client = new BedrockClient(
    region: 'us-east-1',
);
\`\`\`

### Google Vertex AI

\`\`\`php
use Anthropic\\VertexClient;

$client = new VertexClient(
    region: 'us-east5',
    projectId: 'my-project-id',
);
\`\`\`

### Anthropic Foundry

\`\`\`php
use Anthropic\\FoundryClient;

$client = new FoundryClient(
    authToken: getenv("ANTHROPIC_AUTH_TOKEN"),
);
\`\`\`

---

## 基本的なメッセージリクエスト

\`\`\`php
$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 1024,
    messages: [
        ['role' => 'user', 'content' => 'What is the capital of France?'],
    ],
);
echo $message->content[0]->text;
\`\`\`

---

## ストリーミング

\`\`\`php
$stream = $client->messages->createStream(
    model: '{{OPUS_ID}}',
    maxTokens: 1024,
    messages: [
        ['role' => 'user', 'content' => 'Write a haiku'],
    ],
);

foreach ($stream as $event) {
    echo $event;
}
\`\`\`

---

## ツール使用（手動ループ）

PHP SDKはJSONスキーマ経由の生のツール定義をサポートしています。ツール定義形式とエージェントループパターンについては、[共有ツール使用コンセプト](../shared/tool-use-concepts.md)を参照してください。
