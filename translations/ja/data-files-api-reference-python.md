<!--
name: 'データ: Files APIリファレンス — Python'
description: ファイルアップロード、一覧表示、削除、メッセージでの使用を含むPython Files APIリファレンス
ccVersion: 2.1.63
-->
# Files API — Python

Files APIは、Messages APIリクエストで使用するファイルをアップロードします。コンテンツブロックで`file_id`を参照することで、複数のAPI呼び出しにわたる再アップロードを回避できます。

**ベータ版:** API呼び出しで`betas=["files-api-2025-04-14"]`を渡してください（SDKは必要なヘッダーを自動的に設定します）。

## 重要事項

- 最大ファイルサイズ: 500 MB
- 総ストレージ: 組織あたり100 GB
- ファイルは削除されるまで保持されます
- ファイル操作（アップロード、一覧表示、削除）は無料です。メッセージで使用されるコンテンツは入力トークンとして課金されます
- Amazon BedrockおよびGoogle Vertex AIでは利用できません

---

## ファイルのアップロード

```python
import anthropic

client = anthropic.Anthropic()

uploaded = client.beta.files.upload(
    file=("report.pdf", open("report.pdf", "rb"), "application/pdf"),
)
print(f"ファイルID: {uploaded.id}")
print(f"サイズ: {uploaded.size_bytes} バイト")
```

---

## メッセージでのファイル使用

### PDF / テキストドキュメント

```python
response = client.beta.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "このレポートの重要な発見を要約してください。"},
            {
                "type": "document",
                "source": {"type": "file", "file_id": uploaded.id},
                "title": "Q4レポート",           # オプション
                "citations": {"enabled": True}   # オプション、引用を有効化
            }
        ]
    }],
    betas=["files-api-2025-04-14"],
)
print(response.content[0].text)
```

### 画像

```python
image_file = client.beta.files.upload(
    file=("photo.png", open("photo.png", "rb"), "image/png"),
)

response = client.beta.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "この画像には何が写っていますか？"},
            {
                "type": "image",
                "source": {"type": "file", "file_id": image_file.id}
            }
        ]
    }],
    betas=["files-api-2025-04-14"],
)
```

---

## ファイルの管理

### ファイル一覧の取得

```python
files = client.beta.files.list()
for f in files.data:
    print(f"{f.id}: {f.filename} ({f.size_bytes} バイト)")
```

### ファイルメタデータの取得

```python
file_info = client.beta.files.retrieve_metadata("file_011CNha8iCJcU1wXNR6q4V8w")
print(f"ファイル名: {file_info.filename}")
print(f"MIMEタイプ: {file_info.mime_type}")
```

### ファイルの削除

```python
client.beta.files.delete("file_011CNha8iCJcU1wXNR6q4V8w")
```

### ファイルのダウンロード

コード実行ツールまたはスキルによって作成されたファイルのみダウンロードできます（ユーザーがアップロードしたファイルは対象外）。

```python
file_content = client.beta.files.download("file_011CNha8iCJcU1wXNR6q4V8w")
file_content.write_to_file("output.txt")
```

---

## 完全なエンドツーエンドの例

ドキュメントを一度アップロードし、複数の質問をする：

```python
import anthropic

client = anthropic.Anthropic()

# 1. 一度アップロード
uploaded = client.beta.files.upload(
    file=("contract.pdf", open("contract.pdf", "rb"), "application/pdf"),
)
print(f"アップロード完了: {uploaded.id}")

# 2. 同じfile_idを使用して複数の質問をする
questions = [
    "主要な契約条件は何ですか？",
    "解約条項は何ですか？",
    "支払いスケジュールを要約してください。",
]

for question in questions:
    response = client.beta.messages.create(
        model="{{OPUS_ID}}",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": [
                {"type": "text", "text": question},
                {
                    "type": "document",
                    "source": {"type": "file", "file_id": uploaded.id}
                }
            ]
        }],
        betas=["files-api-2025-04-14"],
    )
    print(f"\n質問: {question}")
    print(f"回答: {response.content[0].text[:200]}")

# 3. 完了したらクリーンアップ
client.beta.files.delete(uploaded.id)
```
