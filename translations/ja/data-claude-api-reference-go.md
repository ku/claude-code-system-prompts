<!--
name: 'データ: Claude APIリファレンス — Go'
description: Go SDKリファレンス
ccVersion: 2.1.63
-->
# Claude API — Go

> **注意:** Go SDKはClaude APIと`BetaToolRunner`によるベータツール使用をサポートしています。Agent SDKはまだGoでは利用できません。

## インストール

\`\`\`bash
go get github.com/anthropics/anthropic-sdk-go
\`\`\`

## クライアント初期化

\`\`\`go
import (
    "github.com/anthropics/anthropic-sdk-go"
    "github.com/anthropics/anthropic-sdk-go/option"
)

// デフォルト（ANTHROPIC_API_KEY環境変数を使用）
client := anthropic.NewClient()

// 明示的なAPIキー
client := anthropic.NewClient(
    option.WithAPIKey("your-api-key"),
)
\`\`\`

---

## 基本的なメッセージリクエスト

\`\`\`go
response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    Model:     anthropic.ModelClaudeOpus4_6,
    MaxTokens: 1024,
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("What is the capital of France?")),
    },
})
if err != nil {
    log.Fatal(err)
}
fmt.Println(response.Content[0].Text)
\`\`\`

---

## ストリーミング

\`\`\`go
stream := client.Messages.NewStreaming(context.TODO(), anthropic.MessageNewParams{
    Model:     anthropic.ModelClaudeOpus4_6,
    MaxTokens: 1024,
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("Write a haiku")),
    },
})

for stream.Next() {
    event := stream.Current()
    switch eventVariant := event.AsAny().(type) {
    case anthropic.ContentBlockDeltaEvent:
        switch deltaVariant := eventVariant.Delta.AsAny().(type) {
        case anthropic.TextDelta:
            fmt.Print(deltaVariant.Text)
        }
    }
}
if err := stream.Err(); err != nil {
    log.Fatal(err)
}
\`\`\`

---

## ツール使用

### ツールランナー（ベータ — 推奨）

**ベータ:** Go SDKは`toolrunner`パッケージ経由で自動ツール使用ループのための`BetaToolRunner`を提供します。

\`\`\`go
import (
    "context"
    "fmt"
    "log"

    "github.com/anthropics/anthropic-sdk-go"
    "github.com/anthropics/anthropic-sdk-go/toolrunner"
)

// 自動スキーマ生成のためにjsonschemaタグでツール入力を定義
type GetWeatherInput struct {
    City string \`json:"city" jsonschema:"required,description=The city name"\`
}

// 構造体タグからの自動スキーマ生成でツールを作成
weatherTool, err := toolrunner.NewBetaToolFromJSONSchema(
    "get_weather",
    "Get current weather for a city",
    func(ctx context.Context, input GetWeatherInput) (anthropic.BetaToolResultBlockParamContentUnion, error) {
        return anthropic.BetaToolResultBlockParamContentUnion{
            OfText: &anthropic.BetaTextBlockParam{
                Text: fmt.Sprintf("The weather in %s is sunny, 72°F", input.City),
            },
        }, nil
    },
)
if err != nil {
    log.Fatal(err)
}

// 会話ループを自動的に処理するツールランナーを作成
runner := client.Beta.Messages.NewToolRunner(
    []anthropic.BetaTool{weatherTool},
    anthropic.BetaToolRunnerParams{
        BetaMessageNewParams: anthropic.BetaMessageNewParams{
            Model:     anthropic.ModelClaudeOpus4_6,
            MaxTokens: 1024,
            Messages: []anthropic.BetaMessageParam{
                anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("What's the weather in Paris?")),
            },
        },
        MaxIterations: 5,
    },
)

// Claudeが最終的な応答を生成するまで実行
message, err := runner.RunToCompletion(context.Background())
if err != nil {
    log.Fatal(err)
}
fmt.Println(message.Content[0].Text)
\`\`\`

**Goツールランナーの主な機能：**

- `jsonschema`タグによるGo構造体からの自動スキーマ生成
- シンプルなワンショット使用のための`RunToCompletion()`
- 会話の各メッセージを処理するための`All()`イテレータ
- ステップバイステップの反復のための`NextMessage()`
- `AllStreaming()`を持つ`NewToolRunnerStreaming()`によるストリーミングバリアント

### 手動ループ

きめ細かい制御のためには、JSONスキーマ経由の生のツール定義を使用します。ツール定義形式とエージェントループパターンについては、[共有ツール使用コンセプト](../shared/tool-use-concepts.md)を参照してください。
