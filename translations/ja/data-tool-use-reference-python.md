<!--
name: 'データ: ツール使用リファレンス — Python'
description: ツールランナー、手動エージェンティックループ、コード実行、構造化出力を含むPythonツール使用リファレンス
ccVersion: 2.1.63
-->
# ツール使用 — Python

概念的な概要（ツール定義、ツール選択、ヒント）については、[shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)を参照してください。

## ツールランナー（推奨）

**ベータ版:** ツールランナーはPython SDKでベータ版です。

`@beta_tool`デコレータを使用してツールを型付き関数として定義し、`client.beta.messages.tool_runner()`に渡します：

```python
import anthropic
from anthropic import beta_tool

client = anthropic.Anthropic()

@beta_tool
def get_weather(location: str, unit: str = "celsius") -> str:
    """場所の現在の天気を取得します。

    Args:
        location: 市と州、例: San Francisco, CA。
        unit: 温度単位、「celsius」または「fahrenheit」。
    """
    # ここに実装
    return f"{location}は72°Fで晴れです"

# ツールランナーはエージェンティックループを自動的に処理
runner = client.beta.messages.tool_runner(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    tools=[get_weather],
    messages=[{"role": "user", "content": "パリの天気は？"}],
)

# 各反復はBetaMessageを生成。Claudeが完了すると反復停止
for message in runner:
    print(message)
```

非同期使用の場合、`async def`関数で`@beta_async_tool`を使用します。

**ツールランナーの主な利点:**

- 手動ループなし — SDKがツールの呼び出しと結果のフィードバックを処理
- デコレータによる型安全なツール入力
- 関数シグネチャからツールスキーマを自動生成
- Claudeがツール呼び出しをしなくなると自動的に反復停止

---

## 手動エージェンティックループ

ループを細かく制御する必要がある場合（例：カスタムログ、条件付きツール実行、ヒューマンインザループ承認）に使用：

```python
import anthropic

client = anthropic.Anthropic()
tools = [...]  # ツール定義
messages = [{"role": "user", "content": user_input}]

# エージェンティックループ: Claudeがツール呼び出しを停止するまで続行
while True:
    response = client.messages.create(
        model="{{OPUS_ID}}",
        max_tokens=4096,
        tools=tools,
        messages=messages
    )

    # Claudeが完了した場合（ツール呼び出しなし）、終了
    if response.stop_reason == "end_turn":
        break

    # サーバーサイドツールが反復制限に達した場合、再送信して続行
    if response.stop_reason == "pause_turn":
        messages = [
            {"role": "user", "content": user_input},
            {"role": "assistant", "content": response.content},
        ]
        continue

    # レスポンスからtool_useブロックを抽出
    tool_use_blocks = [b for b in response.content if b.type == "tool_use"]

    # アシスタントのレスポンス（tool_useブロックを含む）を追加
    messages.append({"role": "assistant", "content": response.content})

    # 各ツールを実行して結果を収集
    tool_results = []
    for tool in tool_use_blocks:
        result = execute_tool(tool.name, tool.input)  # あなたの実装
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": tool.id,  # tool_useブロックのidと一致する必要がある
            "content": result
        })

    # ツール結果をユーザーメッセージとして追加
    messages.append({"role": "user", "content": tool_results})

# 最終レスポンステキスト
final_text = next(b.text for b in response.content if b.type == "text")
```

---

## ツール結果の処理

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "パリの天気は？"}]
)

for block in response.content:
    if block.type == "tool_use":
        tool_name = block.name
        tool_input = block.input
        tool_use_id = block.id

        result = execute_tool(tool_name, tool_input)

        followup = client.messages.create(
            model="{{OPUS_ID}}",
            max_tokens=1024,
            tools=tools,
            messages=[
                {"role": "user", "content": "パリの天気は？"},
                {"role": "assistant", "content": response.content},
                {
                    "role": "user",
                    "content": [{
                        "type": "tool_result",
                        "tool_use_id": tool_use_id,
                        "content": result
                    }]
                }
            ]
        )
