---
title: "Reactのエラーハンドリングを実装する(react native)"
emoji: "🕌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: true
---

# ErrorBoundary で検知できるエラー

[React の公式ドキュメント](https://ja.legacy.reactjs.org/docs/error-boundaries.html)では以下の通り

> **error boundary は以下のエラーをキャッチしません：**
>
> イベントハンドラ（詳細）
> 非同期コード（例：setTimeout や requestAnimationFrame のコールバック）
> サーバサイドレンダリング
> （子コンポーネントではなく）error boundary 自身がスローしたエラー

つまり、個別でエラーハンドリングが必要な非同期処理や、APi リクエストを ErrorBoundary で一元的に管理したい場合には、工夫が必要ということらしいです。`react-error-boudanry` などの最適化されたライブラリもあるが、、柔軟に構成したい場合はある程度、独自で実装するほうがいい場合もあります。

## 非同期のエラー

Error Boudanry で非同期エラーを検出するためには、それらのエラーをレンダリングプロセスに組み込む必要があります。

```typescript
const [,. setError] = useState()
useEffect(() => {
  setError(throw new Error("errorです"))
},[])
```
