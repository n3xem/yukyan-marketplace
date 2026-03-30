---
name: create-plugin
description: |
  マーケットプレイスに新しいプラグインの雛形を作成する。
  「プラグインを作りたい」「新しいスキルを追加したい」「プラグインを追加」などで起動する。
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# プラグイン作成スキル

## Instructions

ユーザーからプラグインの概要を聞き取り、このマーケットプレイスにプラグインの雛形を作成する。

### 1. ヒアリング

以下を確認する（ユーザーが既に伝えている場合はスキップ）：

- プラグイン名（kebab-case）
- 何をするプラグインか（description に使う）
- 含めるコンポーネントの種類（skill / command / agent のいずれか、複数可）

### 2. ディレクトリとファイルの作成

マーケットプレイスのルートは `${CLAUDE_PLUGIN_ROOT}/../..` にある。

以下の構成でファイルを生成する：

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json
├── skills/<skill-name>/
│   └── SKILL.md          # skill を含む場合
├── commands/
│   └── <command-name>.md  # command を含む場合
├── agents/
│   └── <agent-name>.md    # agent を含む場合
└── README.md
```

#### plugin.json

```json
{
  "name": "<plugin-name>",
  "version": "1.0.0",
  "description": "<ユーザーが伝えた説明>",
  "author": {
    "name": "yukyan"
  },
  "keywords": [<関連キーワード>]
}
```

#### SKILL.md

frontmatter には name, description, allowed-tools を含める。description にはトリガーワードを入れる。

#### command / agent

それぞれ `.claude/rules/` にあるルールに従って作成する。

### 3. marketplace.json の更新

`${CLAUDE_PLUGIN_ROOT}/../../.claude-plugin/marketplace.json` を読み込み、plugins 配列にエントリを追加する。

- アルファベット順でソート
- metadata.version の patch を +1

### 4. 検証

作成が完了したら以下を実行して検証する：

```bash
cd ${CLAUDE_PLUGIN_ROOT}/../.. && claude plugin validate .
```

### 5. 完了報告

作成したファイル一覧と、ローカルでテストするためのコマンドを提示する：

```bash
claude --plugin-dir ./plugins/<plugin-name>
```