```

---

## 複数のツール呼び出し

```python
tool_results = []

for block in response.content:
    if block.type == "tool_use":
        result = execute_tool(block.name, block.input)
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": result
        })

# すべての結果を一度に送り返す
if tool_results:
    followup = client.messages.create(
        model="{{OPUS_ID}}",
        max_tokens=1024,
        tools=tools,
        messages=[
            *previous_messages,
            {"role": "assistant", "content": response.content},
            {"role": "user", "content": tool_results}
        ]
    )
```

---

## ツール結果のエラー処理

```python
tool_result = {
    "type": "tool_result",
    "tool_use_id": tool_use_id,
    "content": "エラー: 場所 'xyz' が見つかりません。有効な都市名を入力してください。",
    "is_error": True
}
```

---

## ツール選択

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    tools=tools,
    tool_choice={"type": "tool", "name": "get_weather"},  # 特定のツールを強制
    messages=[{"role": "user", "content": "パリの天気は？"}]
)
```

---

## コード実行

### 基本的な使用方法

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    messages=[{
        "role": "user",
        "content": "[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]の平均と標準偏差を計算してください"
    }],
    tools=[{
        "type": "code_execution_20260120",
        "name": "code_execution"
    }]
)

for block in response.content:
    if block.type == "text":
        print(block.text)
    elif block.type == "bash_code_execution_tool_result":
        print(f"stdout: {block.content.stdout}")
```

### 分析用のファイルアップロード

```python
# 1. ファイルをアップロード
uploaded = client.beta.files.upload(file=open("sales_data.csv", "rb"))

# 2. container_uploadブロック経由でコード実行に渡す
# コード実行はGA。Files APIはまだベータ版（extra_headers経由で渡す）
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    extra_headers={"anthropic-beta": "files-api-2025-04-14"},
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "この売上データを分析してください。トレンドを表示し、可視化を作成してください。"},
            {"type": "container_upload", "file_id": uploaded.id}
        ]
    }],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)
```

### 生成されたファイルの取得

```python
import os

OUTPUT_DIR = "./claude_outputs"
os.makedirs(OUTPUT_DIR, exist_ok=True)

for block in response.content:
    if block.type == "bash_code_execution_tool_result":
        result = block.content
        if result.type == "bash_code_execution_result" and result.content:
            for file_ref in result.content:
                if file_ref.type == "bash_code_execution_output":
                    metadata = client.beta.files.retrieve_metadata(file_ref.file_id)
                    file_content = client.beta.files.download(file_ref.file_id)
                    # パストラバーサルを防ぐためにbasenameを使用。結果を検証
                    safe_name = os.path.basename(metadata.filename)
                    if not safe_name or safe_name in (".", ".."):
                        print(f"無効なファイル名をスキップ: {metadata.filename}")
                        continue
                    output_path = os.path.join(OUTPUT_DIR, safe_name)
                    file_content.write_to_file(output_path)
                    print(f"保存: {output_path}")
```

### コンテナの再利用

```python
# 最初のリクエスト: 環境をセットアップ
response1 = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    messages=[{"role": "user", "content": "tabulateをインストールし、サンプルデータでdata.jsonを作成してください"}],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)

# レスポンスからコンテナIDを取得
container_id = response1.container.id

# 2回目のリクエスト: 同じコンテナを再利用
response2 = client.messages.create(
    container=container_id,
    model="{{OPUS_ID}}",
    max_tokens=4096,
    messages=[{"role": "user", "content": "data.jsonを読み込み、整形されたテーブルとして表示してください"}],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)
