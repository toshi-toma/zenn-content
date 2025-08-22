---
title: JavaScriptのリファクタリングツール「jscodeshift」の使い方
tags: JavaScript AST jscodeshift
author: toshi-toma
slide: false
---
# はじめに
JavaScriptのコードを一括で変換したり修正したい場合、正規表現などを使い置換しますか？
シンプルなケースであればそれでも問題ないですが、複雑な変換であればASTベースでコードを自在に変換できる**「[jscodeshift](https://github.com/facebook/jscodeshift)」**が便利です。

`jscodeshift`を利用すると、以下のようなことができます。

## 例) `function`で書かれた関数をアロー関数に一括で変換

`target/arrow-function/index.js`

```js
const fn = function() {
  console.log("foo");
}.bind(this);

[1, 2, 3].map(function(v) {
  return v * v;
});

Promise.resolve()
  .then(function() {
    console.log("foo");
  })
  .then(function() {
    return 1;
  });
```

↓

```console
$ npx jscodeshift -t transforms/arrow-function.js target/arrow-function/index.js
```

[transforms/arrow-function.jsは事前に準備](https://github.com/cpojer/js-codemod/blob/master/transforms/arrow-function.js)

↓

`target/arrow-function/index.js`

```js
const fn = () => {
  console.log("foo");
};

[1, 2, 3].map(v => v * v);

Promise.resolve()
  .then(() => {
    console.log("foo");
  })
  .then(() => 1);
```

コマンドを実行しただけで、コードが一括で変換されました。

この記事では`jscodeshift`の紹介から、基本的な使い方までを紹介します。ASTの入門としても良いと思います。

## コード変換の必要性

フロントエンド開発の中でコードを新しい書き方に変換したいケースに遭遇することは多々あると思います。

例えば

- JavaScriptの新しい構文の登場で、シンプルな記述に
- 不要になった記述を削除
- 新しいライブラリのAPIを使う
- ライブラリのブレイキングチェンジによるコードの修正

などなど、様々なケースがあり、これらを手動で行ったり、正規表現を使った検索や置換では限界があります。また、現状のコードが複数のパターンある場合はそれぞれの正規表現を作成して、別々に適用するのも手間です。

そんなときに`jscodeshift`などのASTベースのツールを使うことで、難しい変換処理をプログラマブルに簡単に行うことができます。

# ASTについて

コードをプログラマブルに変換したい場合、ASTを使うことが多いと思います。

ASTはフロントエンド開発に触れる中で身近な存在になっています。例えば`Babel`、`Prettier`、`ESLint`、`TypeScript`などはASTをベースに作られています。

ASTとは、Abstract Syntax Treeの略で、日本語にすると`抽象構文木`です。

ソースコードを解析し、それを木構造で表現したものです。

例えば`var a = 0`というコードは以下のような木構造で表現されます。

<img width="947" alt="スクリーンショット 2019-11-28 17.11.34.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/107472/7cf2597b-a85d-96d4-ada6-4d8309f7e010.png">

そしてJSONであれば以下のように表現されます。

<img width="947" alt="スクリーンショット 2019-11-28 17.13.10.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/107472/b1965f5e-390b-6a30-f370-d0aaf6661e9a.png">

つまりJavaScriptの世界においてはただのオブジェクト(JSON)なので、それをプログラマブルに扱い、変換することが可能になります。

どのようなASTが生成されるかは、[AST Explorer](https://astexplorer.net/)を使うことで簡単に見れます。是非手元のコードで試してみてください。

## Babel Plugin

JavaScriptコードを変換する代表例として[BabelのPlugin](https://babeljs.io/docs/en/plugins/)を作るというアプローチもあります。
今日紹介する`jscodeshift`とできることは基本的に同じですが、BabelのPluginはビルドプロセスでコンパイル時に毎回適用したいケースに向いています。例えばReactのJSXをJSに変換するのはBabelのPluginを使うことが多いです。

特定のタイミングで1回だけ実行して終わり。というユースケースでは`jscodeshift`を使う方が簡単だと思います。

それでは、ここからは`jscodeshift`について深ぼっていきます。

# jscodeshiftとは

`jscodeshift`とはFacebookが開発しているOSSで、複数のJavaScriptまたはTypeScriptのファイルに対して、ASTによる操作を簡単に行えるツールです。

- https://github.com/facebook/jscodeshift
- https://www.npmjs.com/package/jscodeshift

ASTの操作などを行う[recast](https://github.com/benjamn/recast)というライブラリをラップしたAPIを提供しています。また`recast`が依存している[ast-types](https://github.com/benjamn/ast-types)についても知っておく必要があります。

利用者は、変換処理が記述された自作またはOSSの`transform file`を用意して、`jscodeshift`コマンドで変換対象のコードを指定し、リファクタリングを実行する流れになります。

例) 

```console
$ npx jscodeshift -t transform.js src
```

OSSとして様々な`transform file`が提供されています。

- https://github.com/cpojer/js-codemod
- https://github.com/reactjs/react-codemod

## jscodeshiftを使ってみる

自分で`transform file`を作成する前に、OSSとして用意されているものを使ってみるところから始めましょう。今回は`js-codemod`の`js-codemod/transforms/no-vars.js`を使います。

これは、`var`を`const`または`let`に書き換えてくれる`transform file`です。
`js-codemod`はnpmにも公開されているので、それを使ってもいいのですが最新版がpublishされてないのでリポジトリをcloneします。

```console
git clone https://github.com/cpojer/js-codemod.git
```

それでは`var`で書かれたコードを手元に用意しましょう。

`index.js`

```js
for (var i = 0; i < 10; i++) {

}

var letItBe;
var shouldBeLet = 0;
var shouldBeConst = 0;

function mutate() {
  shouldBeLet = 1;
}

var whileIterator = 10;
while (whileIterator > 0) {
  whileIterator--;
}
```

あとは、`jscodeshift`を使って変換を適用します。

```console
$ npx jscodeshift -t js-codemod/transforms/no-vars.js index.js
```

見事に変換されました。

```js
for (let i = 0; i < 10; i++) {

}

let letItBe;
let shouldBeLet = 0;
const shouldBeConst = 0;

function mutate() {
  shouldBeLet = 1;
}

let whileIterator = 10;
while (whileIterator > 0) {
  whileIterator--;
}
```

# 自分で変換処理を書く

`jscodeshift`を活用していく際は、`transform file`をプロジェクトや要件に合わせて自分で作成していくことになると思います。本記事では`transform file`をJavaScriptファイルとして作成しますが、TypeScriptでも書くことが可能です。

## transform file

`transform file`は以下のように最大で3つの引数を受け取り、ソースコードを文字列として返す関数をモジュールとして作成するだけです

```js
module.exports = function(fileInfo, api, options) {
  // `fileInfo.source`をもとにASTでの変換操作を行う
  // ...
  // 変換操作した結果を返す
  return source;
};
```

`fileInfo`

| Property       |  Description |
|:-----------------|:------------------|
| path             | ファイルのパス(string) |
| source           | ソースコード(string) |

`api`

| Property       |  Description |
|:-----------------|:------------------|
| jscodeshift   | jscodeshiftが提供するAPIの参照 |
| j           | 同上 |
| stats   | --dry実行中にデータを収集するヘルパー |
| report | 渡された文字列を標準出力に出力する関数 |

`options`

全てのオプション情報が含まれます。ここで、プロジェクト固有の値も渡すことができます。

例)

```console
$ jscodeshift -t transform.js src --foo=bar
```

`options`は`{foo: 'bar'}`を含んだオブジェクトになります。

`戻り値`

基本的にはソースコードを文字列として返しますが、戻り値に応じて変換のステータスが決められます。

- 入力と違う文字列を返す: OK
- 何も返さない: skipped
- 入力と同じ文字列を返す: unmodified
- エラー発生: error

`jscodeshift`はこれらの結果に従って変換のステータスを処理します。


## foo.bar()をfoo.baz()に変換
今回は「foo.bar()をfoo.baz()に変換」というシンプルな例を題材に自分で`transform file`を作成してみます。

### 対象のNodeを見つける

まずは、`foo.bar()`を検出します。検出したいコードがどんなAST NodeなのかはAST Explorerで確認します。

`jscodeshift`関数に`source`を渡すと[Collection](https://github.com/facebook/jscodeshift/wiki/jscodeshift-Documentation#collections)オブジェクトが返ってきます。`Collection`が持つ`find`、`filter `、`map`、`forEach`などの汎用的なAPIを利用することができます。
特定のNodeを取得する場合は`find`関数が便利です。また、第2引数で具体的な条件も指定することができます。

今回は`CallExpression`が対象なので、以下のようになります。

```js
module.exports = function(fileInfo, { jscodeshift }, options) {
  const j = jscodeshift;
  const root = jscodeshift(fileInfo.source);
  root
    .find(j.CallExpression, {
      callee: {
        object: { name: "foo" },
        property: { name: "bar" }
      }
    })
};
```

### Nodeを置き換える

次は、`foo.baz()`への変換を行います。`find`関数で検出できたNodeに対して変換処理を記述します。

`forEach`関数のコールバックの第一引数のpathは、実際のASTをラップして`parent`などの追加情報を持つ[NodePath](https://github.com/facebook/jscodeshift/wiki/jscodeshift-Documentation#nodepaths)です。詳しい詳細は[ast-types](https://github.com/benjamn/ast-types#nodepath)を参照してください。

変換については、今回`replaceWith`関数で別のNodeに置き換えます。

```js
module.exports = function(fileInfo, { jscodeshift }, options) {
  const j = jscodeshift;
  // ...
    .forEach(path => {
      j(path).replaceWith(
        // ...
      );
    });
};
```

### 新しいNodeの作成

`replaceWith`関数に渡すNodeは、ASTノードを簡単に作成できる[ast-typesのBuilderメソッド](https://github.com/benjamn/ast-types/blob/master/gen/builders.ts)を利用します。`jscodeshift`経由で利用しましょう。メソッドはNodeのTypeを小文字始まりにしたものです。

```js
module.exports = function(fileInfo, { jscodeshift }, options) {
  const j = jscodeshift;
  // ...
        j.callExpression(
          j.memberExpression(j.identifier("foo"), j.identifier("baz")),
          path.value.arguments
        )
  // ...
};
```


### 成果物

最終的には以下のようなコードになりました。

`transform.js`

```js
module.exports = function(fileInfo, { jscodeshift }, options) {
  const j = jscodeshift;
  const root = jscodeshift(fileInfo.source);

  root
    .find(j.CallExpression, {
      callee: {
        object: { name: "foo" },
        property: { name: "bar" }
      }
    })
    .forEach(path => {
      j(path).replaceWith(
        j.callExpression(
          j.memberExpression(j.identifier("foo"), j.identifier("baz")),
          path.value.arguments
        )
      );
    });
  return root.toSource();
};
```

今回の例では、正規表現で書いたほうが簡単でしたが、自分で`transform file`を作成してプログラマブルに変換する流れが分かったと思うので、次は少し複雑な変換を行います。

## Object.assign()をスプレッド構文で置き換える

最後に、Object.assignで行っていたオブジェクトのコピーやマージをスプレッド構文で書き換えます。

[js-codemod](https://github.com/cpojer/js-codemod)の[rm-object-assign](https://github.com/cpojer/js-codemod/blob/master/transforms/rm-object-assign.js)を参考にして実装していきます。

```js
Object.assign({}, a);

Object.assign({ a: 1 }, b, { c: 3 });
```

上記のコードがスプレッド構文を使った下記のコードになるイメージです。

```js
({
  ...a,
});

({
  a: 1,
  ...b,
  c: 3,
});
```

### 対象のNodeを見つける

まずは変換対象のNodeを`find`関数で取得しましょう。

AST Explorerで以下のコードを見て、どんなASTか確認すると簡単ですね。

```js
Object.assign({}, a);
```

<img width="636" alt="スクリーンショット 2019-11-30 19.21.53.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/107472/0fb83c69-d4c2-0d56-b927-bd33b941988b.png">

```js
module.exports = function(fileInfo, { jscodeshift }, options) {
  const j = jscodeshift;
  const root = jscodeshift(fileInfo.source);

  root
    .find(j.CallExpression, {
      callee: { object: { name: "Object" }, property: { name: "assign" } },
      arguments: [{ type: "ObjectExpression" }]
    })
};
```

※ `Object.assign({}, ...b);`など、`SpreadElement`を引数に渡しているコードだとNodeが持つ情報が違うので[サンプルコードではfilter処理](https://github.com/cpojer/js-codemod/blob/0677c0b6029031e4f71bbbfc406a0465e6f7acd2/transforms/rm-object-assign.js#L33)をしていますが今回は割愛します

### スプレッド構文で置き換える

あとは、取得したCallExpressionをスプレッド構文に置き換えれば完成です。

取得できた各Nodeに対して特定の変換処理を実装していきます。今回は`rmObjectAssignCall`関数を実装していきます。

```js
module.exports = function(fileInfo, { jscodeshift }, options) {
  // ...
  const rmObjectAssignCall = path =>
    j(path).replaceWith();

  root
    .find(j.CallExpression, {
      callee: { object: { name: "Object" }, property: { name: "assign" } },
      arguments: [{ type: "ObjectExpression" }]
    })
    .forEach(rmObjectAssignCall);

  return root.toSource();
};
```

スプレッド構文のコードをAST Explorerで見ると、どんなNodeに置き換えればいいか分かりますね。

```js
({
  a: 1,
  ...b
});
```
<img width="575" alt="スクリーンショット 2019-12-01 10.59.55.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/107472/4e8366bd-bac8-9d0c-ad76-e56a705ecffc.png">


`ObjectExpression`を作成していきます。
AST Nodeを作成するBuilderに何を渡すといいのかはドキュメントがないので、[ast-typesのコード](https://github.com/benjamn/ast-types/blob/c208a17d7eeef6a9676990d364f9236e8b7e512f/gen/builders.ts#L398-L409)を見て確認しましょう。


```js
module.exports = function(fileInfo, { jscodeshift }, options) {
  // ... 
  const rmObjectAssignCall = path => {
    const properties = path.value.arguments.reduce(
      (allProperties, { ...argument }) => {
        if (argument.type === "ObjectExpression") {
          const { properties } = argument;
          return [...allProperties, ...properties];
        }

        return [...allProperties, { ...j.spreadProperty(argument) }];
      },
      []
    );
    return j(path).replaceWith(j.objectExpression(properties));
  };
  // ...
};
```

### 成果物

最終的には以下のようなコードになりました。

`transform.js`

```js
module.exports = function(fileInfo, { jscodeshift }, options) {
  const j = jscodeshift;
  const root = jscodeshift(fileInfo.source);

  const rmObjectAssignCall = path => {
    const properties = path.value.arguments.reduce(
      (allProperties, {  ...argument }) => {
        if (argument.type === "ObjectExpression") {
          const { properties } = argument;
          return [...allProperties, ...properties];
        }

        return [...allProperties, { ...j.spreadProperty(argument) }];
      },
      []
    );
    return j(path).replaceWith(j.objectExpression(properties));
  };

  root
    .find(j.CallExpression, {
      callee: { object: { name: "Object" }, property: { name: "assign" } },
      arguments: [{ type: "ObjectExpression" }]
    })
    .forEach(rmObjectAssignCall);

  return root.toSource();
};
```

# まとめ

- JavaScriptのコードをリファクタリングしたいときに便利なASTベースのツール`jscodeshift`について紹介しました
- OSSとして公開されている`transform file`を利用したり、自作して活用していきます
- 自作する時は、AST Explorerを使っていきます
- ドキュメントがあまり用意されていないので、TypeScriptの型定義を都度参照していくのが良さそうです
    - [@types/jscodeshift](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/jscodeshift)
    - [recast](https://github.com/benjamn/recast)
    - [ast-types](https://github.com/benjamn/ast-types)

# テスト
今回は`jscodeshift`のテストについては触れられませんでした。
同僚の[@pirosikick](https://twitter.com/pirosikick)がテストについて書いてくれたので、是非こちらも合わせて読んでください。

[jscodeshiftのテストを書く - Qiita](https://qiita.com/pirosikick/items/bf0555bc8295386c1023)

# 参考リンク
- https://github.com/facebook/jscodeshift
- https://github.com/facebook/jscodeshift/wiki/jscodeshift-Documentation
- https://github.com/sejoker/awesome-jscodeshift
- https://github.com/cpojer/js-codemod
- https://github.com/reactjs/react-codemod
- https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/jscodeshift
- https://github.com/benjamn/ast-types
- https://github.com/benjamn/recast
