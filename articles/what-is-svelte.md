---
title: "Svelteとは"
emoji: "‍💍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["svelte", "javascript"]
published: true
---

# はじめに

最近、フロントエンドでたまに聞く、Svelteというライブラリについての紹介記事です。

https://svelte.dev/

ReactやVue.jsのような宣言的にUIを記述できるので、簡単にWebアプリケーションやUIを作れる。かつ、バンドルサイズやランタイムが小さく、高いパフォーマンスが期待できるライブラリです。

まだコミュニティやエコシステムが成熟しているわけではないので、不都合はありますが、小さなアプリケーション開発のようなケースでは活躍するのではないかと思っています。

---

Svelteで作成したToDoアプリのサンプルプロジェクトを以下のリポジトリで確認できます。

https://github.com/toshi-toma/svelte-todoapp

# Svelteとは

## Svelteはコンパイラ

「Svelteはコンパイラです」と説明されることが多いので、立ち位置が難しいですが、ReactやVue.jsではなくSvelteを採用する。そういうイメージです。

なので、ライブラリとしての立ち位置としては、WebアプリケーションやUIを構築するためのライブラリです。

しかし、ReactやVue.jsとは違い、SvelteはUIライブラリではなく、コンパイラです。

宣言的なコードを、命令的なPureなJavaScriptコードに変換するコンパイラです。

## 概要

Vue.jsの単一ファイルコンポーネント(SFC)のようなファイル構成で、`.svelte`ファイルの中にコンポーネントとしてHTML, CSS, JavaScript相当のコードを記述します。ReactやVue.jsのようにコンポーネントを組み合わせてWebアプリケーションを構築します。

コンパイルすると、ライブラリのコードがほぼ無い状態のPureなJavaScriptファイルとCSSファイルが生成されます。
あとは、生成されたJSファイルとCSSファイルをHTMLを通してブラウザに読み込むだけです。

アプリケーションを構築する際に必要な記述量は、他のライブラリに比べると少ないです。
ライブラリのコードがほぼ無く、コンパイル後のファイルサイズも小さいので、高いパフォーマンスが期待できます。

Reactのように比較的大きいライブラリを読み込むほどではなく、小さいアプリをさくっと作るようなケースでは非常に活躍しそうですね。

## 最近話題のSvelte

