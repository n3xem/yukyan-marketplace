---
name: suggest-allowlist
description: |
  会話中に使用したBashコマンドをもとに、~/.claude/settings.json の permissions.allow に追加すべきコマンドを提案・追加する。
  「allowlist」「許可リスト」「settings に追加」「コマンドを許可」「allowlist を更新」などで起動する。
allowed-tools: Read, Edit, Bash
---

# allowlist 提案スキル

## Instructions

会話中にユーザーが承認した Bash コマンドを振り返り、`~/.claude/settings.json` の `permissions.allow` に追加すべきコマンドパターンを提案・追加する。

### 1. 会話の振り返り

これまでの会話で実行された Bash コマンドを一覧化する。

### 2. 分類

各コマンドを以下の基準で分類する：

#### 追加を推奨（読み取り専用・副作用なし）
- ファイルやリソースの参照のみを行うコマンド
- 例: `gh issue list`, `gh issue view`, `gh pr list`, `gh pr view`, `pdftotext`, `jq`, `cat`, `ls`

#### 追加可能（軽い副作用あり、ユーザー判断）
- 頻繁に使い、毎回確認するのが煩わしいコマンド
- 例: `gh issue create`, `gh issue edit`, `gh issue close`, `npm install`, `npm run *`

#### 追加非推奨（破壊的・システム変更・セキュリティリスク）
- システムに変更を加えるコマンドや、破壊的操作
- 例: `brew install`, `rm`, `git push`, `git reset --hard`, `chmod`, `sudo *`

### 3. 提案

分類結果を以下の形式でテーブル表示する：

| コマンドパターン | 分類 | 理由 |
|---|---|---|
| `Bash(gh issue list *)` | 推奨 | 読み取り専用 |
| `Bash(gh issue create *)` | 可能 | 頻繁に使うなら便利 |
| `Bash(brew install *)` | 非推奨 | システム変更 |

パターン化のルール：
- 引数が変わりうる部分は `*` に置き換える
- コマンド名自体はそのまま残す
- `Bash(command *)` の形式にする（スペースの前に `*` を置くことで、引数違いにマッチさせる）
- 既に `~/.claude/settings.json` に存在するパターンは除外する

### 4. ユーザー確認

ユーザーにどのコマンドを追加するか確認する。「推奨」のものだけ入れるか、「可能」も含めるかを聞く。

### 5. 設定ファイルの更新

`~/.claude/settings.json` を読み込み、`permissions.allow` 配列にユーザーが選んだパターンを追加する。

- ファイルが存在しない場合は新規作成する
- `permissions` キーがない場合は追加する
- `allow` 配列がない場合は追加する
- 既存のエントリは保持し、重複は追加しない
- アルファベット順にソートする

### 6. 完了報告

更新後の `permissions.allow` の内容を表示する。
