---
title: "MUIを使う上で知っておきたいことまとめ"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mui", "react", "typescript"]
published: false
publication_name: "lincwell_inc"
---

# はじめに

この記事では、React向けUIライブラリのMUI（Material UI）を実際のプロジェクトで使う上で知っておくと便利な知識やコツについてまとめて紹介します。

MUIはReactエコシステムで人気の高いUIライブラリですが、実際に使ってみると「どうやってカスタマイズするの？」「型安全に使うには？」「パフォーマンスを考慮した使い方は？」といった疑問が出てくることがあります。

この記事では、そうした疑問を解決するための実践的な知識を幅広くカバーしています。MUIを使ったことがある方にとって、より効率的で保守性の高い開発の参考になれば幸いです。

# MUIのライブラリ構成について

まず、MUIに関連するライブラリについて整理しましょう。

## 主要なライブラリ

**Material UI**
- MUIの中核となるReact UIコンポーネントライブラリです
- ボタン、テキストフィールド、ダイアログなど、Material Designに基づいたコンポーネントを提供しています

**MUI System**
- MUIに組み込まれているスタイリング系の機能を提供するライブラリです
- sx propsやstyled関数などの機能はこちらで実装されています
- スタイリングやレイアウト系の詳細な仕様については、[MUI Systemのドキュメント](https://mui.com/system/getting-started/)を参照すると良いです

**Joy UI**
- Material UIとは異なるデザインシステムのUIコンポーネントライブラリでした
- しかし、現在は開発が停止されているため、新規プロジェクトでは選択肢から外した方が良さそうです

## FigmaのDesign Kits

デザインとの連携を重視する場合、[MUIのFigma UI Kit](https://mui.com/store/items/figma-react/)が販売されています。Figmaを使ったデザインフローを採用している場合は必要になることが多いです。

# スタイリングとカスタマイズ

MUIのコンポーネントをカスタマイズする方法はいくつかありますが、用途に応じて使い分けることが重要です。

## 基本的なスタイリング方針

基本的にはコンポーネントにsxプロパティを渡すか、BoxやStackのようなレイアウト系のコンポーネントと組み合わせることで大部分のスタイリングは対応できます。

従来のバージョンでは`mr={2}`のような形でmargin-rightを指定できるSystem Propsという仕組みがありましたが、これは非推奨になっているため、すべてsxに統一することを推奨します。

## ある箇所でだけスタイルを変更する場合

**sx propsを渡す**

最も基本的な方法です。ただし、sxはrootの要素にのみ適用されるため、コンポーネント内部の特定の要素をカスタマイズしたい場合は、CSS classを指定してカスタマイズする必要があります。

```tsx
<Button
  sx={{
    backgroundColor: 'primary.main',
    color: 'white',
    '&:hover': {
      backgroundColor: 'primary.dark',
    },
  }}
>
  カスタムボタン
</Button>
```

## カスタマイズされたコンポーネントを再定義する場合

**styled関数**

複数箇所で同じカスタマイズを適用したい場合は、styled関数を使って新しいコンポーネントを作成します。

```tsx
import { styled } from '@mui/material/styles';
import { Button } from '@mui/material';

const CustomButton = styled(Button)(({ theme }) => ({
  backgroundColor: theme.palette.primary.main,
  color: theme.palette.primary.contrastText,
  '&:hover': {
    backgroundColor: theme.palette.primary.dark,
  },
}));
```

**重要**: styled関数を使う際は、@mui/systemではなく@mui/material/stylesからimportしてください。@mui/systemを使うとThemeの型定義が利用できません。

## コンポーネントのベースとなるスタイルを変更する場合

**themeの設定**

アプリ全体で共通のカスタマイズを適用したい場合は、themeで設定します。

# themeの拡張とカスタマイズ

デフォルトのテーマがありますが、そのまま使うことは少ないと思うため、基本的にはカスタマイズすることになります。

[デフォルトテーマ](https://mui.com/material-ui/customization/default-theme/)の内容を確認して、どの部分をカスタマイズするかを決めましょう。

## createThemeを使ったカスタマイズ

createThemeにカスタマイズする内容を入れることで、テーマを拡張できます。

```tsx
import { createTheme } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
  },
  typography: {
    h1: {
      fontSize: '2.5rem',
      fontWeight: 600,
    },
    body3: {
      fontSize: '0.75rem',
      lineHeight: 1.2,
    },
  },
});
```

## typographyの追加

デフォルトにはないtypographyのvariantを追加することができます。上記の例では`body3`を追加していますが、これを使うためには型定義の拡張も必要です。

## 色の追加および色の名前空間

独自のカラーパレットを追加することで、ブランドカラーなどを統一的に管理できます。

## コンポーネントのデフォルトスタイルやpropsの変更

特定のコンポーネントに対して、デフォルトのスタイルやpropsを設定することができます。

```tsx
const theme = createTheme({
  components: {
    MuiButton: {
      defaultProps: {
        variant: 'contained',
        disableElevation: true,
      },
      styleOverrides: {
        root: {
          borderRadius: 8,
        },
      },
    },
  },
});
```

# 型定義の拡張

MUIをTypeScriptで使う場合、カスタマイズした内容に対して型定義を拡張する必要があります。

## カスタムvariantの型定義

デフォルトには存在しないvariantを追加する場合、以下のように型定義を拡張します。

```tsx
declare module "@mui/material/Typography" {
  interface TypographyPropsVariantOverrides {
    body3: true;
    body4: true;
  }
}
```

## sx propsでのタイプセーフティ向上

sx propsで色やスペーシングを指定する際、文字列で指定すると型チェックが効きません。

```tsx
<TableCell
  sx={{
    color: "text.secondary",
    backgroundColor: "blueGrey.50",
  }}
>
  {children}
</TableCell>
```

このような場合でも間違った値を指定してしまう可能性があるため、可能であればtheme objectを使った型安全な方法を検討すると良いでしょう。

# 効率的な開発のための知識

## MUIのimport

パフォーマンスと開発効率の観点から、Barrelパターンの採用は避けて直接importすることを推奨します。

```tsx
// 推奨
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';

// 非推奨
import { Button, TextField } from '@mui/material';
```

この理由は以下の通りです：
- **パフォーマンス**: 必要なコンポーネントのみがバンドルに含まれる
- **@mui/codemodが使える**: 後述するcodemodツールが適切に動作する

## Slots Propsの活用

古いバージョンでは`XXProps`という形でコンポーネントの特定パーツの差し替えやpropsの指定ができましたが、現在は`slots`と`slotProps`に統一されています。

```tsx
<TextField
  slots={{
    input: CustomInput,
  }}
  slotProps={{
    input: {
      customProp: 'value',
    },
  }}
/>
```

## MUI Xの活用

MUIだけでは提供されていない機能については、MUI Xを活用することで補完できます。DateTimePickerやデータテーブルなど、より高度なコンポーネントが提供されています。

# 実装のコツと注意点

## 独自アイコンの定義

MUIのアイコンセットにない独自のアイコンを使いたい場合は、SvgIconコンポーネントを使って定義できます。

```tsx
import SvgIcon, { type SvgIconProps } from "@mui/material/SvgIcon";

type IconProps = SvgIconProps;

export const HospitalRoundedIcon = (props: IconProps) => (
  <SvgIcon {...props}>
    <path
      fillRule="evenodd"
      clipRule="evenodd"
      d="M17 5V7.00912H19C20.1 7.00912 21 7.90912 21 9.00912V19C21 20.1 20.1 21 19 21H14C13.45 21 13 20.55 13 20V17H11V20C11 20.55 10.55 21 10 21H5C3.9 21 3 20.1 3 19V9C3 7.9 3.9 7 5 7H7V5C7 3.9 7.9 3 9 3H15C16.1 3 17 3.9 17 5ZM10.6667 6.5C10.6667 6.22386 10.8905 6 11.1667 6H12.8333C13.1095 6 13.3333 6.22386 13.3333 6.5V8.66667H15.5C15.7761 8.66667 16 8.89052 16 9.16667V10.8333C16 11.1095 15.7761 11.3333 15.5 11.3333H13.3333V13.5C13.3333 13.7761 13.1095 14 12.8333 14H11.1667C10.8905 14 10.6667 13.7761 10.6667 13.5V11.3333H8.5C8.22386 11.3333 8 11.1095 8 10.8333V9.16667C8 8.89052 8.22386 8.66667 8.5 8.66667H10.6667V6.5Z"
    />
  </SvgIcon>
);
```

## Paperコンポーネントの注意点

Paperコンポーネントは、アプリ側で直接使うことはほぼありません。しかし、MUIの多くのコンポーネントが内部的にPaperを使用しているため、Paperのベーススタイルをカスタマイズするとあらゆるコンポーネントにその影響が波及します。変更する際は注意が必要です。

## レイアウトコンポーネントの使い分け

レイアウトは主にBoxやStackコンポーネントで行います。Stackを使うとデフォルトでflexレイアウトになっているため、要素を並べる際に便利です。

```tsx
<Stack direction="row" spacing={2}>
  <Button>ボタン1</Button>
  <Button>ボタン2</Button>
</Stack>
```

## ドキュメントの効果的な活用

MUIのドキュメントを効率的に使うために、以下の点を覚えておくと便利です：

- **Componentsページ**: 各コンポーネントの使用例を確認
- **APIページ**: ページ下部にあるAPIセクションで、渡せるpropsやCSS classesの詳細情報を確認

# アップデートとメンテナンス

## マイグレーションガイドの活用

MUIのバージョンアップを行う際は、必ずマイグレーションガイドを確認しましょう。APIの破壊的変更については、多くの場合codemodが用意されています。

## codemodの活用

codemodは、APIの変更を自動的に適用してくれるツールです。CSS Classesが変更されたりした場合も、codemodで移行できることが多いです。

```bash
npx @mui/codemod@latest <codemod-name> <path>
```

# その他の補足情報

## 足りない機能の補完

MUIには含まれていない機能もあります。例えば、ファイルのDrop系のコンポーネントはないため、react-dropzoneなどのライブラリと組み合わせて自作する必要があります。

## 将来の展望

RSC（React Server Components）サポートのため、ゼロランタイムCSSのPigment CSSという仕組みが開発されています。ただし、まだ実用段階ではないようです。詳細は[マイグレーションガイド](https://mui.com/material-ui/migration/migrating-to-pigment-css/)を参照してください。

# まとめ

この記事では、MUIを実際のプロジェクトで使う上で知っておきたい様々な知識について紹介しました。

特に重要なポイントをまとめると：

- **ライブラリ構成の理解**: Material UI、MUI System、Joy UIの違いを把握する
- **効率的なカスタマイズ**: sx props、styled関数、themeを適切に使い分ける
- **型定義の拡張**: TypeScriptプロジェクトでは型安全性を重視する
- **パフォーマンスを意識したimport**: 直接importでバンドルサイズを最適化する
- **アップデート対応**: codemodを活用して効率的に移行する

MUIは非常に多機能なライブラリですが、これらの知識を持っておくことで、より効率的で保守性の高い開発ができるようになると思います。

この記事が、MUIを使った開発の参考になれば幸いです🎉
