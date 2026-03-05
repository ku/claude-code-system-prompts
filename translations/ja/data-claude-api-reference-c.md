<!--
name: 'データ: Claude APIリファレンス — C#'
description: インストール、クライアント初期化、基本リクエスト、ストリーミング、ツール使用を含むC# SDKリファレンス
ccVersion: 2.1.51
-->
# Claude API — C#

> **注意:** C# SDKはAnthropic公式のC# SDKです。ツール使用はMessages API経由でサポートされています。クラスアノテーションベースのツールランナーは利用できません。JSONスキーマで生のツール定義を使用してください。SDKはMicrosoft.Extensions.AI IChatClient統合と関数呼び出しもサポートしています。

## インストール

\`\`\`bash
dotnet add package Anthropic
\`\`\`

## クライアント初期化

\`\`\`csharp
using Anthropic;

// デフォルト（ANTHROPIC_API_KEY環境変数を使用）
AnthropicClient client = new();

// 明示的なAPIキー（環境変数を使用 — キーをハードコードしない）
AnthropicClient client = new() {
    ApiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY")
};
\`\`\`

---

## 基本的なメッセージリクエスト

\`\`\`csharp
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 1024,
    Messages = [new() { Role = Role.User, Content = "What is the capital of France?" }]
};
var message = await client.Messages.Create(parameters);
Console.WriteLine(message);
\`\`\`

---

## ストリーミング

\`\`\`csharp
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 1024,
    Messages = [new() { Role = Role.User, Content = "Write a haiku" }]
};

await foreach (RawMessageStreamEvent streamEvent in client.Messages.CreateStreaming(parameters))
{
    if (streamEvent.TryPickContentBlockDelta(out var delta) &&
        delta.Delta.TryPickText(out var text))
    {
        Console.Write(text.Text);
    }
}
\`\`\`

---

## ツール使用（手動ループ）

C# SDKはJSONスキーマ経由の生のツール定義をサポートしています。ツール定義形式とエージェントループパターンについては、[共有ツール使用コンセプト](../shared/tool-use-concepts.md)を参照してください。
