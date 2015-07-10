# 05.ユーザ一覧の実装

`USERS`テーブルからIDの昇順に全件取得し、ユーザ一覧画面を表示します。

## ビュー

テンプレートは`views`パッケージに作成します。appディレクトリ配下に`views.user`パッケージを作成し、以下の内容で`list.scala.html`を作成します。

```html
@* このテンプレートの引数 *@
@(users: Seq[models.Tables.UsersRow])

@* main.scala.htmlを呼び出す *@
@main("ユーザ一覧") {

<div>
  <a href="@routes.UserController.edit()" class="btn btn-success" role="button">新規作成</a>
</div>

<div class="col-xs-6">
  <table class="table table-hover">
    <thead>
      <tr>
        <th>ID</th>
        <th>名前</th>
        <th>&nbsp;</th>
      </tr>
    </thead>
    <tbody>
    @* ユーザの一覧をループで出力 *@
    @users.map { user =>
      <tr>
        <td>@user.id</td>
        <td><a href="@routes.UserController.edit(Some(user.id))">@user.name</a></td>
        <td>@helper.form(routes.UserController.remove(user.id)){
          <input type="submit" value="削除" class="btn btn-danger btn-xs"/>
        }
        </td>
      </tr>
    }
    </tbody>
  </table>
</div>

}
```

> **POINT**
> * テンプレートの1行目にはコントローラから受け取る引数を記述します
> * テンプレートには`@`でScalaのコードを埋め込むことができます
> * テンプレートには`@*...*@`でコメントを記述することができます
> * リンクやフォームのURLは、`@routes.・・・`と記述することでルーティングから生成することができます

## コントローラ

`UserController`の`list`メソッドを以下のように実装します。

```scala
def list = Action.async { implicit rs =>
  // IDの昇順にすべてのユーザ情報を取得
  db.run(Users.sortBy(t => t.id).result).map { users =>
    // 一覧画面を表示
    Ok(views.html.user.list(users))
  }
}
```

`Action.async`はアクションの処理結果を`Future`で非同期に返却します。Slick 3.0ではSQLの実行処理を`Future`で返すことができるため、これを利用してPlayのアクションも`Future`でレスポンスを返すようにしています。

上記のコードでは以下の記述でユーザの一覧を取得するクエリを生成しています。

```scala
Users.sortBy(t => t.id).result
```

これは以下のSQLと同じ意味になります。

```sql
SELECT * FROM USERS ORDER BY ID
```

`db.run`で生成したクエリを実行してデータベースから結果を取得し、`map`メソッドで取得した値をテンプレートに渡しています。

このアクションの戻り値は「DBの検索結果をテンプレートに渡し、そのテンプレートのレンダリング結果をレスポンスとして返す`Future`」になります。

> **POINT**
> * Playの標準では`Action { ... }`の中に処理を記述しますが、レスポンスをFutureで返す場合は`Action.async { ... }`に処理を記述します
>   * `implicit rs`はアクションの処理の中でHTTPリクエストやDBのセッションを暗黙的に使用するために必要になる記述です
> * `Ok`に`views.html.・・・`と記述することで、表示したいHTMLのテンプレートを指定できます
>   * 引数にはテンプレートに渡すパラメータを指定します

## 実行

ここまで実装したらブラウザから http://localhost:9000/user/list にアクセスします（`activator run`を実行していない場合は実行してください）。すると以下のような画面が表示されるはずです。

![ユーザ一覧画面](images/user_list.png)

----
[＜ルーティングの定義に戻る](04_define_routing.md) | [ユーザ登録・編集画面の実装に進む＞](06_implement_user_form.md)