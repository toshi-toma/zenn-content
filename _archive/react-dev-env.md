---
title: Reactを使ったモダンなフロントエンド開発の環境構築
tags: React TypeScript storybook prettier ESLint
author: toshi-toma
slide: false
---
# はじめに

Reactを中心としたフロントエンド開発において、以下のような構成を見かけることが多いと思います。

- UIライブラリとして`React`
- 型のある言語として`TypeScript`
- スタイル定義として`styled-components`
- コンポーネントの開発環境として`Storybook`
- Linterとして`ESLint`
- Formatterとして`Prettier`

この記事では、各種ライブラリについて紹介したのち、それらを使う場合の環境構築についてハンズオン形式で説明します。

※ アプリケーションを開発する際に必要になる設定が抜けていたので、追記しました。

# 各種ライブラリの紹介

まず、各ライブラリがどのようなものなのかを簡単に紹介します。

ライブラリの使い方などは公式ドキュメントなどを参照するようにしてください。

## React

[ドキュメント](https://reactjs.org/)

ReactはUI(ボタンやフォームなど)コンポーネントを作成するためのJavaScriptライブラリです。
似た立ち位置として、[Vue.js](https://vuejs.org/index.html)が上げられることも多いと思います。

公式ドキュメントが非常に充実しているので、初めての方は、[Tutorial](https://reactjs.org/tutorial/tutorial.html)をやってみるのが良さそうです。

## TypeScript

[ドキュメント](https://www.typescriptlang.org/)

TypeScriptは型を持ったJavaScriptのスーパーセットとして人気なプログラミング言語です。
コンパイルすると、JavaScriptのソースコードになります。

公式ドキュメント以外に、[TypeScript Deep Dive](https://typescript-jp.gitbook.io/deep-dive/)が情報量が多く、分かりやすいと思います。


## styled-components

[ドキュメント](https://www.styled-components.com/)

styled-componentsは、 CSSとほぼ同様のシンタックスを使ってスタイリングされたReactコンポーネントを作成できるライブラリです。
`.css`ファイルにCSSを書くのではなく、`.js`や`.tsx`ファイル内でCSSを書くようなイメージです。

公式ドキュメントを参照しながら、使ってみるのが良いと思います。

## Storybook

[ドキュメント](https://storybook.js.org/)

Storybookは、UIコンポーネントをアプリケーションとは独立した環境で開発できるツールです。
静的サイトとしてビルドすることで、コンポーネントのカタログとしても利用できます。

[React用のチュートリアル](https://www.learnstorybook.com/react/en/get-started)も用意されています。

## ESLint

[ドキュメント](https://eslint.org/)

ESLintはJavaScriptのLinterです。
Linterを使うことで、コードの一貫性を高め、バグを未然に防げる可能性が高まります。

[@typescript-eslint/eslint-plugin](https://www.npmjs.com/package/@typescript-eslint/eslint-plugin)を使うことで、TypeScriptでも利用可能です。

## Prettier

[ドキュメント](https://prettier.io/)

Prettierはフロントエンド開発において人気なコードフォーマッターです。
JavaScriptをはじめとした複数の言語をサポートしています。

Prettierを使うことで、コードのフォーマットを意識することなく統一することができます。

# 環境構築

GitHubでリポジトリの作成から各種ライブラリのインストール、設定ファイルの作成などを行います。

最終的に、以下の事ができたらOKです。

- `npm run storybook`: Storybookを起動して、React+TypeScript+styled-componentsで作成したコンポーネントが表示できる
- `npm run lint`: ESLint + Prettierを使ってコードのチェックが行える
- `npm run tsc`: TypeScriptのコードに対してコンパイルチェックが行える

## Step0 事前準備

npm/Node.js/gitがインストールされていることを確認してください。

```console
$ git --version
git version 2.20.1 (Apple Git-117)
$ node -v
v12.4.0
$ npm -v
6.9.0 
```

バージョンを揃える必要はありませんが、必要に応じてLTSまたは最新版を利用するようにしましょう。

## Step1 リポジトリの作成/git clone/gitignore

GitHubで今回作業をするリポジトリを作成しましょう。

[新しいリポジトリを作成](https://github.com/new)

リポジトリを作成後は、そのリポジトリをクローンしておいてください。

また、好きなgitignoreをリポジトリに配置しておきます。

例） Node.gitignoreを利用する場合

```console
$ wget https://raw.githubusercontent.com/github/gitignore/master/Node.gitignore
$ mv Node.gitignore .gitignore
```

これでリポジトリの準備は整いました！

## Step2 package.jsonを作成する

npmのinitコマンドを使って、package.jsonの雛形を作成しましょう。

```console
$ npm init -y

Wrote to /dev/github.com/toshi-toma/todo-app-react/package.json:

{
  "name": "todo-app-react",
  "version": "1.0.0",
  "description": "A Todo Application  built with React, styled-components, TypeScript, E
SLint, Prettier, and Storybook",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+ssh://git@github.com/toshi-toma/todo-app-react.git"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/toshi-toma/todo-app-react/issues"
  },
  "homepage": "https://github.com/toshi-toma/todo-app-react#readme"
}

```

npmのCLIには便利なコマンドがたくさん用意されています。[ドキュメント](https://docs.npmjs.com/cli-documentation/cli)をさっと見るだけでも面白いと思います。

## Step3 Reactでコンポーネントを作成して、Storybookに表示する

Reactでコンポーネントを作成して、そのコンポーネントをStorybookに表示して動作や見た目を確認していくとスムーズな開発が行えます。

まずはReactとstyled-componentsでシンプルなコンポーネントを作成してみましょう。

必要なライブラリをインストールします。

```console
$ npm i react react-dom  styled-components
...
+ react@16.9.0
+ react-dom@16.9.0
+ styled-components@4.3.2
added 11 packages from 12 contributors, updated 2 packages and audited 31753 packages in 5.34s
```

Reactとstyled-componentsでシンプルなボタンを作成してみましょう。

`src/index.js`

```jsx
import React from "react";
import styled from "styled-components";

const Button = styled.button`
  background-color: #454545;
  color: #fff;
`;

const SimpleButton = () => <Button>button</Button>; // 省略可能

export default SimpleButton;
```

次に、作成したボタンをStorybookで表示しましょう。

Googleで「Storybook React」と検索すると`Storybook for React`というタイトルの公式ドキュメントが出てくると思います。

[Storybook for React](https://storybook.js.org/docs/guides/guide-react/)

ドキュメントを読むと、Automatic setupコマンドが用意されているようです。

実行してみましょう。

```console
$ npx -p @storybook/cli sb init --type react

...

+ @storybook/react@5.1.10
+ @storybook/addon-links@5.1.10
+ @babel/core@7.5.5
+ babel-loader@8.0.6
+ @storybook/addon-actions@5.1.10
+ @storybook/addons@5.1.10
added 1147 packages from 843 contributors and audited 31533 packages in 41.347s

...

• Installing dependencies. ✓

To run your storybook, type:

   npm run storybook 

For more information visit: https://storybook.js.org

```

これでStorybookの環境構築及びReactコンポーネントを表示する準備が整いました！簡単ですね。

`npm run storybook`を実行するとStorybookが表示されると思います。

あとは、先程作成したボタンをStorybookに表示しましょう。

`stories/index.stories.js`

```jsx
import React from 'react';

import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';

import Button from '../src/index';

storiesOf('Button', module)
  .add('with text', () => <Button onClick={action('clicked')}>Hello Button</Button>);
```

無事にReactとstyled-componentsで作成したコンポーネントをStorybookに表示することができました🎉

## Step4 TypeScriptでコンポーネントを作成して、Storybookに表示する

次はReactとstyled-componentsに加えてTypeScriptを使ってコンポーネントを作成して、そのコンポーネントをStorybookに表示します。

まずはStep3で作成したコンポーネントをTypeScript化しましょう。

必要なライブラリをインストールします。
TypeScriptに加えて、型情報もインストールします。

```console
$ npm i -D typescript
$ npm i -D @types/{react,react-dom,styled-components}

+ @types/styled-components@4.1.18
+ @types/react-dom@16.8.5
+ @types/react@16.9.1
```

TypeScriptの設定ファイルの雛形を`tsc`コマンドで作成します。

```console
$ npx tsc --init  
message TS6071: Successfully created a tsconfig.json file.
```

`tsconfig.json`というファイルが作成されたので、React用に編集します。

tsconfig.json

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "lib": ["es2018", "dom"],
    "jsx": "react",
    "strict": true,
    "rootDirs": [
      "src"
    ],
    "types": [
      "react"
    ],
    "esModuleInterop": true
  },
  "include": [
    "src"
  ],
  "exclode": [
    "node_modules"
  ]
}
```

これでTypeScriptを使う準備が整ったので、コンポーネントをTypeScriptで書き換えてみます。

ReactとTypeScriptを使う場合のコンポーネントファイルの拡張子は`.tsx`になります。

```shell
mv src/index.js src/index.tsx
```

`src/index.tsx`

```tsx
import React, { FC } from "react";
import styled from "styled-components";

const Button = styled.button`
  background-color: #454545;
  color: #fff;
`;

interface Props {
  onClick: () => void;
}

const SimpleButton: FC<Props> = () => <Button>Done</Button>;

export default SimpleButton;
```

最後に書き換えたコンポーネントをStorybookで表示します。

TypeScriptを使う場合、別途Storybookの設定が必要です。

必要な設定については、[公式ドキュメント](https://storybook.js.org/docs/configurations/typescript-config/)が用意されています。

ドキュメントを読むと、必要な設定が`@storybook/preset-typescript`として用意されているので、これを利用します。

必要なライブラリをインストールしましょう。

```console
$ npm i -D @storybook/preset-typescript react-docgen-typescript-loader ts-loader 
$ npm i -D @types/{storybook__addon-actions,storybook__react}

...

+ ts-loader@6.0.4
+ react-docgen-typescript-loader@3.1.1
+ @storybook/preset-typescript@1.1.0

...

+ @types/storybook__addon-actions@3.4.3
+ @types/storybook__react@4.0.2

```

`npm i -D xxx`を実行したログに以下のようなWarningが表示されていると思います。

```shell
...

npm WARN @storybook/preset-typescript@1.1.0 requires a peer of @types/webpack@* but none is installed. You must install peer dependencies yourself

...
```

`@storybook/preset-typescript@1.1.0`は`@types/webpack@*`をpeer dependencyとしているようです。

`@storybook/preset-typescript`のリポジトリを開いて、[package.json](https://github.com/storybookjs/presets/blob/master/packages/preset-typescript/package.json)を見てみましょう。

`preset-typescript/package.json`

```js
  ...
  "peerDependencies": {
    "@types/webpack": "*",
    "react-docgen-typescript-loader": "*",
    "ts-loader": "*"
  }
```

peerDependenciesとして`@types/webpack`が指定されているので、インストールしておきましょう。

```console
$ npm i -D @types/webpack
+ @types/webpack@4.32.1
```

あとは`.storybook/presets.js`を作成して、`.storybook/presets.js`と`.storybook/config.js`を以下のように編集するだけです。

presets.js

```js
module.exports = ["@storybook/preset-typescript"];
```

config.js

```diff
import { configure } from '@storybook/react';

// automatically import all files ending in *.stories.js
- const req = require.context('../stories', true, /\.stories\.js$/);
+ const req = require.context('../src', true, /\.stories\.tsx$/);
function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

storiesファイルの拡張子も`.js`から`.tsx`に変更しましょう。

```console
$ mkdir src/stories
$ mv stories/index.stories.js src/stories/index.stories.tsx
$ rm -rf stories
```

`npm run storybook`でコンポーネントが表示されたら成功です🎉

また、TypeScriptで書いたコードはコンパイルを実行することで静的型付けによる型チェックを行えます。

型チェックを行なうnpm scriptsを追加しておきましょう。

package.json

```diff
"scripts": {
     ...
+    "tsc": "tsc --noEmit",
```

```console
$ npm run tsc
```

ここまでの作業によるpackage.jsonの差分を出しておきます。

```diff
   ...

   "description": "A Todo Application  built with React, styled-components, TypeScript, ESLint, Prettier, and Storybook",
   "main": "index.js",
   "scripts": {
-    "test": "echo \"Error: no test specified\" && exit 1"
+    "test": "echo \"Error: no test specified\" && exit 1",
+    "tsc": "tsc --noEmit",
+    "storybook": "start-storybook -p 6006",
+    "build-storybook": "build-storybook"
   },
   "repository": {
     "type": "git",
@@ -16,5 +18,28 @@
   "bugs": {
     "url": "https://github.com/toshi-toma/todo-app-react/issues"
   },
-  "homepage": "https://github.com/toshi-toma/todo-app-react#readme"
+  "homepage": "https://github.com/toshi-toma/todo-app-react#readme",
+  "dependencies": {
+    "react": "^16.9.0",
+    "react-dom": "^16.9.0",
+    "styled-components": "^4.3.2"
+  },
+  "devDependencies": {
+    "@babel/core": "^7.5.5",
+    "@storybook/addon-actions": "^5.1.10",
+    "@storybook/addon-links": "^5.1.10",
+    "@storybook/addons": "^5.1.10",
+    "@storybook/preset-typescript": "^1.1.0",
+    "@storybook/react": "^5.1.10",
+    "@types/react": "^16.9.1",
+    "@types/react-dom": "^16.8.5",
+    "@types/storybook__addon-actions": "^3.4.3",
+    "@types/storybook__react": "^4.0.2",
+    "@types/styled-components": "^4.1.18",
+    "@types/webpack": "^4.32.1",
+    "babel-loader": "^8.0.6",
+    "react-docgen-typescript-loader": "^3.1.1",
+    "ts-loader": "^6.0.4",
+    "typescript": "^3.5.3"
+  }
 }
```

## Step5 ESLintとPrettierでLintとFormatを導入する

TypeScriptのコードをESLintでLint、PrettierでFormatできるようにしましょう。

今回は手順をいくつか省略できるように、npmで公開されている`eslint-config`を利用します。
ここで利用するルールは自分の好みに応じてカスタマイズしてOKです。

[npmの検索窓で「eslint typescript prettier」と検索](https://www.npmjs.com/search?q=eslint%20typescript%20prettier)してみましょう。

以下の手順では、airbnbが提供している`eslint-config-airbnb`とTypeScript、Prettierのサポートが入った`eslint-config-airbnb-typescript-prettier`を利用します。

[eslint-config-airbnb-typescript-prettier](https://www.npmjs.com/package/eslint-config-airbnb-typescript-prettier)

パッケージのページに書いてある説明を参考に、必要なライブラリをインストールします。

```console
$ npm i -D eslint@^5.3.0 prettier@^1.18.2 eslint-config-airbnb-typescript-prettier
+ prettier@1.18.2
+ eslint@5.16.0
+ eslint-config-airbnb-typescript-prettier@1.2.1
added 106 packages from 69 contributors and audited 32518 packages in 10.364s
```

次にESLintの設定ファイルである`.eslintrc.js`を作成して、以下のように設定を書きます。

.eslintrc.js

```js
module.exports = {
  extends: "airbnb-typescript-prettier"
};
```

ESLintの設定が終わったら、npm scriptsを追加しておきましょう。

package.json

```diff
"scripts": {
+    "lint": "eslint --ext .ts,.tsx src/",
+    "lint:fix": "eslint --ext .ts,.tsx src/ --fix",
```

これで次から`npm run lint`を実行することで、`src`ディレクトリ以下のTypeScriptファイルに対してLintとFormatチェックが実行されます。

```console
$ npm run lint

> eslint --ext .ts,.tsx src/


/dev/github.com/toshi-toma/todo-app-react/src/index.tsx
  15:23  error  Insert `⏎`  prettier/prettier

/dev/github.com/toshi-toma/todo-app-react/src/stories/index.stories.tsx
  1:19  error  Replace `'react'` with `"react"`                                                                                                                                                                                                      prettier/prettier
  3:1   error  '@storybook/react' should be listed in the project's dependencies, not devDependencies                                                                                                                                                import/no-extraneous-dependencies

 ....

✖ 8 problems (8 errors, 0 warnings)
  6 errors and 0 warnings potentially fixable with the `--fix` option.
```

FormatやいくつかのLintエラーは自動で修正が可能です。

`npm run lint:fix`を実行してみましょう。

```console
$ npm run lint:fix

> todo-app-react@1.0.0 lint:fix /dev/github.com/toshi-toma/todo-app-react
> eslint --ext .ts,.tsx src/ --fix


/dev/github.com/toshi-toma/todo-app-react/src/stories/index.stories.tsx
  3:1  error  '@storybook/react' should be listed in the project's dependencies, not devDependencies          import/no-extraneous-dependencies
  4:1  error  '@storybook/addon-actions' should be listed in the project's dependencies, not devDependencies  import/no-extraneous-dependencies

✖ 2 problems (2 errors, 0 warnings)
```

自動で修正されました🎉

`... should be listed in the project's dependencies, not devDependencies  import/no-extraneous-dependencies`とエラーがでています。

ESLintの設定は必要に応じて調整しましょう。

以下のように設定を変更することで、エラーが直るはずです。

.eslintrc.js

```js
module.exports = {
  extends: "airbnb-typescript-prettier",
  rules: {
    "import/no-extraneous-dependencies": ["error", {
      devDependencies: ["**/*.stories.tsx"],
      peerDependencies: false
    }]
  }
};
```

## Step6 VSCodeでファイル保存時に自動でフォーマットする

エディタにVSCodeを利用している人を想定して、ファイル保存時にフォーマットエラーなどの自動修正を行えるようにします。

まず、VSCodeのExtensionsで[VSCode ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)をインストールしておきましょう。

あとは、以下のように`.vscode/settings.json`を修正します。

```json
{
  "eslint.autoFixOnSave": true,
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    { "language": "typescript", "autoFix": true },
    { "language": "typescriptreact", "autoFix": true }
  ]
}
```

# Webアプリの開発(追記)

前の章では、React、styled-components、TypeScript、ESLint、Prettierを使ってコンポーネントを開発して、それをStorybook上で表示するということを行いました。しかし、実際にWebアプリケーションを開発する際は、HTMLファイルを用意して、そこからJavaScriptファイルを読み込ませることになると思います。

この章では、そのために必要なライブラリや設定について紹介します。

## Step0 index.htmlとindex.tsxの作成

まず事前準備として、ブラウザに読み込ませるHTMLファイルとアプリケーションの起点となるindex.tsxファイルを作成しておきましょう。


`public/index.html`

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>todo-app-react</title>
</head>

<body>
  <div id="root"></div>
  <script src="bundle.js"></script>
</body>

</html>
```

`src/index.tsx`

```tsx
import React from "react";
import ReactDOM from "react-dom";

// 実際は、ここでアプリケーションコンポーネントを読み込ませる
ReactDOM.render(<h1>ToDo App</h1>, document.getElementById("root"));

```

## Step1 webpackで必要なJavaScriptファイルを生成する

[webpack](https://webpack.js.org)を利用して、`index.html`に読み込ませるJavaScriptファイル`bundle.js`を生成しましょう。

必要なライブラリをインストールしましょう。

```console
$ npm i -D webpack webpack-cli
```

次に、`webpack.config.js`を作成して、必要な設定を書きましょう。

`webpack.config.js`

```js

const path = require("path");

module.exports = {
  entry: path.resolve(__dirname, "src/index.tsx"),

  output: {
    path: path.resolve(__dirname, "public"),
    filename: "bundle.js"
  },

  module: {
    rules: [{ test: /\.tsx?$/, loader: "ts-loader" }]
  },

  resolve: {
    extensions: [".ts", ".tsx", ".js"]
  }
};

```

※ ここで`ts-loader`が必要ですが、Storybookの環境構築時にインストールされているはずなので、割愛します。

あとは、`bundle.js`を生成するnpm scriptsを追加して、実行します。

```diff
 "scripts": {
+    "build": "webpack --mode production",
```

```console
$ npm run build
```

`public`ディレクトリに、`bundle.js`というファイルが生成されました。

`public/index.html`を開くと、`bundle.js`が読み込まれたページが表示されます。

## Step2 webpack-dev-serverで開発用サーバーを立てて開発する

TypeScriptのファイルを変更するたびに、buildコマンドを実行するのは面倒なので、開発中はローカルに開発用サーバーをたてて、変更時にブラウザを自動でリロードできると楽です。ここでは`webpack-dev-server`を利用して実現します。

ライブラリをインストールして、`webpack.config.js`に設定を追記します。

```console
$ npm i -D webpack-dev-server
```

`webpack.config.js`

```diff
const path = require("path");

module.exports = {
  entry: path.resolve(__dirname, "src/index.tsx"),

  ...

  resolve: {
    extensions: [".ts", ".tsx", ".js"]
- }
+ },
+ devServer: {
+   contentBase: path.resolve(__dirname, "public")
+ }

};

```

あとは、開発用サーバーを起動するnpm scriptsを追加します。

```diff
"scripts": {
+   "start": "webpack-dev-server --mode development --open --devtool cheap-module-source-map",
```

`npm start`を実行すると、開発用サーバーが起動して、`public/index.html`が表示され、`src/index.tsx`を起点としたファイルが読み込まれます。

# さいごに

これで

- React+styled-components+TypeScriptでコンポーネントを開発
- 作成したコンポーネントはStorybookで動作確認
- 適宜ESLintやPrettierを利用してコーディングを行い、ファイル保存時に自動で修正
- webpackやwebpack-dev-serverを利用してWebアプリの開発

が行えるようになり、環境構築終了です🎉

# 最終的なpackage.json

この記事における最終的なpackage.jsonを貼っておきます。

```json

{
  "name": "todo-app-react",
  "version": "1.0.0",
  "description": "A Todo Application  built with React, styled-components, TypeScript, ESLint, Prettier, and Storybook",
  "main": "index.js",
  "scripts": {
    "start": "webpack-dev-server --mode development --open --devtool cheap-module-source-map",
    "build": "webpack --mode production",
    "lint": "eslint --ext .ts,.tsx src/",
    "lint:fix": "eslint --ext .ts,.tsx src/ --fix",
    "tsc": "tsc --noEmit",
    "storybook": "start-storybook -p 6006",
    "build-storybook": "build-storybook"
  },
  "repository": {
    "type": "git",
    "url": "git+ssh://git@github.com/toshi-toma/todo-app-react.git"
  },
  "keywords": [],
  "author": "",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/toshi-toma/todo-app-react/issues"
  },
  "homepage": "https://github.com/toshi-toma/todo-app-react#readme",
  "dependencies": {
    "react": "^16.9.0",
    "react-dom": "^16.9.0",
    "styled-components": "^4.3.2"
  },
  "devDependencies": {
    "@babel/core": "^7.5.5",
    "@storybook/addon-actions": "^5.1.10",
    "@storybook/addon-links": "^5.1.10",
    "@storybook/addons": "^5.1.10",
    "@storybook/preset-typescript": "^1.1.0",
    "@storybook/react": "^5.1.10",
    "@types/react": "^16.9.1",
    "@types/react-dom": "^16.8.5",
    "@types/storybook__addon-actions": "^3.4.3",
    "@types/storybook__react": "^4.0.2",
    "@types/styled-components": "^4.1.18",
    "@types/webpack": "^4.32.1",
    "babel-loader": "^8.0.6",
    "eslint": "^5.16.0",
    "eslint-config-airbnb-typescript-prettier": "^1.2.1",
    "prettier": "^1.18.2",
    "react-docgen-typescript-loader": "^3.1.1",
    "ts-loader": "^6.0.4",
    "typescript": "^3.5.3",
    "webpack": "^4.39.2",
    "webpack-cli": "^3.3.7",
    "webpack-dev-server": "^3.8.0"
  }
}


```
