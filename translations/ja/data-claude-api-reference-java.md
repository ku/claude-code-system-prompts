<!--
name: 'データ: Claude APIリファレンス — Java'
description: インストール、クライアント初期化、基本リクエスト、ストリーミング、ベータツール使用を含むJava SDKリファレンス
ccVersion: 2.1.63
-->
# Claude API — Java

> **注意:** Java SDKはClaude APIとアノテーション付きクラスによるベータツール使用をサポートしています。Agent SDKはまだJavaでは利用できません。

## インストール

Maven:

\`\`\`xml
<dependency>
    <groupId>com.anthropic</groupId>
    <artifactId>anthropic-java</artifactId>
    <version>2.15.0</version>
</dependency>
\`\`\`

Gradle:

\`\`\`groovy
implementation("com.anthropic:anthropic-java:2.15.0")
\`\`\`

## クライアント初期化

\`\`\`java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

// デフォルト（環境からANTHROPIC_API_KEYを読み取る）
AnthropicClient client = AnthropicOkHttpClient.fromEnv();

// 明示的なAPIキー
AnthropicClient client = AnthropicOkHttpClient.builder()
    .apiKey("your-api-key")
    .build();
\`\`\`

---

## 基本的なメッセージリクエスト

\`\`\`java
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.Model;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(1024L)
    .addUserMessage("What is the capital of France?")
    .build();

Message response = client.messages().create(params);
response.content().stream()
    .flatMap(block -> block.text().stream())
    .forEach(textBlock -> System.out.println(textBlock.text()));
\`\`\`

---

## ストリーミング

\`\`\`java
import com.anthropic.core.http.StreamResponse;
import com.anthropic.models.messages.RawMessageStreamEvent;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(1024L)
    .addUserMessage("Write a haiku")
    .build();

try (StreamResponse<RawMessageStreamEvent> streamResponse = client.messages().createStreaming(params)) {
    streamResponse.stream()
        .flatMap(event -> event.contentBlockDelta().stream())
        .flatMap(deltaEvent -> deltaEvent.delta().text().stream())
        .forEach(textDelta -> System.out.print(textDelta.text()));
}
\`\`\`

---

## ツール使用（ベータ）

Java SDKはアノテーション付きクラスによるベータツール使用をサポートしています。ツールクラスは`BetaToolRunner`による自動実行のために`Supplier<String>`を実装します。

### ツールランナー（自動ループ）

\`\`\`java
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.BetaMessage;
import com.anthropic.helpers.BetaToolRunner;
import com.fasterxml.jackson.annotation.JsonClassDescription;
import com.fasterxml.jackson.annotation.JsonPropertyDescription;
import java.util.function.Supplier;

@JsonClassDescription("Get the weather in a given location")
static class GetWeather implements Supplier<String> {
    @JsonPropertyDescription("The city and state, e.g. San Francisco, CA")
    public String location;

    @Override
    public String get() {
        return "The weather in " + location + " is sunny and 72°F";
    }
}

BetaToolRunner toolRunner = client.beta().messages().toolRunner(
    MessageCreateParams.builder()
        .model("{{OPUS_ID}}")
        .maxTokens(1024L)
        .putAdditionalHeader("anthropic-beta", "structured-outputs-2025-11-13")
        .addTool(GetWeather.class)
        .addUserMessage("What's the weather in San Francisco?")
        .build());

for (BetaMessage message : toolRunner) {
    System.out.println(message);
}
\`\`\`

### 非ベータツール使用

ツール使用は、ベータ名前空間を必要とせずに、手動で定義したJSONスキーマを持つ`addTool(Tool)`を使用した非ベータの`com.anthropic.models.messages.MessageCreateParams`経由でも利用できます。ベータ名前空間は、クラスアノテーションの便利なレイヤー（`@JsonClassDescription`、`BetaToolRunner`）にのみ必要です。

### 手動ループ

手動ツールループの場合は、リクエストでJSONスキーマとしてツールを定義し、レスポンスの`tool_use`ブロックを処理し、`tool_result`を送り返し、`stop_reason`が`"end_turn"`になるまでループします。エージェントループパターンについては、[共有ツール使用コンセプト](../shared/tool-use-concepts.md)を参照してください。
