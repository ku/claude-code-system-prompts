<!--
name: 'データ: ストリーミングリファレンス — Python'
description: 同期/非同期ストリーミングと異なるコンテンツタイプの処理を含むPythonストリーミングリファレンス
ccVersion: 2.1.63
-->
# ストリーミング — Python

## クイックスタート

```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "物語を書いてください"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### 非同期

```python
async with async_client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "物語を書いてください"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

---

## 異なるコンテンツタイプの処理

Claudeはテキスト、思考ブロック、またはツール使用を返す場合があります。それぞれ適切に処理してください：

> **Opus 4.6:** `thinking: {type: "adaptive"}`を使用してください。古いモデルでは、代わりに`thinking: {type: "enabled", budget_tokens: N}`を使用してください。

```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "この問題を分析してください"}]
) as stream:
    for event in stream:
        if event.type == "content_block_start":
            if event.content_block.type == "thinking":
                print("\n[思考中...]")
            elif event.content_block.type == "text":
                print("\n[応答:]")

        elif event.type == "content_block_delta":
            if event.delta.type == "thinking_delta":
                print(event.delta.thinking, end="", flush=True)
            elif event.delta.type == "text_delta":
                print(event.delta.text, end="", flush=True)
```

---

## ツール使用とストリーミング

Pythonのツールランナーは現在、完全なメッセージを返します。ツールでトークンごとのストリーミングが必要な場合は、手動ループ内で個々のAPI呼び出しにストリーミングを使用してください：

```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    tools=tools,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    response = stream.get_final_message()
    # response.stop_reason == "tool_use"の場合、ツール実行を続行
```

---

## 最終メッセージの取得

```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "こんにちは"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    # ストリーミング後に完全なメッセージを取得
    final_message = stream.get_final_message()
    print(f"\n\n使用トークン: {final_message.usage.output_tokens}")
```

---

## 進捗更新付きストリーミング

```python
def stream_with_progress(client, **kwargs):
    """進捗更新付きでレスポンスをストリーム。"""
    total_tokens = 0
    content_parts = []

    with client.messages.stream(**kwargs) as stream:
        for event in stream:
            if event.type == "content_block_delta":
                if event.delta.type == "text_delta":
                    text = event.delta.text
                    content_parts.append(text)
                    print(text, end="", flush=True)

            elif event.type == "message_delta":
                if event.usage and event.usage.output_tokens is not None:
                    total_tokens = event.usage.output_tokens

        final_message = stream.get_final_message()

    print(f"\n\n[使用トークン: {total_tokens}]")
    return "".join(content_parts)
```

---

## ストリームでのエラー処理

```python
try:
    with client.messages.stream(
        model="{{OPUS_ID}}",
        max_tokens=1024,
        messages=[{"role": "user", "content": "物語を書いてください"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
except anthropic.APIConnectionError:
    print("\n接続が失われました。再試行してください。")
except anthropic.RateLimitError:
    print("\nレート制限されました。しばらく待ってから再試行してください。")
except anthropic.APIStatusError as e:
    print(f"\nAPIエラー: {e.status_code}")
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

1. **常に出力をフラッシュ** — `flush=True`を使用してトークンを即座に表示
2. **部分的なレスポンスを処理** — ストリームが中断された場合、コンテンツが不完全な可能性がある
3. **トークン使用量を追跡** — `message_delta`イベントに使用量情報が含まれる
4. **タイムアウトを使用** — アプリケーションに適切なタイムアウトを設定
5. **デフォルトでストリーミング** — ストリーミング時でも`.get_final_message()`を使用して完全なレスポンスを取得し、個々のイベントを処理せずにタイムアウト保護を得る
