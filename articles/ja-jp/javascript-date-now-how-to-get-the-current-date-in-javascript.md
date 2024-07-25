---
title: JavaScript Date Now – JavaScript で現在の日付を取得する方法
date: 2024-07-25T06:54:26.517Z
author: Vijit Ail
authorURL: https://www.freecodecamp.org/news/author/vijit/
OriginalURL: https://www.freecodecamp.org/news/javascript-date-now-how-to-get-the-current-date-in-javascript/
Proofreader: ""
---

アプリケーションを開発する際、リソースの作成日やアクティビティのタイムスタンプなど、何らかの日付コンポーネントを扱う場面が多いでしょう。

<!-- more -->

日付やタイムスタンプのフォーマットを扱うのは大変ですが、このガイドでは JavaScript で現在の日付をさまざまな形式で取得する方法を学びます。

## JavaScript の Date オブジェクト

JavaScript には日付と時間を管理するための組み込み `Date` オブジェクトがあります。この `Date` オブジェクトを新しく生成するには `new` キーワードを使用します。

```js
const date = new Date();
```

`Date` オブジェクトは、1970 年 1 月 1 日からの経過ミリ秒を表す `Number` を内部に持っています。特定の日付を表すオブジェクトを作成するには、`Date` コンストラクタに日付文字列を渡すことができます。

```js
const date = new Date('Jul 12 2011');
```

現在の年を取得するには `Date` オブジェクトのインスタンスメソッド `getFullYear()` を使用します。`getFullYear()` メソッドは、`Date` コンストラクタで指定された日付の年を返します。

```js
const currentYear = date.getFullYear();
console.log(currentYear); //2020
```

同様に、現在の月の日や現在の月を取得するためのメソッドもあります。

```js
const today = date.getDate();
const currentMonth = date.getMonth() + 1; 
```

`getDate()` メソッドは、現在の月の日付 (1-31) を返します。`getMonth()` メソッドは指定された日付の月を返します。`getMonth()` メソッドが注意すべき点は、0 インデックス (0-11) の値を返すことです（0 は 1 月、11 は 12 月）。したがって、月の値を正しく表現するために 1 を加えています。

## Date now

`now()` は `Date` オブジェクトの静的メソッドです。このメソッドは、1970 年から経過した時間をミリ秒単位で返します。`now()` メソッドから返されたミリ秒を `Date` コンストラクタに渡して新しい `Date` オブジェクトを生成することができます。

```js
const timeElapsed = Date.now();
const today = new Date(timeElapsed);
```

## 日付のフォーマット

`Date` オブジェクトのメソッドを使って、日付をさまざまな形式 (GMT、ISO など) にフォーマットできます。

`toDateString()` メソッドは、人間が読みやすい形式で日付を返します。

```js
today.toDateString(); // "Sun Jun 14 2020"
```

`toISOString()` メソッドは、ISO 8601 拡張形式に従った日付を返します。

```js
today.toISOString(); // "2020-06-13T18:30:00.000Z"
```

`toUTCString()` メソッドは、UTC タイムゾーン形式で日付を返します。

```js
today.toUTCString(); // "Sat, 13 Jun 2020 18:30:00 GMT"
```

`toLocaleDateString()` メソッドは、ローカルに特化した形式で日付を返します。

```js
today.toLocaleDateString(); // "6/14/2020"
```

`Date` メソッドの完全なリファレンスは [MDN ドキュメント][1] で確認できます。

## カスタム日付フォーマッター関数

前述の形式以外にも、アプリケーションによっては `yy/dd/mm` や `yyyy-dd-mm` などの異なる形式が必要な場合があります。この問題に対処するために、複数のプロジェクトで使用できる汎用的な関数を作成するのが良いでしょう。

以下のセクションでは、引数で指定された形式で日付を返すユーティリティ関数を作成します。

```js
const today = new Date();

function formatDate(date, format) {
	// 
}

formatDate(today, 'mm/dd/yy');
```

引数で渡されたフォーマット文字列から「mm」、「dd」、「yy」などの文字列をそれぞれの月、日、年の値に置き換える必要があります。そのためには、以下のように `replace()` メソッドを使用します。

```js
format.replace('mm', date.getMonth() + 1);
```

しかし、このままでは多くのメソッドチェーンが必要となり、関数が複雑で保守しにくくなります。

```js
format.replace('mm', date.getMonth() + 1)
    .replace('yy', date.getFullYear())
	.replace('dd', date.getDate());
```

メソッドチェーンの代わりに、`replace()` メソッドと正規表現を使用することができます。まず、キーと値のペアを表すオブジェクトを作成します。

```js
const formatMap = {
	mm: date.getMonth() + 1,
    dd: date.getDate(),
    yy: date.getFullYear().toString().slice(-2),
    yyyy: date.getFullYear()
};
```

次に、正規表現を使って文字列を一致させて置き換えます。

```js
formattedDate = format.replace(/mm|dd|yy|yyy/gi, matched => map[matched]);
```

完全な関数は以下の通りです。

```js
function formatDate(date, format) {
    const map = {
        mm: date.getMonth() + 1,
        dd: date.getDate(),
        yy: date.getFullYear().toString().slice(-2),
        yyyy: date.getFullYear()
    }

    return format.replace(/mm|dd|yy|yyy/gi, matched => map[matched])
}
```

この関数には、タイムスタンプをフォーマットする機能も追加できます。

## まとめ

これで JavaScript の `Date` オブジェクトについての理解が深まったことでしょう。アプリケーションで日付を扱う際は、`datesj` や `moment` といったサードパーティのライブラリも利用できます。

[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date

```markdown
# JavaScript における Date オブジェクトの使い方

JavaScript では、日時を扱うために `Date` オブジェクトが提供されています。このオブジェクトを利用することで、現在の日時を取得したり、特定の日付を操作したりすることが簡単にできます。ここでは、基本的な `Date` オブジェクトの使用方法について解説します。

## 現在の日時を取得する

現在の日時を取得するためには、次のように `Date` オブジェクトのインスタンスを生成します。

```javascript
let now = new Date();
console.log(now);
```

このコードを実行すると、現在の日時が表示されます。

## 特定の日付を設定する

特定の日付を設定するには、`Date` オブジェクトのコンストラクタに年、月、日などを引数として渡します。

```javascript
let specificDate = new Date(2023, 9, 25); // 2023 年 10 月 25 日を設定
console.log(specificDate);
```

ここで注意が必要なのは、月の値が 0 から始まるため、「10 月」は `9` で表されることです。

## 日付の操作

日付を加算・減算する場合、`Date` オブジェクトの各メソッドを利用します。たとえば、1 日を加算する方法は次の通りです。

```javascript
let tomorrow = new Date();
tomorrow.setDate(tomorrow.getDate() + 1);
console.log(tomorrow);
```

このコードを実行すると、1 日後の日時が表示されます。

## まとめ

`Date` オブジェクトは、JavaScript で日時を扱うための強力なツールです。現在の日時の取得や、特定の日付の設定、日付の操作など、さまざまな用途に対応しています。詳細な情報やより高度な使用例については、[MDN の公式ドキュメント][1] を参考にしてください。

[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date
```

