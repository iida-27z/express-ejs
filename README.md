# `Express`における静的ファイル配信とEJSテンプレートエンジン

## 静的ファイル配信とは

前回までに、`Express`を使ってWEBサーバーを立ち上げ、ルーティングを設定し、さらにはデータベースと連携する方法を学んできました。<br>
これまで作ってきたアプリケーションでは、`res.send()`を使って文字列やJSONデータをクライアントに返していましたが、実際のWEBアプリケーションでは、以下のような種類のファイルをクライアントに届ける必要があります。<br>

- HTMLファイル: ページの構造を定義する
- CSSファイル: ページの見た目を装飾する
- JavaScriptファイル: ページに動きを追加する
- 画像ファイル: ページに表示する画像を提供する

このようなサーバー側で加工せずにそのままクライアントに送信するファイルのことを **「静的ファイル」** と呼びます。<br>
一方で、この資料の後半で詳しく解説しますが、サーバー側でデータを埋め込む処理を行うことが可能な **「テンプレートファイル」** を使うと動的なコンテンツを作成することも可能です。<br>
今回はまず、`Express`の機能を使って、HTMLやCSSなどの静的ファイルを配信する方法を学んでいきましょう。<br>

## `express.static`ミドルウェアの利用

`Express`で静的ファイルを配信するには、標準で用意されている **`express.static`** というミドルウェアを使用します。<br>
まずはプロジェクト内に、静的ファイルを配置するためのディレクトリを作成しましょう。一般的には **`public`** という名前がよく使われます。<br>
プロジェクトディレクトリの中に `public` ディレクトリを作成し、その中に `index.html`と`style.css` というファイルを作って以下のコードを記述してみてください。<br>

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTMLでページを表示</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
    <h1>Hello, Express!!</h1>
</body>
</html>
```
```css
@charset "UTF-8";
body {
    background-color: #f0f8ff;
    color: #333;
    font-family: sans-serif;
}
h1 {
    color: #0066cc;
}
```

続いて、`src/server.js`を編集して、この `public` ディレクトリの中身を配信できるように設定します。<br>
以下のコードのように、`app.use(express.static('public'));` を含むコードを記述してください。<br>

```JavaScript
const express = require('express');

const app = express();

// publicディレクトリ内のファイルを静的ファイルとして配信するミドルウェア
app.use(express.static('public'));

app.listen(3000, () => {
    console.log('サーバー起動中…（ポート番号:3000）');
});
```

この状態でサーバーを起動（`node src/server.js`）し、WEBブラウザで [`http://localhost:3000/index.html`](http://localhost:3000/index.html) にアクセスしてみましょう。<br>
単なる文字だけでなく、`public/style.css` が読み込まれ、背景色が薄い青色になり、見出しが青色に装飾された`index.html`の中身が表示されるはずです。<br>

`express.static` を使うと、指定したディレクトリ（ここでは `public`）の中にあるファイルが、URLのルート（`/`）直下にあるものとして公開されます。<br>
例えば、`public/images/logo.png` という画像を配置すれば、ブラウザからは `http://localhost:3000/images/logo.png` でアクセスできるようになります。<br>

## テンプレートエンジンとは

さて、静的ファイルを読み込ませることはできましたが、今のままでは、毎回同じ内容を表示することしかできません。<br>
データベースから取得したユーザー名や商品データなどを、HTMLの狙った場所に動的に埋め込みたいという要件も必ず出てきます。<br>

そこで活躍するのが **「テンプレートエンジン」** です。<br>
テンプレートエンジンを使うと、HTMLの雛形（テンプレート）となるファイルをあらかじめ用意しておき、<br>
リクエストが来たタイミングで、サーバー側でデータをテンプレートに流し込み、完成したHTMLを生成してクライアントに返すことができます。<br>

`Express`では様々なテンプレートエンジンを利用できますが、今回はHTMLに非常に近い記述ができ、<br>
学習コストも低い **`EJS (Embedded JavaScript templating)`** を使ってみましょう。<br>

## EJSのインストールと設定

まずは、`EJS` パッケージをインストールします。ターミナルを開き、以下のコマンドを実行してください。<br>

```bash
npm install ejs
```

インストールが完了したら、`Express`に「ビューエンジン（テンプレートエンジン）としてEJSを使用する」ことを教える必要があります。<br>
`src/server.js` を以下のように修正しましょう。<br>

```JavaScript
const express = require('express');

const app = express();

// ビューエンジンとしてejsを設定
app.set('view engine', 'ejs');

// 静的ファイルの配信
app.use(express.static('public'));

app.get('/ejs', (req, res) => {
    // res.sendの代わりにres.renderを使用する
    res.render('index', { title: 'EJSのテスト', message: 'こんにちは、EJSの世界へ！' });
});

app.listen(3000, () => {
    console.log('サーバー起動中…（ポート番号:3000）');
});
```

ここで新しく登場したのが **`app.set('view engine', 'ejs');`** と **`res.render()`** です。<br>
`app.set` で設定を行うと、`Express`はデフォルトでプロジェクトディレクトリ内の **`views`** というディレクトリの中にあるテンプレートファイルを探しにいくようになります。<br>
そして `res.render` の第1引数には、表示したいテンプレートファイル名（拡張子の `.ejs` は省略可）を、<br>
第2引数には、テンプレートファイルに渡したいデータをオブジェクト形式で指定します。<br>

