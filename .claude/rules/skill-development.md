---
paths: plugins/**/skills/**/*
---

# スキル開発ルール

## ファイル配置

`skills/<skill-name>/SKILL.md` に配置する。長くなる場合は REFERENCE.md 等に分割し、SKILL.md は 500行以下に保つ。

## SKILL.md の frontmatter

- `name`: kebab-case、最大64文字
- `description`: 何をするか・いつ使うか・トリガーワードを含める（最大1024文字）
- `allowed-tools`: 自動許可するツール

## MCP ツールの扱い

`allowed-tools` に `mcp__` ツールを直接指定しない。スキル本文内で ToolSearch を使って動的にロードする。

## スクリプト参照

パスは `${CLAUDE_PLUGIN_ROOT}` 経由で指定する。