```

### レスポンス構造

```python
for block in response.content:
    if block.type == "text":
        print(block.text)  # Claudeの説明
    elif block.type == "server_tool_use":
        print(f"実行中: {block.name} - {block.input}")  # Claudeが行っていること
    elif block.type == "bash_code_execution_tool_result":
        result = block.content
        if result.type == "bash_code_execution_result":
            if result.return_code == 0:
                print(f"出力: {result.stdout}")
            else:
                print(f"エラー: {result.stderr}")
        else:
            print(f"ツールエラー: {result.error_code}")
    elif block.type == "text_editor_code_execution_tool_result":
        print(f"ファイル操作: {block.content}")
```

---

## メモリツール

### 基本的な使用方法

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=2048,
    messages=[{"role": "user", "content": "私の好みの言語がPythonであることを覚えておいてください。"}],
    tools=[{"type": "memory_20250818", "name": "memory"}],
)
```

### SDKメモリヘルパー

`BetaAbstractMemoryTool`をサブクラス化：

```python
from anthropic.lib.tools import BetaAbstractMemoryTool

class MyMemoryTool(BetaAbstractMemoryTool):
    def view(self, command): ...
    def create(self, command): ...
    def str_replace(self, command): ...
    def insert(self, command): ...
    def delete(self, command): ...
    def rename(self, command): ...

memory = MyMemoryTool()

# ツールランナーで使用
runner = client.beta.messages.tool_runner(
    model="{{OPUS_ID}}",
    max_tokens=2048,
    tools=[memory],
    messages=[{"role": "user", "content": "私の好みを覚えておいてください"}],
)

for message in runner:
    print(message)
```

完全な実装例については、WebFetchを使用してください：

- `https://github.com/anthropics/anthropic-sdk-python/blob/main/examples/memory/basic.py`

---

## 構造化出力

### JSON出力（Pydantic — 推奨）

```python
from pydantic import BaseModel
from typing import List
import anthropic

class ContactInfo(BaseModel):
    name: str
    email: str
    plan: str
    interests: List[str]
    demo_requested: bool

client = anthropic.Anthropic()

response = client.messages.parse(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": "抽出: Jane Doe (jane@co.com)はEnterprise希望、APIとSDKに興味あり、デモ希望。"
    }],
    output_format=ContactInfo,
)

# response.parsed_outputは検証済みのContactInfoインスタンス
contact = response.parsed_output
print(contact.name)           # "Jane Doe"
print(contact.interests)      # ["API", "SDKs"]
```

### 生のスキーマ

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": "情報を抽出: John Smith (john@example.com)はEnterpriseプラン希望。"
    }],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "email": {"type": "string"},
                    "plan": {"type": "string"},
                    "demo_requested": {"type": "boolean"}
                },
                "required": ["name", "email", "plan", "demo_requested"],
                "additionalProperties": False
            }
        }
    }
)

import json
data = json.loads(response.content[0].text)
```

### 厳密なツール使用

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "3月15日に2名で東京へのフライトを予約してください"}],
    tools=[{
        "name": "book_flight",
        "description": "目的地へのフライトを予約",
        "strict": True,
        "input_schema": {
            "type": "object",
            "properties": {
                "destination": {"type": "string"},
                "date": {"type": "string", "format": "date"},
                "passengers": {"type": "integer", "enum": [1, 2, 3, 4, 5, 6, 7, 8]}
            },
            "required": ["destination", "date", "passengers"],
            "additionalProperties": False
        }
    }]
)
```

### 両方を一緒に使用

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "来月パリへの旅行を計画してください"}],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "summary": {"type": "string"},
                    "next_steps": {"type": "array", "items": {"type": "string"}}
                },
                "required": ["summary", "next_steps"],
                "additionalProperties": False
            }
        }
    },
    tools=[{
        "name": "search_flights",
        "description": "利用可能なフライトを検索",
        "strict": True,
        "input_schema": {
            "type": "object",
            "properties": {
                "destination": {"type": "string"},
                "date": {"type": "string", "format": "date"}
            },
            "required": ["destination", "date"],
            "additionalProperties": False
        }
    }]
)
```
