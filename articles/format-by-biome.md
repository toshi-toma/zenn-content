---
title: "コードフォーマッターとしてBiomeを使う"
emoji: "📎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript", "Biome"]
published: false
publication_name: "lincwell_inc"
---

Web 開発のための高速なツールチェーンである [Biome](https://biomejs.dev/) が使われることが増えてきました。今のところ、linter と formatter の機能を備えています。

普段、JavaScript および TypeScript プロジェクトのコードフォーマッターには Prettier を使っていますが、速度面の魅力がある biome を使って見ました。ESLint で使いたいルールや plugin があるので、lint 機能は使わないで ESLint に任せる前提です。

この記事では、biome をコードフォーマッターとしてだけ使うケースにフォーカスして書きます。

# セットアップ

ドキュメントに従ってセットアップするだけなので、導入はシンプルだと思います。

```bash
npm install --save-dev --save-exact @biomejs/biome
```

意図しないタイミングで patch バージョンなどが上がり、format が変わらないようにバージョンは範囲演算子を使わない形が推奨されています。

設定ファイルを出力します

```bash
npx @biomejs/biome init
```

出力された設定ファイル(`biome.json`)

```json
{
  "$schema": "https://biomejs.dev/schemas/1.5.3/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  }
}
```

フォーマットの設定項目が無いですが biome はデフォルトでフォーマット機能が有効になっています。

# フォーマットの実行

フォーマットの実行

```bash
npx @biomejs/biome format --write src
```

linter の機能は使わないので、設定で OFF にしておきます。

```json
{
  "$schema": "https://biomejs.dev/schemas/1.5.3/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": false
  }
}
```

## Imports Sorting

https://biomejs.dev/analyzer/

import の並び替え機能も用意されていますが、フォーマットではなく Analyzer という機能分類なので、利用したい場合は format ではなく check コマンドを使うことで、format と import の並び替えが適用されます。

```bash
npx @biomejs/biome check --apply src
```

設定項目は`organizeImports`です。

現状では Analyzer には Imports Sorting しか機能が無いようですが、今後利用したい機能が出てくる可能性を考慮して、フォーマッターとして使う場合も、check コマンドを利用するのが良さそうに思っています。

# デフォルトフォーマット

biome は Prettier と違う点としてデフォルトがインデントタブです。

インデントにタブを使うことはアクセシビリティ上の利点があるようです。

https://sosukesuzuki.dev/posts/prettier-uses-tabs/

Prettier との違いは以下のドキュメント参照

https://biomejs.dev/formatter/differences-with-prettier/

# CI 上で使うコマンド

```bash
npx @biomejs/biome ci src
```

organizeImports 使わない場合は以下でも OK

```bash
npx @biomejs/biome format src
```

# VSCode での保存時にフォーマット

公式で VSCode の拡張が用意されているのでインストールします。

https://marketplace.visualstudio.com/items?itemName=biomejs.biome

`.vscode/extensions.json`

```json
{
  "recommendations": ["biomejs.biome"]
}
```

`.vscode/settings.json`

```json
{
  "editor.defaultFormatter": "biomejs.biome",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports.biome": "explicit"
  }
}
```

TypeScript ファイルにだけ適用したい場合

```json
  "[typescript]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "biomejs.biome",
    "editor.formatOnSave": true
  },
  "editor.codeActionsOnSave": {
    "source.organizeImports.biome": "explicit"
  }
```

## organizeImports を有効にしていると、VSCode でファイル保存時に import 文が壊れる

以下の Issue で報告されている通り、VScode でファイル保存時に import 文が壊れるという問題がありました。

https://github.com/biomejs/biome/issues/1570

結果的に organizeImports は OFF にして、ESLint で import 文の order 調整をするようにしました。

# まとめ

Biomeをフォーマッターとして使う部分にフォーカスして紹介しました。
実際に使ってみて、普段の開発では1ファイルごとの保存時フォーマットなので速度面での違いは感じませんが、全ファイルに適用するケースでは高速だなと実感します。

これから使っていく中でPrettierとの差としては、TypeScriptの新しい構文のサポート速度あたりが気になっています。