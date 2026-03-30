# yukyan-marketplace

yukyan の個人用 Claude Code プラグインマーケットプレイス

## プロジェクト概要

このリポジトリは、個人で使用する Claude Code プラグインを一元管理・配布するためのマーケットプレイスです。

## ディレクトリ構成

```
yukyan-marketplace/
├── .claude-plugin/
│   └── marketplace.json    # マーケットプレイスメタデータ
├── .claude/
│   └── rules/              # 開発ルール
├── plugins/                # プラグイン格納ディレクトリ（スキル・コマンド・エージェント）
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/
│       ├── agents/
│       └── skills/
├── external_plugins/       # 外部MCPサーバー接続専用プラグイン
│   └── <service-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── .mcp.json
└── README.md
```

## 開発ルール

### 言語
- 常に日本語で返答してください

### 開発プロセス
プラグインやスキルを実装する際は、以下の優先順位でドキュメントを参照すること：

1. **claude-code-guide エージェント**: 最新の公式ドキュメントを参照
2. **rules ディレクトリ**: プロジェクト固有のルールを参照

### プラグイン命名規則
- kebab-case 形式を使用（例: `code-formatter`, `deployment-tools`）
- スペースは使用不可
- ユニークな識別子を使用

### バージョニング
- セマンティックバージョニング（MAJOR.MINOR.PATCH）に従う
- MAJOR: 破壊的変更
- MINOR: 後方互換性のある新機能
- PATCH: バグ修正
- プラグインを変更した際は、必ず該当プラグインの `version` を上げること。Claude Code は各プラグインの `version` をキーにキャッシュするため、バージョンを上げないと変更が反映されない

### Author（著者）
- plugin.json の author は、そのプラグインにコミットした人の名前を設定する
- `git log --format="%aN" -- plugins/<plugin-name>/ | sort -u` でコミット履歴から著者を取得

### marketplace.json の更新

プラグインを追加・更新・削除した場合は、**必ず** `marketplace.json` も更新してください。

#### 更新手順

1. `plugins/<plugin-name>/.claude-plugin/plugin.json` の内容を確認
2. `.claude-plugin/marketplace.json` の `plugins` 配列を更新
   - 新規追加の場合: 配列に新しいエントリを追加（アルファベット順でソート）
   - 更新の場合: 該当エントリの `description`, `version`, `author`, `keywords` を更新
   - 削除の場合: 該当エントリを配列から削除
3. `metadata.version` をインクリメント（patch バージョンを +1）
4. `claude plugin validate .` で検証

#### plugins 配列のエントリ形式

ローカルパス（このリポジトリ内のプラグイン）:

```json
{
  "name": "plugin-name",
  "source": "./plugins/plugin-name",
  "description": "プラグインの説明",
  "version": "1.0.0",
  "author": {
    "name": "作者名"
  },
  "keywords": ["keyword1", "keyword2"]
}
```

外部gitリポジトリ（URL指定）:

```json
{
  "name": "external-plugin",
  "source": {
    "source": "url",
    "url": "https://github.com/org/plugin-repo.git"
  },
  "description": "外部リポジトリのプラグイン",
  "version": "1.0.0"
}
```

GitHubリポジトリ:

```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo"
  },
  "description": "GitHubリポジトリのプラグイン",
  "version": "1.0.0"
}
```

### external_plugins（外部MCPサーバー接続）

`external_plugins/` ディレクトリは、外部MCPサーバーへの接続設定のみを提供するプラグインを格納する。スキルやコマンドは持たず、`.mcp.json` でMCPサーバーの接続情報を定義するだけのシンプルな構成。

- 配置先: `external_plugins/<service-name>/`
- 必須ファイル: `.claude-plugin/plugin.json` と `.mcp.json`
- marketplace.json の source: `./external_plugins/<service-name>`
- インストール後、`/mcp` コマンドで OAuth 認証を行う

### allowed-tools における MCP ツールの制限

`allowed-tools` に `mcp__` プレフィックスのツールを指定する場合、**そのプラグイン自身の `.mcp.json` で定義されたMCPサーバーのツールのみ** 許可すること。

- MCPサーバーの登録名はユーザーの環境に依存するため、セキュリティリスクがある
- MCPツールを使用するスキルでは、スキル本文内で `ToolSearch` を使ってツールを動的にロードすること

### プラグインのソースについて

このマーケットプレイスは、自前でプラグインを実装するだけの場にとどまらない。外部の優れたスキルやプラグインを見つけて取り込むキュレーションの場でもある。

- **自前実装**: `plugins/` ディレクトリ内にプラグインを作成し、`source` にローカルパスを指定
- **外部から取り込む**: 良質なスキルやプラグインを持つ外部gitリポジトリを `source` に指定して配布
- **ブランチ・タグ固定**: `ref` や `sha` で特定バージョンにピン留め可能

## よく使うコマンド

```bash
# マーケットプレイスを検証
claude plugin validate .

# プラグインをローカルでテスト
claude --plugin-dir ./plugins/<plugin-name>

# マーケットプレイスをローカルで追加
/plugin marketplace add .
```
