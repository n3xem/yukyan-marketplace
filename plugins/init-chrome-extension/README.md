# init-chrome-extension

Chrome拡張機能の開発環境をゼロから構築するスキル。

## 技術スタック

- **WXT** - Chrome拡張機能フレームワーク
- **Biome** - リンター＆フォーマッター
- **TypeScript** - 型チェック
- **pnpm** - パッケージマネージャー
- **GitHub Actions** - CI/CD（lint, typecheck, build, release）

## 使い方

```
Chrome拡張を作りたい
拡張機能の環境構築をして
WXTでChrome拡張のプロジェクトを初期化して
```

## 生成されるもの

- WXT によるビルド・開発サーバー設定
- Biome によるリント・フォーマット設定
- TypeScript の厳密な型チェック設定
- GitHub Actions の CI（lint + typecheck + build）
- GitHub Actions の Release（タグプッシュで zip 生成）
- エントリポイントのテンプレート（popup, background, content など）
