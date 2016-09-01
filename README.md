# Riot.js Handson

## 0. 概要

> A React-like user interface micro-library

- Javascript製の**UIライブラリ**
	- Javascript製MVCライブラリ（Backbone.jsやAngularJSなど）や、React.jsなどといったWEBアプリケーション作成ライブラリの一つ
	- SPA（single page application）を作るのに適している
	- 「React.jsがUIにフォーカスしてるのは賢い」と認めております
- 「カスタムタグ」「ルーティング」「オブザーバブル」という三つの機能（と付属機能）しか持たない
	- 故に軽い（React.jsが44.32KB／Polymerが45.69KB／Riot.jsが9.38KB）
- WEBコンポーネントの抽象化レイヤー（DOMに対するjQueryが目標）

[サンプルまとめ](https://mcatm.github.io/study_riotjs/)

### 0-1. どんなものができるか

https://mcatm.github.io/study_riotjs/finish.html

- TOPでは、CINRA.JOBの会社リストを取得
- MEMBERSでは、エンジニアチームのメンバー一覧を表示
	- バックエンド／フロントエンドで絞り込みできる
	- 画面最下部のフォームから、本物の部長を追加できる
- 画面遷移が非同期（ハッシュリンクだから当たり前だけれども…）
- 戻る／進むボタンが機能していることを確認できる

### 0-2. 他のフレームワークとの違い

参考：[RiotをReact・Polymerと比較する · Riot\.js](http://riotjs.com/ja/compare/)

#### React.js

- 重い！冗長！
- 文法キモい！

#### Polymer.js

- 重い！
- 後方互換ばっか気にしてる！

#### Vue.js

- サーバーサイドレンダリングに対応してない
- ビューとロジックが分離している！（良し悪し）
- DOM直接操作する前提！

---

## 1. カスタムタグ

### 1-1. カスタムタグ

#### カスタムタグとは？

- 参考：[カスタムタグ · Riot\.js](http://riotjs.com/ja/api/)
- HTMLタグの拡張で、自由なタグを定義できる
	- アプリケーションのビューにあたる
	- テンプレートとして実装することも可能
- レイアウト（HTML）とロジック（Javascript）の組み合わせ
- Cofeescriptなどのプリプロセッサや、Jadeなどのテンプレート言語を使うことも可能（まだ試してない）
- CSSも書ける。カスタムタグ内にスコープされるので、完全に切り分けられる

#### 最小のカスタムタグ実装

```html
<app/>

<script type="riot/tag">
<app>
<h1>APPタグ（何でもいい。ここはただのHTMLだから）</h1>
</app>
</script>

<script>riot.mount('app')</script>
```

- タグの名前は自由に指定できる
   -（ハイフン使うとか、W3Cの細かい規約はあるっぽい）
- `riot/tag`という独自DSL（domain-specific language）を使用する
	- 裏側で、riot.jsが、`riot/tag`をピュアなJSにコンパイルする
	- プリレンダリング可能（`riot/tag` → `text/javascript`）
	- サーバーサイドレンダリングも可能
- `riot:mount()`：カスタムタグは、JSでマウントして初めて処理される
	- `riot.mount('*')`で、全てのカスタムタグをマウントすることが出来る

##### ちなみに…

- これをReactで書くと…

```js
import React from 'react';
import ReactDOM from 'react-dom';

class App extends React.Component {
    render() {
        return (
            <h1>APPタグ（何でもいい。ここはただのHTMLだから）</h1>
        );
    }
}

ReactDOM.render(<App/>, mountNode);
```

#### コーディングポリシー

- セミコロン不要！
	- 「あってもいいけど、公式では取り込まないよ」

#### 実装

https://mcatm.github.io/study_riotjs/customtag-1.html

- 【注意】**/boilarplate.html**を複製してから、**index.html**に名称変更して作業してください。
- HTMLにカスタムタグが二つ（`app`と`nav`）記述してあることを確認してください

- カスタムタグ「app」を定義する
	- `<script>`のtypeが`riot/tag`であることに注意する

```html
<script type="riot/tag">
  <app>
    <h1>カスタムタグがマウントされました</h1>
  </app>
</script>
```

- マウントする
	- `riot.mount('カスタムタグ')`：カスタムタグをマウントする

```js
riot.mount('app');
```

---

### 1-2. ループとデータ構造

https://mcatm.github.io/study_riotjs/loop-1.html

- 参考：[カスタムタグ · Riot\.js](http://riotjs.com/ja/guide/#section-23)

#### 実装

- ループを作成する
	- `each`: 配列を与えるとループする。子要素の中で、配列のプロパティにアクセスできる
	- `if`: 値が負だったら、表示しない

```html
<app>
  <h1>エンジニアチーム</h1>
  <ul>
    <li each={ members } if={ active }>
      <h2>{ name }<small>（{ job }）</small></h2>
    </li>
  </ul>
</app>
```

- データを作成する
- `this`がカスタムタグ自身を示すスコープ
    - ここでは後々再利用するために、別変数（`_this`）に格納しています

```json
  <h1>エンジニアチーム</h1>
  <ul>
    （中略）
  </ul>
  
  var _this = this
  _this.members = [
		{ name: '濱田', job: 'Backend', active: true },
		{ name: '青木', job: 'Frontend', active: true },
		{ name: '二階', job: 'Backend', active: true },
		{ name: '大石', job: 'Frontend', active: true },
		{ name: '宇根', job: 'Backend', active: true },
		{ name: '伊藤', job: 'Frontend', active: true },
	]
</app>
```

- ループが確認できるはず

### 1-3. イベントハンドラ

https://mcatm.github.io/study_riotjs/event-1.html

- 参考：[カスタムタグ · Riot\.js](http://riotjs.com/ja/guide/#section-20)
- フォームを作る
	- `onsubmit`：`<form>`が送信された時に走るイベントを定義
		- 古式ゆかしいイベント記法ですが、riot.jsが上手いこと処理してくれます（出力されるHTMLには、`onclick`属性は出力されていない）
		- `preventDefault()`不要。riot.jsが勝手にやってくれる
		- `onclick`とかも同様に書けます

```html
<app>
　（中略）
  <form onsubmit="{ add_member }">
    <div class="form-group">
      <label>名前</label><br>
      <input type="text" name="name" class="form-control">
    </div>
    <div class="form-group">
      <label>担当</label><br>
      <select name="job" class="form-control">
        <option value="Backend">Backend</option>
        <option value="Frontend">Frontend</option>
      </select>
    </div>
    <div class="form-group">
      <button class="btn btn-default">追加する</button>
    </div>
  </form>
  （省略）
```

- onsubmitイベントの処理を定義する
	- 配列の中身を変えると即座に表示に反映されることが確認できるはず

```
<app>
  （中略）

  this.add_member = function(e) {
    _this.members.push( { name: this.name.value, job: this.job.value, active: true } )
  }
</app>
```

#### 1-3-1. 変数操作

https://mcatm.github.io/study_riotjs/event-2.html

- フィルターを作成
	- 各`<a>`タグに、onclickイベントを設定。`filter`メソッドを発火させる
	- （ちょっと今回面倒くさいので、`data`属性を定義して、jQueryで利用しています。もっと良いやり方はあるはず）

```html
<app>
<h1>エンジニアチーム</h1>
<ul>
  （中略）
</ul>
<hr>
Filter: <a onclick="{ filter }">All</a> - <a onclick="{ filter }" data-job="Backend">Backend</a> - <a onclick="{ filter }" data-job="Frontend">Frontend</a>
（省略）
</app>
```

- フィルターの機能を定義
	- フィルターでやってることは、単に`this.members`の内容を変えているだけ（選択しているJobと異なる場合に、`active: false`を設定する）

```js
<app>
（省略）

this.filter = function(e) {
  var job = $(e.target).data('job')
  _this.members.map(function(member) {
    member.active = member.job == job || !job
  })
}
</app>
```

- CSSを定義：aタグはポインターにしたいよね
	- `:scope`：このタグ自身
	- `a`も、`app a`を示していることが確認できるはず

```html
<app>
（省略）

<style scoped>
:scope { display: block; padding-bottom: 120px }
a { cursor: pointer }
</style>

</app>
```

---

## 2. ルーティング

### 2-1. ルーティングとは

- HistoryAPIを利用したルーティングを実現できる
    - 代替ライブラリとしては、`page.js`などが挙げられる
- Javascript MVCや、ReactJSも、ルーティングの機構は持っていて、より複雑
    - riot.jsのルーティングは非常に単純
    - そこがネックになる可能性はあるが、Nodeモジュールで代替可能
- `#`を使ったハッシュリンクがデフォルトだが、適切にサーバー設定することで、`/`をベースにした通常の遷移もすべてラップできる
- 遷移する毎にイベントが発生するので、遷移中のアニメーションなどが実装しやすい
- 参考：[ルータ · Riot\.js](http://riotjs.com/ja/api/route/)

### 2-2. ルートの設定

https://mcatm.github.io/study_riotjs/route-0.html

#### 準備

- 先ほどのコードの`<app>`タグを、`<members>`タグに変更

```html
<members>
<h1>エンジニアチーム</h1>
（省略）
</members>
```

- `<top>`タグを追加

```html
<app>（省略）</app>
<top>
  <h1>中身はなんでも良い、とりあえず今は</h1>
</top>
```

- `<nav>`タグを定義

```js
<app>（省略）</app>
<top>（省略）</top>

<nav>
<h2>Navigation</h2>
<ul class="list-group">
  <li each={ navs } class="list-group-item"><a href="#{ uri }">{ label }</a></li>
</ul>

this.navs = [
  {
    label: 'Top',
    uri: '',
  },
  {
    label: 'Members',
    uri: 'members',
  }
]
</nav>
```

- `<nav>`タグをマウント

```js
riot.mount('nav')
```

- `<app>`タグが定義されていないので、ナビしか表示されなくなったはず

#### 実装

https://mcatm.github.io/study_riotjs/route-1.html

- TOPページを作成
   - `<script>`内に記述
   - `riot.route('ルート', callback)`：ルートに書かれた条件に一致した場合、callbackを発火
	   - ルートの例：
		   - `''`：http://localhost/
		   - `'about`：http://localhost/about
		   - `'about/*'`：http://localhost/about/xxxxx
	- `riot.mount('app')`を下記に置き換え
	- `riot.mount('app', 'top')`：`<app>`タグの位置に`<top>`タグをマウント

```js
riot.route('', function() {
  riot.mount('app', 'top')
})
```

- riot.routeを有効にする
	- `riot.route.start(true)`：ルーティングがスタートする。自サイト内の全ての遷移がラップされる
  - リロードしてみると、トップページが表示されるはず

```js
riot.route(（省略）)

riot.route.start(true)
```

- Membersページを`<script>`内に追加

```js
riot.route('members', function() {
	riot.mount('app', 'members')
})
```

#### 確認

- `#members`に遷移すると、先ほどのエンジニアチームのメンバー表が表示される
- `#`に遷移すると、先ほどのトップページが表示される
- 戻る／進むが正常に機能する

---

## 3. オブザーバブル

### 3-1. オブザーバブルとは

#### 概要

> Observableはイベントを送ったり受け取ったりするための汎用的なツールです

- イベントを設定・送信・監視・受信するためのツール
- カスタムタグの中（`riot/tag`）にでも、外（`text/javascript`）にでも、好きなように置ける
    - アプリケーション全体を監視するようなObservableインスタンスを定義し、タグの動きを監視させることが可能
    - GAのキックとか、SNSボタンの初期化とか、非同期遷移でよくハマるポイントは、これを使うことで解消されますね
- 参考：[オブザーバブル · Riot\.js](http://riotjs.com/ja/api/observable/)

#### 実装

- `<script>`にオブザーバーを追加
	- `riot.observable(this)`：インスタンスを監視する。変更があった際（`trigger()`）に、定義された処理を発火する
	- `on('イベント名', callback)`：イベントの定義。コールバックはオブジェクトを受け取ることが出来る
	- `update()`：イベントハンドラやマウント以外での要素の変更は、明示的に表示を更新してやる必要がある

```
<script>
（省略）

function Receiver() {

// Receiverインスタンスを監視
riot.observable(this)

// 'get'イベントの監視
this.on('get', function(_this) {

  var page = page ? page : 1

  $.ajax({
    url: 'https://job.cinra.net/api/get/?pg=' + page,
    dataType: 'json'
  }).done(function(m) {
    _this.companies = m.companies
    _this.pagination = m.pagination
    $('.loading').fadeOut(800, function(e) {
      _this.update()
    })
  });
})

}

// インスタンスを作る
var receiver = new Receiver()

</script>
```

- `<top>`を書き換え
	`receiver.trigger('イベント名', オブジェクト)`：イベントをトリガー。Observableであるreceiverインスタンスに、イベントを送信している
	
```
<top>
<h1>TOP</h1>
<span class="loading">Loading...</span>
<div each={ companies }>
  <h2><a href={ permalink } target="_blank">{ title }</a></h2>
  <p>{ job }</p>
</div>

var _this = this
    page = 1;

receiver.trigger('get', _this);

</top>
```

### 3-2. アプリケーション設計

- 上記、全くオブザーバブルを使わずに実装することも出来る（[サンプル](https://mcatm.github.io/study_riotjs/observable-2.html)）
- 設計が柔軟で自由
	- 故に、設計者のセンスが問われる

---

## 4. まとめ

- すげー使いやすい
    - 学習コストが極端に低い
    - 変数のスコープを気にする必要があまりないので、設計の自由度が高い
- 設計にセンスが必要
    - チームで制作する場合、各々のエンジニアが設計しちゃったり、設計者の意図を無視した実装をすると、プロジェクトが簡単に破綻する（予感）
    - ReactJSで言うところの「Flux」のような仕組み（考え方）が必要になってくるかも
        - Flux：「データフローを一方通行にする」とか「データを扱うモデルを作る」とか、システムを設計するパターンみたいなもん（という理解）
            - MVC、みたいなもんですね
        - [jimsparkman/RiotControl: Event Controller / Dispatcher For RiotJS, Inspired By Flux](https://github.com/jimsparkman/RiotControl)
- 既存のサイトの置き換えが楽
    - 「システムはかっちり作ってあるけど、ガワを変えたい…」みたいな要望に応えやすい
    - APIとフロントの切り分けは、今後大いに推進していくべきだろう
        - とは思いつつ、コンテンツ配信は別サーバー立てないと、とか、プレビューどうする？とか、解決しなければならない問題はある
        - Docker！

---

### 参考URL

- [Riot\.js — A React\-like user interface micro\-library · Riot\.js](http://riotjs.com/ja/)
- [Riot\.js 触ってみたメモとサンプル \| WebTecNote](http://tenderfeel.xsrv.jp/javascript/2220/)
- [Riotjsのいいところ \- Qiita](http://qiita.com/jgs/items/afa7bca6d4d88812b7e4)
