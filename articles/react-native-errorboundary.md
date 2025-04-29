---
title: "Reactのエラーハンドリングを実装する(react native)"
emoji: "🕌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: true
---

# ErrorBoundary で検知できないエラーに対応するには

※サーバーサイドレンダリングに関してはここでは扱いません。(react native 実装をしているため。)

[React の公式ドキュメント](https://ja.legacy.reactjs.org/docs/error-boundaries.html)では以下の通り

> **error boundary は以下のエラーをキャッチしません：**
>
> イベントハンドラ
> 非同期コード（例：setTimeout や requestAnimationFrame のコールバック）
> サーバサイドレンダリング
> （子コンポーネントではなく）error boundary 自身がスローしたエラー

つまり、個別でエラーハンドリングが必要な非同期処理や、APi リクエストを ErrorBoundary で一元的に管理したい場合には、工夫が必要ということらしいです。

`react-error-boudanry` などの最適化されたライブラリもあるが、こちらが上記の場合のエラーを内部実装で考慮しているかは不明。おそらく、

## イベントハンドラー・非同期のエラー

Error Boudanry で非同期エラーを検出するためには、それらのエラーをレンダリングプロセスに組み込む必要があります。

#### 参考

https://zenn.dev/nissy_dev/scraps/c9a201c0b81e1e
https://medium.com/@deasamniashvili_82561/error-boundary-with-asynchronous-code-in-react-f593af001d96

```typescript
// hooks
export function useErrorHandler(
  givenError?: unknown
): (error: unknown) => void {
  const [error, setError] = React.useState<unknown>(null);
  if (givenError != null) throw givenError;
  if (error != null) throw error;
  return setError;
}

// ---------------------------------

// component
useEffect(() => {
  (async () => {
    try {
      throw new Error("error");
    } catch (error) {
      errorHandler(error as Error);
    }
  })();
}, [errorHandler]);
```

上記の Hooks と ErrorBoundary を組み合わせることで、非同期で try catch したエラーを一元的にハンドリングすることができるようになりました。

:::message
注意点としては、非同期処理内では上記の useErrorHandler で解決しなければエラーは errorBoundary でキャッチできないので、非同期処理や、イベントフロー内ではからなず、try catch するようにしましょう。

[こちらの記事](https://www.asobou.co.jp/blog/web/error-boundary)で書かれている window のイベントリスナーを errorBoundary で定義すれば、可能なようなのですが React Native で widnow インスタンスが存在しないので使用できません。
:::

# React Native での ErrorBoundary の実装

**候補**

- react-error-boundanry
- react-native-error-boundary
- ゴリゴリ実装

試してみた結果

### react-native-error-boundary

- 自分の環境だけなのか、import⇩ 際の型が合わずエラーとなってしまって、ちょっとめんどくさかった。

```typescript
import * as ErrorBoundaryModule from "react-native-error-boundary";
import type { Props } from "react-native-error-boundary/lib/ErrorBoundary";
type State = { error: Error | null };
const ErrorBoundary = ErrorBoundaryModule as unknown as React.Component<
  Props,
  State
>;
```

- option が、react-error-boundary より少ないので、独自実装が必要

### ゴリゴリ実装

- スクラッチで書いていくので、複雑にエラーハンドリングを実装していきたい場合にちょっとめんどくさい
- 例えばでいうと、errorInfo ををどのように外部コンポーネントから参照できるようにするか
- reset を外部から発火させたい場合には errorBoundary を Context 化して、hooks で state を切り替える実装が必要そうであったりなど
