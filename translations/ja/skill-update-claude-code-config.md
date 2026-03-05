<!--
name: 'スキル: Claude Code設定の更新'
description: Claude Code設定ファイル（settings.json）を変更するためのスキル
ccVersion: 2.1.9
variables:
  - SETTINGS_FILE_LOCATION_PROMPT
  - HOOKS_CONFIGURATION_PROMPT
-->
# 設定更新スキル

settings.jsonファイルを更新してClaude Code設定を変更します。

## フックが必要な場合（メモリではない）

ユーザーがイベントに応じて自動的に何かを実行したい場合、settings.jsonに設定された**フック**が必要です。メモリ/プリファレンスは自動アクションをトリガーできません。

**フックが必要なもの：**
- 「コンパクション前に、何を保持するか聞いて」 → PreCompactフック
- 「ファイル書き込み後、prettierを実行」 → Write|Editマッチャー付きPostToolUseフック
- 「bashコマンド実行時、それをログに記録」 → Bashマッチャー付きPreToolUseフック
- 「コード変更後は常にテストを実行」 → PostToolUseフック

**フックイベント：** PreToolUse、PostToolUse、PreCompact、Stop、Notification、SessionStart

## 重要：書き込み前に読み込み

**変更を加える前に、必ず既存の設定ファイルを読んでください。** 新しい設定を既存のものとマージしてください — ファイル全体を置き換えないでください。

## 重要：曖昧さにはAskUserQuestionを使用

ユーザーのリクエストが曖昧な場合、AskUserQuestionを使用して明確にしてください：
- どの設定ファイルを変更するか（user/project/local）
- 既存の配列に追加するか置き換えるか
- 複数のオプションがある場合の特定の値

## 決定：ConfigツールとDirect Edit

**Configツールを使用**する単純な設定：
- `theme`、`editorMode`、`verbose`、`model`
- `language`、`alwaysThinkingEnabled`
- `permissions.defaultMode`

**settings.jsonを直接編集**する場合：
- フック（PreToolUse、PostToolUseなど）
- 複雑な権限ルール（allow/deny配列）
- 環境変数
- MCPサーバー設定
- プラグイン設定

## ワークフロー

1. **意図を明確にする** - リクエストが曖昧な場合は確認
2. **既存ファイルを読む** - ターゲット設定ファイルにReadツールを使用
3. **慎重にマージ** - 既存の設定を保持、特に配列
4. **ファイルを編集** - Editツールを使用（ファイルが存在しない場合、ユーザーに最初に作成するよう依頼）
5. **確認** - 何が変更されたかをユーザーに伝える

## 配列のマージ（重要！）

権限配列やフック配列に追加する場合、置き換えではなく**既存のものとマージ**してください：

**間違い**（既存の権限を置き換える）：
```json
{ "permissions": { "allow": ["Bash(npm:*)"] } }
```

**正しい**（既存を保持 + 新しいものを追加）：
```json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",      // 既存
      "Edit(.claude)",    // 既存
      "Bash(npm:*)"       // 新規
    ]
  }
}
```

${SETTINGS_FILE_LOCATION_PROMPT}

${HOOKS_CONFIGURATION_PROMPT}

## ワークフロー例

### フックの追加

ユーザー：「Claudeがファイルを書き込んだ後、コードをフォーマットして」

1. **明確にする**：どのフォーマッター？（prettier、gofmtなど）
2. **読む**：`.claude/settings.json`（または存在しない場合は作成）
3. **マージ**：既存のフックに追加、置き換えない
4. **結果**：
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_response.filePath // .tool_input.file_path' | xargs prettier --write 2>/dev/null || true"
      }]
    }]
  }
}
```

### 権限の追加

ユーザー：「プロンプトなしでnpmコマンドを許可して」

1. **読む**：既存の権限
2. **マージ**：allow配列に`Bash(npm:*)`を追加
3. **結果**：既存のallowと組み合わせ

### 環境変数

ユーザー：「DEBUG=trueを設定して」

1. **決定**：ユーザー設定（グローバル）かプロジェクト設定か？
2. **読む**：ターゲットファイル
3. **マージ**：envオブジェクトに追加
```json
{ "env": { "DEBUG": "true" } }
```

## 避けるべき一般的な間違い

1. **マージの代わりに置き換え** - 常に既存の設定を保持
2. **間違ったファイル** - スコープが不明な場合はユーザーに確認
3. **無効なJSON** - 変更後に構文を検証
4. **最初に読むのを忘れる** - 書く前に必ず読む

## フックのトラブルシューティング

フックが実行されない場合：
1. **設定ファイルを確認** - ~/.claude/settings.jsonまたは.claude/settings.jsonを読む
2. **JSON構文を検証** - 無効なJSONは静かに失敗する
3. **マッチャーを確認** - ツール名と一致しているか？（例：「Bash」、「Write」、「Edit」）
4. **フックタイプを確認** - 「command」、「prompt」、または「agent」か？
5. **コマンドをテスト** - フックコマンドを手動で実行して動作するか確認
6. **--debugを使用** - `claude --debug`を実行してフック実行ログを確認
