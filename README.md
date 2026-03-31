# yukyan-marketplace

個人用 Claude Code プラグインマーケットプレイス

## Directory Structure

```
.
├── .claude/
│   └── rules/                # 開発ルール（Claude Code が自動参照）
├── .claude-plugin/
│   └── marketplace.json      # Marketplace metadata（自動生成）
├── plugins/                  # プラグイン（スキル・コマンド・エージェント）
│   └── <your-plugin>/
├── external_plugins/         # 外部MCPサーバー接続のみのプラグイン
│   └── <service-name>/
├── CLAUDE.md                 # プロジェクト指示
└── README.md
```

## Setup

### 1. マーケットプレイスを登録する

Claude Code 上で以下を実行：

```bash
/plugin marketplace add n3xem/yukyan-marketplace
```

### 2. プラグインをインストールする

```bash
/plugin install <plugin-name>@yukyan-marketplace
```

### ローカルで開発中のプラグインを試す

```bash
claude --plugin-dir ./plugins/<plugin-name>
```

## Available Plugins

| プラグイン | 説明 |
|-----------|------|
| chrome-extension-store-review | Chrome拡張機能のCWS審査通過を包括的にレビュー（[n3xem/chrome-extension-store-review-skill](https://github.com/n3xem/chrome-extension-store-review-skill)） |
| [create-plugin](./plugins/create-plugin) | マーケットプレイスに新しいプラグインの雛形を作成 |
| [init-chrome-extension](./plugins/init-chrome-extension) | Chrome拡張機能の開発環境を構築（WXT + Biome + TS + pnpm + CI） |
| [jwt-design](./plugins/jwt-design) | JWTの設計・定義を壁打ち（アルゴリズム選定・クレーム設計・鍵管理） |

## Contributing

### プラグイン構造

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json       # 必須：プラグインメタデータ
├── commands/             # スラッシュコマンド
│   └── *.md
├── agents/               # カスタムエージェント
│   └── *.md
├── skills/               # スキル
│   └── <skill-name>/
│       └── SKILL.md
└── README.md             # プラグイン説明（推奨）
```

### 命名規則

- プラグイン名: `kebab-case`（例: `code-formatter`, `deployment-tools`）
- スペース不可、小文字とハイフンのみ

## Validation

```bash
claude plugin validate .
```
