---
paths:
  - plugins/**/*
  - external_plugins/**/*
---

# プラグイン開発ルール

## 構成

```
plugins/<plugin-name>/
├── .claude-plugin/plugin.json   # 必須
├── skills/<skill-name>/SKILL.md
├── commands/*.md
├── agents/*.md
└── README.md
```

## plugin.json

name, version, description, author, keywords を記載する。name は kebab-case。

## 変更時

plugin.json を変更したら marketplace.json も合わせて更新し、`claude plugin validate .` で検証する。

## 外部MCP専用プラグイン

MCP接続設定だけのプラグインは `external_plugins/` に配置し、plugin.json と .mcp.json のみ置く。
