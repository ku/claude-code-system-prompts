<!--
name: 'データ: Claude APIリファレンス — Ruby'
description: インストール、クライアント初期化、基本リクエスト、ストリーミング、ベータツールランナーを含むRuby SDKリファレンス
ccVersion: 2.1.63
-->
# Claude API — Ruby

> **注意:** Ruby SDKはClaude APIをサポートしています。ツールランナーは`client.beta.messages.tool_runner()`経由でベータ版で利用可能です。Agent SDKはまだRubyでは利用できません。

## インストール

\`\`\`bash
gem install anthropic
\`\`\`

## クライアント初期化

\`\`\`ruby
require "anthropic"

# デフォルト（ANTHROPIC_API_KEY環境変数を使用）
client = Anthropic::Client.new

# 明示的なAPIキー
client = Anthropic::Client.new(api_key: "your-api-key")
\`\`\`

---

## 基本的なメッセージリクエスト

\`\`\`ruby
message = client.messages.create(
  model: :"{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "What is the capital of France?" }
  ]
)
puts message.content.first.text
\`\`\`

---

## ストリーミング

\`\`\`ruby
stream = client.messages.stream(
  model: :"{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a haiku" }]
)

stream.text.each { |text| print(text) }
\`\`\`

---

## ツール使用

Ruby SDKは生のJSONスキーマ定義によるツール使用をサポートし、自動ツール実行のためのベータツールランナーも提供しています。

### ツールランナー（ベータ）

\`\`\`ruby
class GetWeatherInput < Anthropic::BaseModel
  required :location, String, doc: "City and state, e.g. San Francisco, CA"
end

class GetWeather < Anthropic::BaseTool
  doc "Get the current weather for a location"

  input_schema GetWeatherInput

  def call(input)
    "The weather in #{input.location} is sunny and 72°F."
  end
end

client.beta.messages.tool_runner(
  model: :"{{OPUS_ID}}",
  max_tokens: 1024,
  tools: [GetWeather.new],
  messages: [{ role: "user", content: "What's the weather in San Francisco?" }]
).each_message do |message|
  puts message.content
end
\`\`\`

### 手動ループ

ツール定義形式とエージェントループパターンについては、[共有ツール使用コンセプト](../shared/tool-use-concepts.md)を参照してください。
