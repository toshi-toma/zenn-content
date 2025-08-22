---
title: jscodeshift with TypeScript
tags: jscodeshift TypeScript JavaScript AST
author: toshi-toma
slide: false
---


# はじめに

以前、[jscodeshit](https://github.com/facebook/jscodeshift)という `JavaScript`および`TypeScript`コードのリファクタリングツールの紹介記事を書きました。

[JavaScriptのリファクタリングツール「jscodeshift」の使い方](https://qiita.com/toshi-toma/items/93f1dfdf0a820fe6fccc)

このツールは、ASTベースでコードを自在に変換でき、とても便利です。`jscodeshift`は、Transform File（変換処理を記述するファイル）として変換処理を書くのですが、これまで`JavaScript`で変換処理を書くことが多かったです。

しかし、Transform Fileは`TypeScript`で書くことが可能です。

最近、`TypeScript`で変換処理を書いた際、その開発体験がとても良かったので紹介します。ASTベースのコーディングに`TypeScript`の型推論や補完があると、開発が快適になることは想像しやすいでしょう。

この記事で出てくるコードやファイルは以下のリポジトリで確認できます。

[jscodeshift-with-typescript](https://github.com/toshi-toma-sandbox/jscodeshift-with-typescript)

# セットアップ

まずは、`jscodeshift`のTransform Fileを`TypeScript`で書く準備をします。

## ライブラリのインストール

必要なライブラリをインストールします。`typescript`の他に、`@types/jscodeshift`をインストールします。

<!-- @types/nodeもいるかも -->

```bash
$ npm i -D jscodeshift typescript @types/jscodeshift
```

## tsconfig.json

`tsconfig.json`を作成します。設定はお好みで調整してください。

```bash
$ npx tsc --init
```

# TypeScriptで変換処理を書く

セットアップが終わったら、`TypeScript`でTransform Fileを作成します。

## jscodeshift + TypeScriptの土台となる記述

`jscodeshift` + `TypeScript`でのコーディングの土台となる記述を書きます。
型推論により、型注釈を消すことはできますが、説明の便宜上書いています。

```typescript
import { Transform, FileInfo, API } from "jscodeshift";

const transform: Transform = ({ source }: FileInfo, { jscodeshift }: API) => {
  const j = jscodeshift;
  return j(source).toSource();
};

export default transform;
```

あとは、補完が効くので、快適に開発を進めていくことができます。

## 変換処理 `foo.bar()をfoo.baz()に変換`

前回の記事でもシンプルな例として出した、`foo.bar()をfoo.baz()に変換`処理を書いていきます。

といっても、主に利用する関数や値は型推論があるので、JavaScriptの時と比べても記述量は変わりません。規模が大きくなり、変換処理を関数として切り出す際などに型注釈を付与することになるでしょう。

### 対象のNodeを見つける

```typescript

  // ...
  const j = jscodeshift;
  return j(source)
    .find(j.CallExpression, {
      callee: {
        object: { name: "foo" },
        property: { name: "bar" },
      },
    })
  // ...
};

export default transform;
```

`find`関数にわたすAST NodeのTypeを選択する際も、エディターでの補完の恩恵を得ることができ、非常に便利です。

<img "alt="NodeのTypeの補完が表示される" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/107472/aac85b1d-4c04-a8b5-6568-c11c4571e994.png">


### Nodeを置き換える

```typescript

    // ...
    .find(j.CallExpression, {
      callee: {
        object: { name: "foo" },
        property: { name: "bar" },
      },
    })
    .replaceWith((path) => {
      // TODO
    })
    // ....
};
```

別のAST Nodeへ置き換える`replaceWith`関数に渡ってくる`path`オブジェクトの型も推論されます。

<img "alt="path`オブジェクトの補完が表示される" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/107472/7437c7f4-8831-b173-c8d1-11578d93ed21.png">


### 新しいNodeの作成

```typescript
  const j = jscodeshift;
  // ...
    .replaceWith((path) => {
      return j.callExpression(
        j.memberExpression(j.identifier("foo"), j.identifier("baz")),
        path.value.arguments
      );
    })
  // ...
};

export default transform;


```

置き換えるAST Nodeを作成する際に、利用する便利な`Builder`メソッド(e.g. `j.memberExpression`)へ渡すことが可能な値も検査してくれます。
`JavaScript`のときは、このあたりが特に苦労したので、とても助かります。

<img "alt="Builderメソッドの型情報が表示される" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/107472/3d7d47b4-baa4-0a99-fc7d-b1ab99090775.png">

## 成果物

最終的には以下のようなコードになりました。

`transform.ts`

```typescript
import { Transform, FileInfo, API } from "jscodeshift";

const transform: Transform = ({ source }: FileInfo, { jscodeshift }: API) => {
  const j = jscodeshift;
  return j(source)
    .find(j.CallExpression, {
      callee: {
        object: { name: "foo" },
        property: { name: "bar" },
      },
    })
    .replaceWith((path) => {
      return j.callExpression(
        j.memberExpression(j.identifier("foo"), j.identifier("baz")),
        path.value.arguments
      );
    })
    .toSource();
};

export default transform;

```

## 動作確認

`jscodeshift`コマンドにTransform Fileと対象ファイルを渡して動作確認しましょう。
`-p`はprintです。`-d`はdry runです。
また、対象ファイルとして`TypeScript`ファイルを指定する場合は、`--parser`オプションで`ts`を指定します。（`tsx`も指定可能です）

```bash
$ npx jscodeshift -t transform.ts index.ts --parser ts -p -d
Processing 1 files...
Spawning 1 workers...
Running in dry mode, no files will be written!
Sending 1 files to free worker...
const foo: any = {};

foo.baz();

All done.
Results:
0 errors
0 unmodified
0 skipped
1 ok
Time elapsed: 0.855seconds
```

# テスト

最後に、テストについても触れておきます。

`jscodeshift`のテストについては以下の記事を見ると良いです。

[jscodeshiftのテストを書く](https://qiita.com/pirosikick/items/bf0555bc8295386c1023)

今回は`jscodeshift`のテストおよびテストのfixture fileに`TypeScript`ファイルを使う方法を紹介します。

## ライブラリのインストール

`jest`と`ts-jest`をインストールします。

```bash
$ npm i -D jest ts-jest
```

## jestのセットアップ

`jest.config.js`の`preset`で`ts-jest`を指定します。

```javascript

module.exports = {
  preset: "ts-jest",
};
```

## fixture fileとテストファイルの作成

`jscodesfhit`のドキュメントで指定されているようなディレクトリ構造でfixture fileとテストファイルを作成します。

[jscodeshift | Unit Testing](https://github.com/facebook/jscodeshift#unit-testing)

今回は、`JavaScript`ファイルと`TypeScript`ファイルのfixture fileを配置します。

```
/transform.ts
/__tests__/transform.test.js
/__testfixtures__/foobar.input.ts
/__testfixtures__/foobar.output.ts
/__testfixtures__/foobar.input.js
/__testfixtures__/foobar.output.js
```

`__tests__/transform.test.js` を以下のように記述しました。
`TypeScript`ファイルを変換する場合は、`parser`オプションを指定する必要があるので、`defineTest`関数で指定しています。


```typescript
// @ts-ignore
import { defineTest } from "jscodeshift/dist/testUtils";

describe("test with .js file", () => {
  defineTest(__dirname, "transform", null, "foobar");
});

describe("test with .ts file", () => {
  defineTest(__dirname, "transform", null, "foobar", { parser: "ts" });
});

```

```bash
$ npx jest
> jest

 PASS  __tests__/transform.ts
  test with .js file
    transform
      ✓ transforms correctly using "foobar" data (129 ms)
  test with .ts file
    transform
      ✓ transforms correctly using "foobar" data (10 ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.14 s
```


# まとめ

`jscodeshift`のTransform Fileを`TypeScript`で書く方法と、`TypeScript`ファイルでのテストの方法を紹介しました。

ASTベースのプログラミングは、型の恩恵を強く受けることができ、`TypeScript`との相性が非常に良いです。

今後は、`jscodeshift`を使う際は、`TypeScript`で書くのが良さそうです。