[The State of JavaScript 2019](https://2019.stateofjs.com/)でも紹介されており、最近のFront-end-Frameworkで注目されていることが分かります。

- https://2019.stateofjs.com/front-end-frameworks/
- [https://2019.stateofjs.com/awards/ | Prediction Award](https://2019.stateofjs.com/awards/)

公式ドキュメントのトップページに「Who's using Svelte?」が書いてあるので、見ると面白いです。
代表的なのは、作者が所属している、[New York Times](https://www.nytimes.com/)です。

# Svelteの記述例

`.svelte`という拡張子のファイルにコンポーネントを記述し、1つのファイルにHTML/CSS/JavaScriptにあたるコードを記述します。
このコンポーネントを組み合わせてアプリケーションを構築します。

コンポーネント指向という意味ではReactやVue.jsと似た感覚で開発を進めることができます。
コンポーネントを作って、コンポーネントにpropsとしてデータやハンドラーを渡すイメージです。


`Counter.svelte`

```js

<script>
  let count = 0;
  const increment = () => {
    count += 1;
  }
</script>

<style>
  h1 {
    color: purple;
    font-family: 'Comic Sans MS', cursive;
    font-size: 2em;
  }
</style>

<h1>Count: {count}!</h1>
<button on:click={increment}> + </button>
```

# コンパイルで生成されるコード

```js
<script>
  let name = "World!!";
</script>

<style>
  h1 {
    color: tomato;
  }
</style>

<main>
  <h1>Hello {name}!</h1>
</main>
```

この`App.svelte`をコンパイルすると、以下のようにJavaScriptファイルとCSSファイルが出力されます。サンプルとして置いているbuildディレクトリのコードは、見やすいようにminifyを外しています。

- JavaScriptファイル(例: bundle.js)
  - [sample bundle.js](https://github.com/toshi-toma-sandbox/svelte-compiled-code/blob/master/public/build/bundle.js)
  - PureなJavaScript+Svelteのとても小さなランタイムコード
- CSSファイル(例: bundle.css)
  - [sample bundle.css](https://github.com/toshi-toma-sandbox/svelte-compiled-code/blob/master/public/build/bundle.css)
  - コンポーネント内で擬似的にスコープが閉じるように変換されたCSS

公式サイトのREPLで、記述したSvelteファイルの挙動や出力結果を確認できます。

[Svelte REPL](https://svelte.dev/repl/hello-world)

出力されるCSSファイルは、styleタグに記述したセレクターが`svelte-xuaa3f`のようなハッシュが付与された形で出力されます。
これにより、コンポーネントで擬似的にスタイルのスコープが閉じます。他のスタイルとの衝突など、意識することが減って便利です。
適用されないようなstyle(Unused CSS selector)はコンパイル時に削除されます。

# Svelteの特徴
公式ドキュメントには、主に3つの特徴が書かれています。

![Svelte公式サイト](https://storage.googleapis.com/zenn-user-upload/qbulbf48ifxe4mbayeh4tw1j1uon)

## Write less code（記述量が少ない）
https://svelte.dev/blog/write-less-code

Sveteは、アプリケーションの構築に必要なコードの記述量が減ることを1つの強みにしています。
全体的にシンプルな記述に加えて、Vue.jsにあるような双方向バインディングが強力な点や、stateの更新はミュータブルに行うので記述量が減っています。

同じことを実現するために必要なコード量は、ReactやVue.jsよりも少なくなります。
記事で書かれている例だと、Reactで442文字、Vue.jsで263文字、Svelteは145文字で実現できます。

ここは簡単に書けるということで「Easy」だけど、「Simple」かどうかは別の話なので、好みは分かれそうです。（ただし、小さいアプリをさくっと作るケースなら本当に良さそう）

## No virtual DOM（仮想DOMではない）
https://svelte.dev/blog/virtual-dom-is-pure-overhead

ReactやVue.jsなど仮想DOMのアプローチは、実際のDOM操作に加えて差分計算などのオーバーヘッドがあります。
Svelteは、コンパイル時にどの値が変化するかを知ることで、効率的に差分を検知してDOMに反映するコードを生成します。

全体のファイルサイズが小さくなり、ランタイムがより高速になります。
コンパイル後のコードは、必要な処理だけが入ることになるので、ムダにサイズが大きくなるということもありません。

## Truly reactive（Reactive）
https://svelte.dev/blog/svelte-3-rethinking-reactivity

「No virtual DOM」と関連しますが、Svelteはコンパイル時に「値(state)の変更を検知してDOMへ反映する」コードを生成することで効率的な処理を可能にします。
Svelteは、ある値が変更された際にそれを参照している値にも変更を反映するような、Reactiveな処理を代入によってトリガーします。
なので、配列の操作なども`numbers = [...numbers, numbers.length + 1];`, 値の更新も`number +- 1;`のように代入を使います。

コンパイル結果のコードでは、値が変更されるような代入式を、`$$invalidate(0, (count += 1));`と表現します。
これでコンポーネントの値の変更を検知してDOMへ反映します。

また、「値が変わったらその値を使って計算し直したり、特定の処理をする」ようなReactiveな処理は`$: b = a + 1;`のように書きます。
逆に、 コンポーネントのscriptタグのトップレベルで`b = a + 1;`とだけ書いた場合、`a`が変更されても`b`は変更されません。

ここはReactのコンポーネントとは大きく違うので、最初ハマりました。

```js
<script>
  let count = 0;

  console.log(`the count is ${count}`);  // 初回描画時にしか実行されない

  function handleClick() {
    count += 1;
  }
  $: {
    console.log(`the count is ${count}`);  // countが変わるたびに実行される
    alert(`I SAID THE COUNT IS ${count}`);
  }
</script>

<button on:click={handleClick}>
  Clicked {count} {count === 1 ? 'time' : 'times'}
</button>
```

## その他
その他にも、コンパイルレベルでいろいろな恩恵を受けることができます。

- 組み込みでアクセシビリティチェックがある
- 未使用のCSS削除
- etc

# Svelteを使うメリットとデメリット
## メリット
- 記述量が少ない
  - 簡単にWebアプリケーションを構築できる
- 頑張らなくてもある程度は高いパフォーマンスが期待できる
  - バンドルサイズが小さい
    - コンパイル時に必要なコードだけが含まれる
    - バンドルサイズが小さいと初回の描画が早くなる
  - ランタイムが小さい
    - ライブラリのランタイムが、ないと言って良いレベル
    - もちろん規模が大きくなったり、重く非効率な処理をユーザーが書くと、それに比例して遅くはなります
- 開発の構成がシンプル
  - テンプレートプロジェクトがあり、基本的にそのままの設定でいいはず
  - PureなJavaScriptとCSSだけが出力されて、それをHTMLで読み込むだけ
  - Webアプリケーションを構築するために必要な基本的な機能はSvelteに備わってる
    - UIの構築,スタイリング,アニメーション,ライフサイクル,State管理
- 十分なDX(開発体験)
  - 基本的なWebアプリケーションを構築する上でのDXは十分良いと思いました
  - 他にもTypeScriptがサポートされている、エディター体験も良い、フォーマットもあるなど

雑に書くと簡単にWebアプリケーションが作れて、ある程度高いパフォーマンスが期待でき、それでいて十分なDXで開発できる。

## デメリット
- まだ世の中に情報や開発者が少ない
  - 記事などを調べてもあまり多くは見つからない
- ReactやVue.jsなどに比べると、エコシステムが育ってない
  - Reactなどエコシステムが育った他のライブラリにあるような高いDXと比較すると不足しているものがある

TypeScriptが公式でサポートされたことで、不足してたDXの大きな点が解消された印象はあります。

# どこで学ぶか

Svelteは公式のチュートリアルが非常に充実しているので、まずは一通りみるのをおすすめします。
量が多いので、アニメーションなど必要に応じて読み飛ばすのが良いです。

[Svelte Tutorial](https://svelte.dev/tutorial/basics)

ほかにも、[FrontendMasters](https://frontendmasters.com/courses/svelte/)にコースが用意されていたり、YouTubeにも動画がいくつかあります。

- [Let’s Learn Svelte (with Rich Harris) — Learn With Jason](https://youtu.be/ogXETl_I0Dg)
- [Svelte Society Day 2020: Rich Harris: Frequently Asked Questions](https://youtu.be/luM5uobewhA)
- [Rich Harris - Rethinking reactivity](https://youtu.be/AdNJ3fydeao)

# 開発体験

Svelteの開発体験周りを紹介します。

## 新規プロジェクト作成

新規プロジェクトのテンプレートが公式で用意されています。以下の手順で簡単にプロジェクトを作ることができます。
プロジェクト作成後、まず`App.svelte`を見るところからスタートです。

```bash
$ npx degit sveltejs/template my-svelte-project
$ cd my-svelte-project
$ npm install
$ npm run dev
```

## Svelte組み込みの仕組み
Svelteではいくつかの独自記法があります。

例えば、 Await blocksという記法があります。

https://svelte.dev/tutorial/await-blocks

```js
<script>
  let promise = getRandomNumber();
  async function getRandomNumber() {
      // ...
  }
<script>

{#await promise}
    <!-- promise is pending -->
    <p>waiting for the promise to resolve...</p>
{:then value}
    <!-- promise was fulfilled -->
    <p>The value is {value}</p>
{:catch error}
    <!-- promise was rejected -->
    <p>Something went wrong: {error.message}</p>
{/await}
```

他にも[ライフサイクル](https://svelte.dev/tutorial/onmount)、[State管理の仕組み](https://svelte.dev/tutorial/writable-stores)、[アニメーション](https://svelte.dev/tutorial/animate)、[トランジション](https://svelte.dev/tutorial/transition)などの機能がデフォルトで備わっています。

## エディター

VSCodeがもっとも良いです。

公式で提供されている拡張は以下の「Svelte for VS Code」です。

https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode

これを入れるとコードジャンプ、ホバー時の情報やエラー・警告の表示などに加えて、[Prettierを利用したフォーマット](https://github.com/sveltejs/prettier-plugin-svelte)が手に入ります。

## TypeScript

今年のリリースで、公式にTypeScriptをサポートしました！

新規でプロジェクトを作る際は、公式のテンプレートに setupスクリプトが用意されています。

詳しくは以下の記事を確認してください。

https://svelte.dev/blog/svelte-and-typescript

e.g. `svelte-ts-sample.svelte`

```ts
<script lang="ts">
  let count: number = 0;
  const increment = () => {
    count += 1;
  }
</script>

<style>
  h1 {
    color: purple;
    font-family: 'Comic Sans MS', cursive;
    font-size: 2em;
  }
</style>

<h1>Count: {count}!</h1>
<button on:click={increment}> + </button>
```

## ESLint/Prettier

公式でeslint-pluginやprettier-pluginが提供されています。

- https://github.com/sveltejs/prettier-plugin-svelte
- https://github.com/sveltejs/eslint-plugin-svelte3


## ブラウザ拡張
ReactのDeveloper Toolsのような便利なブラウザ拡張が用意されています。

これでコンポーネント名を使ったtreeが表示されます。あとはコンポーネントのpropsとstateも確認可能です。

- Chrome
  - https://chrome.google.com/webstore/detail/svelte-devtools/ckolcbmkjpjmangdbmnkpjigpkddpogn
- Firefox
  - https://addons.mozilla.org/ja/firefox/addon/svelte-devtools/

## ブラウザサポート

デフォルトでは、IE11では動作しないバージョンのJavaScriptコードを出力します。
必要に応じて、PolyfillやBabelでのトランスパイルを通す必要があります。

## その他のエコシステム

### Preprocess

https://github.com/sveltejs/svelte-preprocess

svelte-preprocessを使うことで、SassやPostCSS, Babel, TypeScriptといったライブラリと組み合わせることができます。

### Routing

- svelte-routing
  - https://github.com/EmilTholin/svelte-routing
- svelte-spa-router
  - https://github.com/ItalyPaleAle/svelte-spa-router

### Sapper
https://github.com/sveltejs/sapper

ReactでいうNext.jsの立ち位置としてSapperというフレームワークがあります。

### Elder.js (Static Site Generator)

https://elderguide.com/tech/elderjs/


### Svelte Native
https://svelte-native.technology/

### Svelte WebGL
https://github.com/sveltejs/gl
