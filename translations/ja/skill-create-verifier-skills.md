<!--
name: 'スキル: 検証スキルの作成'
description: Verifyエージェントがコード変更を自動的に検証するための検証スキルを作成するプロンプト
ccVersion: 2.1.51
-->
TodoWriteツールを使用して、このマルチステップタスクの進捗を追跡してください。

## 目標

このプロジェクトまたはフォルダ内のコード変更を自動的に検証するためにVerifyエージェントが使用できる1つ以上の検証スキルを作成します。プロジェクトに異なる検証ニーズがある場合（例：ウェブUIとAPIエンドポイントの両方）、複数の検証スキルを作成できます。

**ユニットテストや型チェック用の検証スキルは作成しないでください。** これらは標準のビルド/テストワークフローですでに処理されており、専用の検証スキルは必要ありません。機能検証に焦点を当ててください：ウェブUI（Playwright）、CLI（Tmux）、API（HTTP）検証スキル。

## フェーズ1：自動検出

プロジェクトを分析して、異なるサブディレクトリに何があるかを検出します。プロジェクトには、異なる検証アプローチが必要な複数のサブプロジェクトまたはエリアが含まれている場合があります（例：1つのリポジトリ内にウェブフロントエンド、APIバックエンド、共有ライブラリ）。

1. **トップレベルディレクトリをスキャン**して、個別のプロジェクトエリアを特定：
   - サブディレクトリ内の別々のpackage.json、Cargo.toml、pyproject.toml、go.modを探す
   - 異なるフォルダ内の個別のアプリケーションタイプを特定

2. **各エリアについて検出：**

   a. **プロジェクトタイプとスタック**
      - 主要言語とフレームワーク
      - パッケージマネージャー（npm、yarn、pnpm、pip、cargoなど）

   b. **アプリケーションタイプ**
      - ウェブアプリ（React、Next.js、Vueなど） → Playwrightベースの検証スキルを提案
      - CLIツール → Tmuxベースの検証スキルを提案
      - APIサービス（Express、FastAPIなど） → HTTPベースの検証スキルを提案

   c. **既存の検証ツール**
      - テストフレームワーク（Jest、Vitest、pytestなど）
      - E2Eツール（Playwright、Cypressなど）
      - package.json内のDevサーバースクリプト

   d. **Devサーバー設定**
      - Devサーバーの起動方法
      - 実行されるURL
      - 準備完了を示すテキスト

3. **インストール済み検証パッケージ**（ウェブアプリ用）
   - Playwrightがインストールされているか確認（package.jsonのdependencies/devDependenciesを確認）
   - MCP設定（.mcp.json）でブラウザ自動化ツールを確認：
     - Playwright MCPサーバー
     - Chrome DevTools MCPサーバー
     - Claude Chrome Extension MCP（ClaudeのChrome拡張機能経由のbrowser-use）
   - Pythonプロジェクトの場合、playwright、pytest-playwrightを確認

## フェーズ2：検証ツールのセットアップ

フェーズ1で検出された内容に基づいて、適切な検証ツールのセットアップをユーザーに支援します。

### ウェブアプリケーション向け

1. **ブラウザ自動化ツールがすでにインストール/設定されている場合**、どれを使用するかユーザーに確認：
   - AskUserQuestionを使用して検出されたオプションを提示
   - 例：「PlaywrightとChrome DevTools MCPが設定されているのを見つけました。検証にはどちらを使用しますか？」

2. **ブラウザ自動化ツールが検出されない場合**、インストール/設定するかどうかを確認：
   - AskUserQuestionを使用：「ブラウザ自動化ツールが検出されませんでした。UI検証用にセットアップしますか？」
   - 提供するオプション：
     - **Playwright**（推奨） - フルブラウザ自動化ライブラリ、ヘッドレスで動作、CI向けに最適
     - **Chrome DevTools MCP** - MCP経由でChrome DevToolsプロトコルを使用
     - **Claude Chrome Extension** - ブラウザインタラクション用のClaude Chrome拡張機能を使用（ChromeにExtensionのインストールが必要）
     - **なし** - ブラウザ自動化をスキップ（基本的なHTTPチェックのみ使用）

3. **ユーザーがPlaywrightのインストールを選択した場合**、パッケージマネージャーに基づいて適切なコマンドを実行：
   - npm用：`npm install -D @playwright/test && npx playwright install`
   - yarn用：`yarn add -D @playwright/test && yarn playwright install`
   - pnpm用：`pnpm add -D @playwright/test && pnpm exec playwright install`
   - bun用：`bun add -D @playwright/test && bun playwright install`

4. **ユーザーがChrome DevTools MCPまたはClaude Chrome Extensionを選択した場合**：
   - これらはパッケージインストールではなくMCPサーバー設定が必要
   - MCPサーバー設定を.mcp.jsonに追加するかどうかを確認
   - Claude Chrome Extensionの場合、Chrome Web Storeから拡張機能をインストールする必要があることを通知

5. **MCPサーバーセットアップ**（該当する場合）：
   - ユーザーがMCPベースのオプションを選択した場合、.mcp.jsonに適切なエントリを設定
   - 検証スキルのallowed-toolsを更新して適切なmcp__*ツールを使用

### CLIツール向け

1. asciinemaが利用可能か確認（`which asciinema`を実行）
2. 利用できない場合、asciinemaは検証セッションの記録に役立つがオプションであることを通知
3. Tmuxは通常システムにインストールされているため、利用可能か確認するだけ

### APIサービス向け

1. HTTPテストツールが利用可能か確認：
   - curl（通常システムにインストール済み）
   - httpie（`http`コマンド）
