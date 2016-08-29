# Riot.js Handson

## 0. 概要

---

## 1. カスタムタグ

### 1-1. カスタムタグ

#### カスタムタグとは？

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

#### 実装



