<!--
name: 'データ: Message Batches APIリファレンス — Python'
description: バッチ作成、ステータスポーリング、50%割引での結果取得を含むPython Batches APIリファレンス
ccVersion: 2.1.63
-->
# Message Batches API — Python

Batches API（`POST /v1/messages/batches`）は、Messages APIリクエストを標準価格の50%で非同期に処理します。

## 重要事項

- バッチあたり最大100,000リクエストまたは256 MB
- ほとんどのバッチは1時間以内に完了。最大24時間
- 結果は作成後29日間利用可能
- すべてのトークン使用量で50%のコスト削減
- すべてのMessages API機能をサポート（ビジョン、ツール、キャッシングなど）

---

## バッチの作成

```python
import anthropic
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

message_batch = client.messages.batches.create(
    requests=[
        Request(
            custom_id="request-1",
            params=MessageCreateParamsNonStreaming(
                model="{{OPUS_ID}}",
                max_tokens=1024,
                messages=[{"role": "user", "content": "気候変動の影響を要約してください"}]
            )
        ),
        Request(
            custom_id="request-2",
            params=MessageCreateParamsNonStreaming(
                model="{{OPUS_ID}}",
                max_tokens=1024,
                messages=[{"role": "user", "content": "量子コンピューティングの基礎を説明してください"}]
            )
        ),
    ]
)

print(f"バッチID: {message_batch.id}")
print(f"ステータス: {message_batch.processing_status}")
```

---

## 完了のポーリング

```python
import time

while True:
    batch = client.messages.batches.retrieve(message_batch.id)
    if batch.processing_status == "ended":
        break
    print(f"ステータス: {batch.processing_status}, 処理中: {batch.request_counts.processing}")
    time.sleep(60)

print("バッチ完了！")
print(f"成功: {batch.request_counts.succeeded}")
print(f"エラー: {batch.request_counts.errored}")
```

---

## 結果の取得

> **注意:** 以下の例では`match/case`構文を使用しており、Python 3.10以上が必要です。それ以前のバージョンでは、代わりに`if/elif`チェーンを使用してください。

```python
for result in client.messages.batches.results(message_batch.id):
    match result.result.type:
        case "succeeded":
            print(f"[{result.custom_id}] {result.result.message.content[0].text[:100]}")
        case "errored":
            if result.result.error.type == "invalid_request":
                print(f"[{result.custom_id}] バリデーションエラー - リクエストを修正して再試行してください")
            else:
                print(f"[{result.custom_id}] サーバーエラー - 再試行しても安全です")
        case "canceled":
            print(f"[{result.custom_id}] キャンセルされました")
        case "expired":
            print(f"[{result.custom_id}] 期限切れ - 再送信してください")
```

---

## バッチのキャンセル

```python
cancelled = client.messages.batches.cancel(message_batch.id)
print(f"ステータス: {cancelled.processing_status}")  # "canceling"
```

---

## プロンプトキャッシング付きバッチ

```python
shared_system = [
    {"type": "text", "text": "あなたは文学アナリストです。"},
    {
        "type": "text",
        "text": large_document_text,  # すべてのリクエストで共有
        "cache_control": {"type": "ephemeral"}
    }
]

message_batch = client.messages.batches.create(
    requests=[
        Request(
            custom_id=f"analysis-{i}",
            params=MessageCreateParamsNonStreaming(
                model="{{OPUS_ID}}",
                max_tokens=1024,
                system=shared_system,
                messages=[{"role": "user", "content": question}]
            )
        )
        for i, question in enumerate(questions)
    ]
)
```

---

## 完全なエンドツーエンドの例

```python
import anthropic
import time
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

# 1. リクエストを準備
items_to_classify = [
    "製品の品質が素晴らしいです！",
    "ひどいカスタマーサービス、二度と利用しません。",
    "普通です、特別なことはありません。",
]

requests = [
    Request(
        custom_id=f"classify-{i}",
        params=MessageCreateParamsNonStreaming(
            model="{{HAIKU_ID}}",
            max_tokens=50,
            messages=[{
                "role": "user",
                "content": f"ポジティブ/ネガティブ/ニュートラルとして分類（一単語で）: {text}"
            }]
        )
    )
    for i, text in enumerate(items_to_classify)
]

# 2. バッチを作成
batch = client.messages.batches.create(requests=requests)
print(f"バッチ作成: {batch.id}")

# 3. 完了を待機
while True:
    batch = client.messages.batches.retrieve(batch.id)
    if batch.processing_status == "ended":
        break
    time.sleep(10)

# 4. 結果を収集
results = {}
for result in client.messages.batches.results(batch.id):
    if result.result.type == "succeeded":
        results[result.custom_id] = result.result.message.content[0].text

for custom_id, classification in sorted(results.items()):
    print(f"{custom_id}: {classification}")
```
