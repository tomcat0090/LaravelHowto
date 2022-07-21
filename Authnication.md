# 認証の実装

> このセクションでは、ユーザーがアプリケーションで認証して「ログイン」する機能を実装します。<br>
> Laravelには認証機能を実装する手段がいくつかあり、スターターキットであるBreeze、より多機能で強力な認証機能を提供するJetstream、手動で実装するなどがあります。<br>
> このセクションでは、Breezeを使用して実装方法を紹介していきます。

<br>

## Breezeとは

>ログイン、ユーザー登録、パスワードのリセット、メールの検証、パスワードの確認など、Laravelのすべての認証機能を最小限シンプルに実装できるパッケージです。<br>



<br>

## インストール

> Composerを使用してLaravel Breezeをインストールするため下記のコマンドを実行する
~~~
sail composer require laravel/breeze --dev
php composer require laravel/breeze --dev
~~~

> インストールが完了したら、Artisanでのインストールを行います。
以下のコマンドにより、認証のためのビューやルート、コントローラなどが作成されます。
~~~
sail artisan breeze:install
php artisan breeze:install
~~~

> 下記のコマンドでCSSなどのアセットをビルドします。
~~~
sail npm install
sail npm run dev
~~~

<br>

> 以上で認証機能の実装の準備が終わりました。下記のURLでログインページにアクセスできます。

~~~
http://localhost/login
~~~

> 下記のURLで新しいユーザーを作成してみましょう。<br>
> 作成する前に、[sail artisan migrate]コマンドを実行して、usersテーブルを作成しておきます。Laravelには初期からusersテーブルを作成するマイグレーションファイルがあるので、userマイグレーションファイルを手動で生成する必要はありません。

~~~
http://localhost/register
~~~
> これでユーザ作成と同時にログインされ、ダッシュボード画面が表示されました。
> ちなみにユーザ情報は、先ほどのマイグレーション実行によって作成された「users」テーブルに追加されます。<br>
> [sail mysql]コマンドでデータベースにアクセスし、確認してみてください。

<br>

## 関連するファイルの確認

> 追加されたルーティングを知りたい場合はweb.phpファイルに新たにルーティングの行が追加されているので、routesディレクトリの下にあるauth.phpファイルを見ることで確認できます。

~~~php

Route::get('/', function () {
    return view('welcome');
});

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth'])->name('dashboard');

require __DIR__.'/auth.php';  //これが追加
~~~

> 認証に関するコントローラーはHttp¥Controllers¥Authの下に保存されています。

>ユーザ登録画面やログイン画面を変更したい場合は、resources¥viewsの下にあるauth、componentsディレクトリのBladeファイルを更新することで行うことができます。

>またユーザ登録時のバリデーションの内容などはapp¥Actions¥Fortifyフォルダの中のファイルで確認することができます。

> Breeze後に追加されたルーティングはルーティングファイルweb.phpと同じフォルダのroutes¥auth.phpファイルに記述されています。

<br>

> その他、ファイル構成や仕組みを知りたい場合は、[こちら](https://reffect.co.jp/laravel/laravel8-breeze)の詳しい記事を参照してください。<br>
> 日本語に対応させるやり方なども書いてあります。

<br>

## アクセスできるviewファイルに制限をかける
>現在、web.phpファイル内に定義されているrouteにいかのようなものが定義されているかと思います。

~~~php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth'])->name('dashboard');
~~~
>->middleware(['auth']) によってdashboard.blade.phpへはログイン中のユーザーしかアクセスできないと定義し、制限しています。逆を言えば、'guest'とすれば未ログインユーザーしかそこへはアクセスできません。<br>
この機能をミドルウェアと言います。

> また、以下のように、コントローラーの__construct()メソッド内に定義すれば、たとえばregisterを管理するコントローラーにそもそもアクセスできなくなります。

~~~php
class RegisterController extend Controller{

    public function __construct(){
        $this->middleware(['guest']); // 現在ログインしていないユーザーのみアクセスが可能
    }
}
~~~

<br>

## ログイン中のユーザーレコード取得

> コントローラーまたはbladeファイル内でログイン中のユーザー情報を使いたい場面などよくあると思います。<br>
その場合は以下のようなコードで簡単に取得できます。

~~~php
$my_user_record = auth()->user(); // 登録情報をすべて取得
$my_name = auth()->user()->name; // resisterで登録した名前を取得
~~~

>auth()->user()でUserモデルを取得できます。また、もしこれがコメントを投稿（post）できるアプリケーションだとした場合、Postモデルがあることが予想できます。UserモデルとPostモデルはリレーション関係なはずなので、以下のようにしてログイン中のユーザーに紐つくPostsモデルを取得できます。

~~~php
$my_post = auth()->user()->posts()
~~~
> これを可能にするためには、定義しなければいけない条件があります。

>[条件1]<br>
Postsモデル（postsテーブル）には、user_idカラムの存在が必須で、Laravelはこの名前（user_id）を探し、自動でログイン中のユーザーと紐づけてくれます。なので上記のコードのみで紐づくPostsモデルを取得できます。

>[条件2]<br>
Userモデルには、以下のようにPostモデルとのリレーションを定義してください。

~~~php
public function posts(){
    return $this->hasmany(Post::class);
}
~~~

> また、新たな投稿をデータベーステーブルに作成追加する際は、以下のようにして定義できます。

~~~PHP
class RegisterController extend Controller{

    public function store(Request $request)
    {
        $this->validate($request, [
            'body' => 'required'
        ]);

        $request->user()->posts()->create([
            'user_id' => auth()->user()->id,
            'body' => $request->body
        ]);

        return back(); 
    }
}
~~~
>back()で、直前のViewに返します。

<br>

## bladeファイル内でユーザーとゲスト処理を分ける

> 以下のように宣言すれば、ログインしていればhogehoge, していなければ（guest扱い）hogehogeなどの処理が簡単にできます。

~~~html
@auth
    <a ...>
    <input ...>
@endauth

@guest
    <a ...>
    <input ...> 
@endguest
~~~

<br>

> [こちらの動画](https://www.youtube.com/watch?v=MFh0Fd7BsjE&t=2669s)の中でも詳しく解説しているので、よかったら見てください。長い動画ですが、セクションごとに飛ばして視聴できるので、認証実装の部分を見るといいかと思います。