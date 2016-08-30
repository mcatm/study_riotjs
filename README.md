# Riot.js Handson

## 0. 概要

> A React-like user interface micro-library

- Javascript製の**UIライブラリ**
- 「カスタムタグ」「ルーティング」「オブザーバブル」という三つの機能（と付属機能）しか持たない
- 故に軽い（React.jsが44.32KB／Polymerが45.69KB／Riot.jsが9.38KB）
- WEBコンポーネントの抽象化レイヤー（DOMに対するjQueryが目標）

### 0-1. どんなものができるか

https://mcatm.github.io/study_riotjs/finish.html

---

## 1. カスタムタグ

### 1-1. カスタムタグ

#### カスタムタグとは？

#### 最小のカスタムタグ実装

```html
<app/>
```

- タグを自由に指定できる

```js
<script type="riot/tag">
<app>
<h1>APPタグ</h1>
</app>
</script>
```

- `riot/tag`という独自DSL（domain-specific language）を使用する
	- 裏側で、riot.jsが、`riot/tag`をピュアなJSにコンパイルする

```js
<script>riot.mount('app')</script>
```

- カスタムタグをマウントする
- `riot.mount('*')`で、全てのカスタムタグをマウントすることが出来る

#### コーディングポリシー

- セミコロン不要！
	- 「あってもいいけど、公式では取り込まないよ」

#### 実装

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

```js
riot.mount('app');
```

---

### 1-2. ループとデータ構造

#### 実装

- データを作成する
- `this`がカスタムタグ自身を示すスコープ

```json
var _this = this

_this.members = [
	{ name: '濱田', job: 'Backend' },
	{ name: '青木', job: 'Frontend' },
	{ name: '二階', job: 'Backend' },
	{ name: '大石', job: 'Frontend' },
	{ name: '宇根', job: 'Backend' },
	{ name: '伊藤', job: 'Frontend' },
]
```

- ループを作成する

```html
<app>
  <h1>エンジニアチーム</h1>
  <ul>
    <li each='{ members }'>
      <h2>{ name }<small>（{ job }）</small></h2>
    </li>
  </ul>
</app>
```

### 1-3. イベントハンドラ

- フォームを作る

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
  
  var _this = this

      _this.members = [
        （中略）
      ]

      this.add_member = function(e) {
        _this.members.push( { name: this.name.value, job: this.job.value, active: true } );
      }
</app>
```
 
- `<form onsubmit="{ add_member }">`
 
```js
this.add_member = function(e) {
  _this.members.push( { name: this.name.value, job: this.job.value, active: true } );
 }
```

---

## 2. ルーティング

### 2-1. ルートの設定

#### 準備

- 先ほどの実装を、`<app>`タグから`<members>`タグに変更

```html
<members>
<h1>エンジニアチーム</h1>
<ul>
  <li each={ members } if={ active }>
    <h2>{ name }<small>（{ job }）</small></h2>
  </li>
</ul>
<hr>
Filter: <a onclick="{ filter }">All</a> - <a onclick="{ filter }" data-job="Backend">Backend</a> - <a onclick="{ filter }" data-job="Frontend">Frontend</a>
<hr>
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

var _this = this

_this.members = [
  { name: '濱田', job: 'Backend', active: true },
  { name: '青木', job: 'Frontend', active: true },
  { name: '二階', job: 'Backend', active: true },
  { name: '大石', job: 'Frontend', active: true },
  { name: '宇根', job: 'Backend', active: true },
  { name: '伊藤', job: 'Frontend', active: true },
]

this.add_member = function(e) {
  _this.members.push( {
    name: this.name.value,
    job:  this.job.value,
    active: true,
  } )
}

this.filter = function(e) {
  var job = $(e.target).data('job')
  _this.members.map(function(member) {
    member.active = member.job == job || !job
  })
}

</members>
```

- `<top>`タグを追加

```html
<top>
  <h1>中身はなんでも良い、とりあえず今は</h1>
</top>
```

- ナビゲーションを追加

```html
<div class="container">
  <div class="row">
    <div class="col-xs-8">
      <app></app>
    </div>
    <div class="col-xs-4">
      <nav></nav>
    </div>
  </div>
</div>
```

- `<nav>`タグを追加

```js
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

### 実装

- TOPページを作成

```js
riot.route('', function() {
  riot.mount('app', 'top') # <app>タグの位置に<top>タグをマウント
})
```

- riot.routeを有効にする
  - リロードしてみると、トップページが表示されるはず

```js
riot.route.start(true)
```

- Membersページを追加

```js
riot.route('members', function() {
	riot.mount('app', 'members')
})
```

#### 確認
- `#members`に遷移すると、先ほどのエンジニアチームのメンバー表が表示される
- `#`に遷移すると、先ほどのトップページが表示される

## 3. オブザーバブル

### 概要

### 実装

- `<top>`を書き換え

```
<top>
<h1>TOP</h1>
<span class="loading"><span id="outerCircle"></span></span>
<div each='{ companies }'>
  <h2><a href="{ permalink }" target="_blank">{ title }</a></h2>
  <p>{ job }</p>
</div>

var _this = this
    page = 1;

receiver.trigger('get', _this);

</top>
```

- `<script>`にオブザーバーを追加

```
// Observable

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
```

---

### 参考URL

- [Riot\.js — A React\-like user interface micro\-library · Riot\.js](http://riotjs.com/ja/)
- [Riot\.js 触ってみたメモとサンプル \| WebTecNote](http://tenderfeel.xsrv.jp/javascript/2220/)
