---
title: CSS でリンクの下線を消す方法 – HTML スタイルガイド
date: 2024-08-19T13:21:26.752Z
author: Kolade Chris
authorURL: https://www.freecodecamp.org/news/author/koladechris/
originalURL: https://www.freecodecamp.org/news/remove-underline-from-link-in-css/
posteditor: ""
proofreader: ""
---

Check if the workflow works with PAT

ウェブ開発者なら、ページにリンクを追加したときにデフォルトで表示される下線を消したいと思ったことがあるでしょう。

<!-- more -->

幸いなことに、他の要素と同様にリンクを表示するアンカータグをスタイルすることができます。

この記事では、CSS を使ってリンクの下線を消す方法を紹介します。また、リンクの4つの状態と、それぞれの状態で下線を消す方法も説明します。

## CSS でリンクの下線を消す方法

まず、リンクタグ（アンカータグ）がブラウザでデフォルトでどのように表示されるかを見てみましょう: ![ss1-4](https://www.freecodecamp.org/news/content/images/2022/06/ss1-4.png)

まず、リンクタグ（アンカータグ）には以下の4つの異なる状態（疑似クラス）があることを知っておくことが重要です:

-   `a:link`: リンクがアクティブでも訪問済みでもなく、ホバーもしていない通常の状態
-   `a:visited`: ユーザーがリンクをクリックした、つまり訪問済みの状態
-   `a:hover`: ユーザーがリンクにホバーしている状態
-   `a:active`: ユーザーがリンクをクリックしている状態

**注意:** CSS のカスケード特性により、状態（疑似クラス）は上記の順序で記述する必要があります。

デフォルトのリンクの下線を消すためには、すべての疑似クラスをターゲットにして、`text-decoration` プロパティに `none` を設定します。

```
<p>This is a <a href="#">link</a></p>
```

```
 a:link {
      text-decoration: none;
}

a:visited {
      text-decoration: none;
}

a:hover {
      text-decoration: none;
}

a:active {
      text-decoration: none;
}
```

![ss2-4](https://www.freecodecamp.org/news/content/images/2022/06/ss2-4.png)

また、アンカー要素セレクタを使用して一括でデフォルトの下線を消すこともできます。

```
 a {
       text-decoration: none;
}
```

![ss3-5](https://www.freecodecamp.org/news/content/images/2022/06/ss3-5.png)

リンクタグの4つの疑似クラスで遊んでみましょう。この CodePen が便利です:

<iframe width="100%" height="350" src="https://codepen.io/koladechris/embed/bGLPzXr" style="aspect-ratio: 16 / 9; width: 100%; height: auto;" title="CodePen embed" scrolling="no" allowtransparency="true" allowfullscreen="true" loading="lazy"></iframe>

## 結論

この記事が CSS でリンクのデフォルトの下線を消す方法を学ぶ手助けになれば幸いです。

この記事が役立った場合は、ぜひ友人や家族とシェアしてください。

読んでいただきありがとうございました。

