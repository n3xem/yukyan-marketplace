---
name: init-chrome-extension
description: |
  Chrome拡張機能の開発環境をゼロから構築する。
  「Chrome拡張を作りたい」「拡張機能の環境構築」「Chrome extension のプロジェクト初期化」「WXTでChrome拡張」などで起動する。
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Chrome拡張機能 環境構築スキル

WXT + Biome + TypeScript + pnpm + GitHub Actions を使ったモダンなChrome拡張機能の開発環境を構築する。

## Instructions

### 1. ヒアリング

以下を確認する（ユーザーが既に伝えている場合はスキップ）：

- 拡張機能の名前
- 何をする拡張機能か（manifest の description に使う）
- 必要な permissions（activeTab, storage, contextMenus, tabs など。不明なら後で追加可能と伝える）
- host_permissions が必要か（特定のドメインやlocalhostへの通信がある場合）
- 必要なエントリポイント（以下から選択、複数可。デフォルトは popup + background）
  - popup: ポップアップUI
  - background: バックグラウンドスクリプト
  - content: コンテンツスクリプト（Webページに注入）
  - options: オプションページ
  - sidepanel: サイドパネル

### 2. プロジェクト構成の生成

カレントディレクトリをプロジェクトルートとして、以下の構成でファイルを生成する。
ユーザーが既存リポジトリの中に作りたい場合は、サブディレクトリ（例: `extension/`）を提案する。

```
<project-root>/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
├── public/
│   └── icon/
│       └── (アイコンは後で追加するよう案内)
├── src/
│   ├── entrypoints/
│   │   ├── background.ts        # background を含む場合
│   │   ├── content.ts           # content を含む場合
│   │   └── popup/               # popup を含む場合
│   │       ├── index.html
│   │       ├── main.ts
│   │       └── style.css
│   └── lib/                     # 共有ユーティリティ用（空ディレクトリは作らない）
├── .gitignore
├── biome.json
├── package.json
├── tsconfig.json
└── wxt.config.ts
```

### 3. 各ファイルの内容

#### package.json

```json
{
  "name": "<拡張機能名をkebab-caseで>",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "wxt",
    "build": "wxt build",
    "zip": "wxt zip",
    "typecheck": "wxt prepare && tsc --noEmit",
    "check": "biome check --write ."
  },
  "devDependencies": {
    "@biomejs/biome": "^2.4.9",
    "typescript": "^6.0.2",
    "wxt": "^0.20.11"
  }
}
```

#### wxt.config.ts

```typescript
import { defineConfig } from "wxt";

export default defineConfig({
  srcDir: "src",
  manifest: {
    name: "<拡張機能名>",
    description: "<ユーザーが伝えた説明>",
    version: "0.1.0",
    permissions: [<ヒアリング結果>],
    // host_permissions が必要な場合のみ追加
  },
});
```

#### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["ESNext", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noEmit": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*", ".wxt/**/*"]
}
```

#### biome.json

`vcs.root` はプロジェクトルートからの相対パスに合わせて調整する。
単独リポジトリなら `"."`, モノレポのサブディレクトリなら `".."` など。

```json
{
  "$schema": "https://biomejs.dev/schemas/2.4.9/schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true,
    "root": "."
  },
  "files": {
    "ignoreUnknown": true,
    "includes": [
      "**",
      "!**/.wxt",
      "!**/node_modules",
      "!**/dist",
      "!**/*.css",
      "!**/*.html"
    ]
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedImports": "error",
        "noUnusedVariables": "error"
      },
      "style": {
        "noNonNullAssertion": "off"
      }
    }
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "double",
      "semicolons": "always"
    }
  }
}
```

#### .gitignore

```
node_modules/
.DS_Store
/tmp/
*.log
.output/
.wxt/
```

#### .github/workflows/ci.yml

`working-directory` はプロジェクトルートに合わせる。リポジトリ直下なら `.`、サブディレクトリなら `extension` など。
`cache-dependency-path` も同様に合わせる。

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: <プロジェクトルートのパス>
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 10
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm
          cache-dependency-path: <プロジェクトルートのパス>/pnpm-lock.yaml
      - run: pnpm install --frozen-lockfile
      - run: pnpm biome check .
      - run: pnpm typecheck
      - run: pnpm build
```

#### .github/workflows/release.yml

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: <プロジェクトルートのパス>
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 10
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm
          cache-dependency-path: <プロジェクトルートのパス>/pnpm-lock.yaml
      - run: pnpm install --frozen-lockfile
      - run: pnpm zip
      - name: Create GitHub Release
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          gh release create "${{ github.ref_name }}" \
            --repo "${{ github.repository }}" \
            --title "${{ github.ref_name }}" \
            --generate-notes \
            .output/*.zip
```

#### エントリポイントのテンプレート

**background.ts**（background を含む場合）:

```typescript
export default defineBackground(() => {
  console.log("Background script loaded");
});
```

**content.ts**（content を含む場合）:

```typescript
export default defineContentScript({
  matches: ["<all_urls>"],
  main() {
    console.log("Content script loaded");
  },
});
```

**popup/index.html**（popup を含む場合）:

```html
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><拡張機能名></title>
    <link rel="stylesheet" href="./style.css" />
  </head>
  <body>
    <div id="app">
      <h1><拡張機能名></h1>
    </div>
    <script type="module" src="./main.ts"></script>
  </body>
</html>
```

**popup/main.ts**:

```typescript
console.log("Popup loaded");
```

**popup/style.css**:

```css
body {
  width: 360px;
  padding: 16px;
  font-family: system-ui, sans-serif;
}
```

### 4. 依存関係のインストール

ファイル生成後、プロジェクトルートで以下を実行する：

```bash
pnpm install
```

### 5. 初期動作確認

インストール完了後、型チェックとビルドが通ることを確認する：

```bash
pnpm typecheck && pnpm build
```

エラーがあれば修正する。

### 6. 完了報告

生成したファイル一覧と、以下の開発コマンドを案内する：

```
pnpm dev       # 開発サーバー起動（ホットリロード）
pnpm build     # 本番ビルド
pnpm zip       # Chrome Web Store 提出用 zip 作成
pnpm typecheck # 型チェック
pnpm check     # Biome によるリント・フォーマット
```

また、Chrome での読み込み方法も案内する：

1. `chrome://extensions/` を開く
2. 「デベロッパーモード」を有効にする
3. 「パッケージ化されていない拡張機能を読み込む」で `.output/chrome-mv3/` を選択

アイコンが未設定の場合は、`public/icon/` に 16x16, 48x48, 128x128 の PNG を配置するよう案内する。