## EJSテンプレートの作成と基本構文

設定が済んだら、テンプレートファイルを作成しましょう。<br>
プロジェクトディレクトリ直下に **`views`** というディレクトリを作成し、その中に **`index.ejs`** というファイルを作成してください。<br>
ファイルの中身は以下のように記述します。<br>

```ejs
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title><%= title %></title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
    <h1><%= message %></h1>
    <p>これはEJSを使ってレンダリングされたHTMLページです。</p>
</body>
</html>
```

ここで注目してほしいのが、**`<%= title %>`** や **`<%= message %>`** という記述です。<br>
これがEJSの基本的な構文で、`res.render` の第2引数で渡したデータの値（プロパティ）をこの部分に出力（表示）してくれます。<br>
基本的にはHTMLと同じように書くことができ、動的に変えたい部分だけを `<%= %>` で囲むだけでよいので、とても直感的ですね。<br>

それでは、サーバーを再起動して [`http://localhost:3000/ejs`](http://localhost:3000/ejs) にアクセスしてみましょう。<br>
ブラウザのタブ名（タイトル）が「EJSのテスト」になり、画面には「こんにちは、EJSの世界へ！」と表示されていれば大成功です！<br>

## EJSの制御構文（条件分岐とループ処理）

EJSが真価を発揮するのは、単なる変数の表示だけでなく、`JavaScript`の文法を使ってHTMLの出力をコントロールできる点です。<br>
条件分岐（if文）やループ処理（for文など）を組み込む書き方を見てみましょう。<br>

EJSで処理を記述する場合は、出力を伴う `<%= %>` ではなく、**`<% %>`** （イコールがない形）を使用します。<br>
`src/server.js` のルートハンドラを少し書き換えて、配列データを渡すようにしてみましょう。<br>

```JavaScript
app.get('/users', (req, res) => {
    // データベースから取得したと仮定するユーザーデータの配列
    const userList = [
        { name: 'Alice', age: 20 },
        { name: 'Bob', age: 17 },
        { name: 'Charlie', age: 25 }
    ];
    res.render('users', { users: userList });
});
```

次に、`views` ディレクトリ内に **`users.ejs`** というファイルを作成し、以下のように記述します。<br>

```ejs
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>ユーザー一覧</title>
</head>
<body>
    <h1>ユーザー一覧</h1>
    
    <% if (users.length > 0) { %>
        <ul>
            <% for (let i = 0; i < users.length; i++) { %>
                <li>
                    <%= users[i].name %> さん
                    
                    <% if (users[i].age >= 18) { %>
                        （成人）
                    <% } else { %>
                        （未成年）
                    <% } %>
                </li>
            <% } %>
        </ul>
    <% } else { %>
        <p>ユーザーが登録されていません。</p>
    <% } %>
</body>
</html>
```

サーバーを再起動し、[`http://localhost:3000/users`](http://localhost:3000/users) にアクセスしてみてください。<br>
配列内のデータがループ処理でリスト表示され、さらに年齢による条件分岐（成人と未成年の表示）が正しく行われていることが確認できるはずです。<br>

このように、EJSを使うことで、データベースから取得した配列やオブジェクトのデータ構造に合わせて、<br>
柔軟にHTMLを組み立ててクライアントに返すことが可能になります。<br>

### よく使うEJSタグのまとめ

|タグ|説明|
|-|-|
|`<% 処理 %>`|`JavaScript`の処理（if文やfor文など）を記述します。画面には何も出力されません。|
|`<%= 変数 %>`|変数の値をHTMLエスケープして（安全に）出力します。通常はこちらを使います。|
|`<%- include('ファイルパス') %>`|別のEJSファイルを読み込んで埋め込みます。ヘッダーやフッターの共通化に便利です。|

## この後の学習

ここまで、静的ファイルの配信とEJSによるテンプレートのレンダリングについて学んできました。<br>
これらの知識と前回学んだデータベースとの連携を組み合わせることで、本格的なWEBアプリケーションを作る準備が整いました。<br>
ここからは、どのような学習を進めればいいかを示してみます。<br>

* ヘッダー（`header.ejs`）やフッター（`footer.ejs`）を作成し、`<%- include() %>` を使って複数のページで共通部品として使い回してみる。
* クライアントサイドの `JavaScript` ファイルを `public/js` ディレクトリに配置し、EJS側で `<script>` タグを使って読み込ませてみる。
* `Bootstrap` などのCSSフレームワークのファイルを `public` に配置するかCDNを利用して、アプリケーションの見た目を本格的に整える。
* データベースから `SELECT` 文で取得した複数件のレコード（配列）を、`res.render` に渡してEJSの `for` 文でテーブル（`<table>`）として表示してみる。
* HTMLの `<form>` タグ（メソッドは `POST`）をEJSで作成し、入力されたデータを `Express` で受け取ってデータベースに `INSERT` する一連の処理を作ってみる。
* データの登録完了後やエラー発生時に、`res.redirect` を使って別の画面へ遷移させる処理を実装してみる。