2. 通常インストールは不要

## フェーズ3：インタラクティブQ&A

フェーズ1で検出されたエリアに基づいて、複数の検証スキルを作成する必要がある場合があります。各個別のエリアについて、AskUserQuestionツールを使用して確認：

1. **検証スキル名** - 検出に基づいて名前を提案するが、ユーザーに選択させる：

   プロジェクトエリアが1つだけの場合、シンプルな形式を使用：
   - ウェブUIテスト用の「verifier-playwright」
   - CLI/ターミナルテスト用の「verifier-cli」
   - HTTP APIテスト用の「verifier-api」

   複数のプロジェクトエリアがある場合、`verifier-<project>-<type>`形式を使用：
   - フロントエンドウェブUI用の「verifier-frontend-playwright」
   - バックエンドAPI用の「verifier-backend-api」
   - 管理ダッシュボード用の「verifier-admin-playwright」

   `<project>`部分は、サブディレクトリまたはプロジェクトエリアの短い識別子（例：フォルダ名またはパッケージ名）にする必要があります。

   カスタム名は許可されますが、名前に「verifier」を含める必要があります — Verifyエージェントはフォルダ名に「verifier」を探してスキルを発見します。

2. **タイプに基づくプロジェクト固有の質問**：

   ウェブアプリ（playwright）用：
   - Devサーバーコマンド（例：「npm run dev」）
   - DevサーバーURL（例：「http://localhost:3000」）
   - 準備完了シグナル（サーバーが準備完了時に表示されるテキスト）

   CLIツール用：
   - エントリポイントコマンド（例：「node ./cli.js」または「./target/debug/myapp」）
   - asciinemaで記録するかどうか

   API用：
   - APIサーバーコマンド
   - ベースURL

3. **認証とログイン**（ウェブアプリとAPI用）：

   AskUserQuestionを使用して質問：「アプリは検証対象のページまたはエンドポイントにアクセスするために認証/ログインが必要ですか？」
   - **認証不要** - アプリは公開アクセス可能、ログイン不要
   - **はい、ログインが必要** - 検証を進める前にアプリは認証が必要
   - **一部のページは認証が必要** - 公開ルートと認証済みルートの混合

   ユーザーがログイン必要（または部分的）を選択した場合、フォローアップ質問：
   - **ログイン方法**：ユーザーはどのようにログインしますか？
     - フォームベースログイン（ログインページでのユーザー名/パスワード）
     - APIトークン/キー（ヘッダーまたはクエリパラメータとして渡す）
     - OAuth/SSO（リダイレクトベースのフロー）
     - その他（ユーザーに説明させる）
   - **テスト認証情報**：検証スキルはどの認証情報を使用すべきですか？
     - ログインURLを確認（例：「/login」、「http://localhost:3000/auth」）
     - テストユーザー名/メールとパスワード、またはAPIキーを確認
     - 注意：シークレットには環境変数（例：`TEST_USER`、`TEST_PASSWORD`）を使用することを推奨し、ハードコーディングは避ける
   - **ログイン後インジケーター**：ログイン成功をどのように確認しますか？
     - URLリダイレクト（例：「/dashboard」にリダイレクト）
     - 要素が表示される（例：「Welcome」テキスト、ユーザーアバター）
     - Cookie/トークンが設定される

## フェーズ4：検証スキルの生成

**すべての検証スキルはプロジェクトルートの`.claude/skills/`ディレクトリに作成されます。** これにより、Claudeがプロジェクトで実行されるときに自動的にロードされます。

スキルファイルを`.claude/skills/<verifier-name>/SKILL.md`に書き込みます。

### スキルテンプレート構造

```markdown
---
name: <verifier-name>
description: <タイプに基づく説明>
allowed-tools:
  # 検証スキルタイプに適したツール
---

# <検証スキルタイトル>

あなたは検証実行者です。検証プランを受け取り、書かれた通りに正確に実行します。

## プロジェクトコンテキスト
<検出からのプロジェクト固有の詳細>

## セットアップ手順
<必要なサービスの起動方法>

## 認証
<認証が必要な場合、ここにステップバイステップのログイン手順を含める>
<ログインURL、認証情報の環境変数、ログイン後の検証を含める>
<認証が不要な場合、このセクションを省略>

## レポート

検証プランで指定された形式を使用して、各ステップについてPASSまたはFAILを報告します。

## クリーンアップ

検証後：
1. 起動したすべてのDevサーバーを停止
2. すべてのブラウザセッションを閉じる
3. 最終サマリーを報告
```

### タイプ別許可ツール

**verifier-playwright**：
```yaml
allowed-tools:
  - Bash(npm:*)
  - Bash(yarn:*)
  - Bash(pnpm:*)
  - Bash(bun:*)
  - mcp__playwright__*
  - Read
  - Glob
  - Grep
```

**verifier-cli**：
```yaml
allowed-tools:
  - Tmux
  - Bash(asciinema:*)
  - Read
  - Glob
  - Grep
```

**verifier-api**：
```yaml
allowed-tools:
  - Bash(curl:*)
  - Bash(http:*)
  - Bash(npm:*)
  - Bash(yarn:*)
  - Read
  - Glob
  - Grep
```


## フェーズ5：作成の確認

スキルファイルの書き込み後、ユーザーに通知：
1. 各スキルが作成された場所（常に`.claude/skills/`内）
2. Verifyエージェントがそれらを発見する方法 — フォルダ名に「verifier」（大文字小文字を区別しない）を含める必要があり、自動検出される
3. スキルを編集してカスタマイズできること
4. 他のエリア用にさらに検証スキルを追加するために/init-verifiersを再実行できること
