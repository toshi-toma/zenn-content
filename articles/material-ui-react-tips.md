---
title: "MUIを使う際に知っておきたいポイントまとめ"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mui", "react"]
published: false
publication_name: "lincwell_inc"
---

# はじめに

最近、[MUI](https://mui.com/material-ui/)を使う機会があり、初見だとライブラリの基本的な概念や方針、スタイリングやカスタマイズはどのように行うのか、どの機能を使うのかといった点を把握するのに時間がかかりました。

https://mui.com/

この記事では、React向けUIライブラリのMUIを実際のプロジェクトで使う際、事前に知っておくとキャッチアップがスムーズにいきそうな点を紹介します。

:::message
この記事はMUI v7の情報を元に書いています
:::

# MUIのライブラリ構成や基本情報

まず、MUIに関連するライブラリについて整理します。

## 主要なライブラリ

**Material UI**
- MUIの中核となるReact UIコンポーネントライブラリ
- ボタン、テキストフィールド、ダイアログなど、Material Designに基づいたコンポーネントを提供

**MUI System**
- MUIに組み込まれているスタイリング系の機能を提供するライブラリ
- MUIでスタイリングを行う際に利用するsx propsやstyled関数などの機能はこちらで実装
- スタイリングやレイアウト系の詳細な仕様については[MUI Systemのドキュメント](https://mui.com/system/getting-started/)を参照する

**Joy UI**
- Material UIとは異なるデザインシステムのUIコンポーネントライブラリであった
- しかし、現在は開発が停止されているため、新規プロジェクトでは選択肢から外した方が良い

## FigmaのDesign Kits

[MUIのFigma UI Kit](https://mui.com/store/items/figma-react/)が販売されています。Figmaを使ったデザインフローを採用する場合は必要になることが多いです。

## MUI X

MUIだけでは提供されていない機能(例えばDatePicker)については、[MUI X](https://mui.com/x/)を活用することで補完できます。DateTimePickerやグラフなど、より高度なコンポーネントが提供されています。
基本的な機能は無料で利用できるが、日付の範囲選択機能など一部機能がPro Planとして有料で利用できます。

## MUIのドキュメント

[MUIのドキュメント](https://mui.com/material-ui/getting-started/)で基本的な使い方やコンポーネント、カスタマイズ方法などを知ることができます。
ComponentsページとAPIページが分かれています。

- **Componentsページ**: 各コンポーネントの使用例を確認。APIセクションにAPIドキュメントへの導線がある
- **APIページ**: 渡せるpropsやCSS classesの詳細情報を確認。


# スタイリングとカスタマイズ

MUIはsx propsというスタイリングの仕組みを提供しているため、CSS ModulesやTailwindのようなCSSライブラリを使う必要はなく、MUIだけで完結します。Themeとの連携を考えるとMUIのsx propsに寄せた方が良いです。
また、コンポーネントをカスタマイズする方法はいくつかありますが、用途に応じて使い分けることが重要です。

## 基本的なスタイリング方針

基本的にはMUIのコンポーネントにsx propsを渡してスタイリングしたり、BoxやStackのようなレイアウト系のコンポーネントを組み合わせることで大部分のスタイリングは対応できます。

従来のバージョンではコンポーネントのpropsで`mr={2}`のような形で`margin-right`を指定できるSystem Propsという仕組みがありました。しかし、これは非推奨になっているため、すべてsx propsに統一することを推奨します。

## 1箇所でだけスタイルを変更する場合

**sx propsを渡す**

最も基本的な方法です。ただし、sxはrootの要素にのみ適用されるため、コンポーネント内部の特定の要素をカスタマイズしたい場合は、CSS classを指定してカスタマイズする必要があります。各コンポーネントごとのAPIドキュメントに利用できるCSS classesが記載されています。

```tsx
<Button
  sx={{
    backgroundColor: 'primary.main',
    color: 'white',
    '&:hover': {
      backgroundColor: 'primary.dark',
    },
    '& .Mui-disabled': {
      backgroundColor: 'primary.dark'
    }
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

:::message
styled関数を使う際は、`@mui/system`ではなく`@mui/material/styles`からimportしてください。`@mui/system`を使うとThemeの型定義の変更が反映されません。
:::

## コンポーネントのベースとなるスタイルを変更する場合

**themeの設定**

アプリ全体で共通のカスタマイズを適用したい場合は、themeで設定します。

# themeの拡張とカスタマイズ

デフォルトのテーマがありますが、そのまま使うことは少ないため、基本的にはカスタマイズすることになります。

[デフォルトテーマ](https://mui.com/material-ui/customization/default-theme/)を見ると初期で設定されている値が分かります。

## createThemeを使ったカスタマイズ

createThemeにカスタマイズする内容を入れることで、テーマを拡張できます。デフォルト値の上書きです。

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

デフォルトにはないtypographyのvariantを追加できます。上記の例では`body3`を追加していますが、これを使うためには型定義の拡張も必要です。

## コンポーネントのデフォルトスタイルやpropsの変更

特定のコンポーネントに対して、デフォルトのスタイルやpropsを設定できます。

```tsx
const theme = createTheme({
  components: {
    MuiButtonBase: {
      defaultProps: {
        disableRipple: true,
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

## 型定義の拡張

MUIをTypeScriptで使う場合、Themeでカスタマイズした内容を型定義に反映する必要があります。

### カスタムvariantの型定義

Typographyでデフォルトには存在しないvariantを追加する場合、以下のように型定義を拡張します。

```tsx
declare module "@mui/material/styles" {
  interface TypographyVariants {
    body3: React.CSSProperties;
  }
  interface TypographyVariantsOptions {
    body3: React.CSSProperties;
  }
}
declare module "@mui/material/Typography" {
  interface TypographyPropsVariantOverrides {
    body3: true;
  }
}
```

他にも同様の方法でpaletteの名前空間やvariantを増やすこともできます。

### sx propsからThemeへのアクセス

sx propsで色などのTheme情報を指定する際、文字列で指定すると型チェックが効きません。

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

間違った値を指定してしまう可能性があるため、可能であればtheme objectを使ったり、文字列ではなく関数形式でtheme objectにアクセスする方法も検討すると良いでしょう。


# アップデートとメンテナンス

## マイグレーションガイドの活用

MUIのバージョンアップを行う際は、必ずマイグレーションガイドを確認しましょう。APIの破壊的変更については、多くの場合codemodが用意されています。

## codemodの活用

codemodは、APIの変更を自動的に適用してくれるツールです。CSS Classesが変更されたりした場合も、codemodで移行できます。

```bash
npx @mui/codemod@latest <codemod-name> <path>
```

# その他の補足情報

## 独自アイコンの定義

MUIのアイコンセットにない独自のアイコンを使いたい場合は、[SvgIconコンポーネント](https://mui.com/material-ui/icons/#svgicon)を使って定義できます。

```tsx
import SvgIcon, { type SvgIconProps } from "@mui/material/SvgIcon";

type IconProps = SvgIconProps;

export const MyIcon = (props: IconProps) => (
  <SvgIcon {...props}>
    <svg
      xmlns="http://www.w3.org/2000/svg"
      viewBox="0 0 24 24"
      fill="none"
    >
      <path
        d="..."
      />
    </svg>
  </SvgIcon>
);
```

## レイアウトコンポーネントの使い分け

レイアウトは主にBoxやStackコンポーネントで行います。Stackを使うとデフォルトでflexレイアウトになっているため、要素を並べる際に便利です。

```tsx
<Stack direction="row" spacing={2}>
  <Button>ボタン1</Button>
  <Button>ボタン2</Button>
</Stack>
```

## Paperコンポーネント

Paperコンポーネントは、アプリ側で直接使うことはほぼありません。しかし、MUIの多くのコンポーネントが内部的にPaperを使用しているため、Paperのベーススタイルをカスタマイズするとあらゆるコンポーネントにその影響が波及します。変更する際は注意が必要です。

## MUIのimport

[開発体験のパフォーマンス観点](https://mui.com/material-ui/guides/minimizing-bundle-size/)からbarrel importsは避けた方が良いです。
また、mui codemodへの対応観点から、ラッパーファイル経由でのimportは基本的には避けて直接importすることを推奨します。
muiのcodemodは直接muiパッケージからimportされているファイルのみ対応してるため、直接importしておくことでAPIの移行が容易になります。

```tsx
// 推奨
// sample.tsx
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';

// 非推奨
// mui-wrapper.ts
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';
export { Button, TextField }
// sample.tsx
import { Button, TextField } from './mui.ts';
```

## Slots Propsの活用

古いバージョンでは`XXProps`という形でコンポーネントの特定パーツの差し替えやpropsの指定ができましたが、現在は`slots`と`slotProps`に統一されています。

```tsx
<TextField
  slots={{
    input: CustomInput,
  }}
  slotProps={{
    input: {
      sx: {},
    },
  }}
/>
```

## 足りない機能の補完

MUIには含まれていない機能も多くあります。例えば、ファイルのDrop系のコンポーネントは組み込みだとないため、react-dropzoneなどのライブラリと組み合わせて自作する必要があります。

## RSC対応

RSC（React Server Components）サポートのため、ゼロランタイムCSSのPigment CSSという仕組みが開発されています。ただし、まだ実用段階ではないようです。詳細は[マイグレーションガイド](https://mui.com/material-ui/migration/migrating-to-pigment-css/)を参照してください。

# まとめ

MUIを実際のプロジェクトで使う際、個人的に事前に知っておけると嬉しかった情報を紹介しました。
この記事が、初めてMUIを使う際の参考になれば幸いです。
