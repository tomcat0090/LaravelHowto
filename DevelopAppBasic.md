# プロジェクト構築編

# ルート

> Laravelでのルートを定義するデフォルトファイルは,
>example_app/routes/web.php ファイルです。
デフォルトでLaravelのwelcomeページを出力するルートが定義されてますが、これが一番シンプルなルート定義の例です。

<br>

* 基本の定義方法1
~~~php
Route::get('/', function () {
    return view('welcome');
});
~~~

> 上記のように定義されている場合、localhost/ をブラウザで開けば、example_app/resources/view/フォルダ内にある welcome.blade.php ファイルがブラウザ上で開かれます。

<br>

* 基本の定義方法2
~~~php
Route::get('/database', function(){
    return view('database');
});
~~~

> こう定義した場合、localhost/database をブラウザで開けば、example_app/resources/view/フォルダ内にある database.blade.php ファイルが開かれます。

<br>

* 基本の定義方法3
~~~php
Route::get('/user', [UserController::class, 'index']);
~~~
> このように定義すると、localhost/userをブラウザで開けば、example_app/Http/Controllers/フォルダ内のUserController.php のindex メソッドが実行されます。

<br>

* 基本の定義方法4
~~~php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
~~~
> 加えて、HTTPメソッドを定義する場合は上記のように定義できます。（$uri, $callback の部分は簡略化してあります）

<br>

* 基本の定義方法5
  
~~~php
Route::redirect('/here', '/there');
~~~
> リダイレクトは上記の形で定義できる。

<br>

* 基本の定義方法6

~~~php
Route::redirect('/here', '/there', 301);
Route::permanentRedirect('/here', '/there');
~~~
ステータスコードを返す場合は上記の形(2つとも同じ処理)で定義できる。

<br>

* 基本の定義方法7
~~~php
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
~~~

> 名前付きルートの定義ができ、->name('example_name'); を上記コードのように記述すれば、定義した名前を指定するだけでルートへのアクセスが可能になり便利です。

<br>

* その他のルート定義方法

[日本語版公式ドキュメントをURLを参照してください。](
https://readouble.com/laravel/9.x/ja/routing.html)

<br>

* 定義されているルートを確認する
~~~php
sail artisan route:list
php artisan route:list (sail未使用の場合)
~~~
> 上記のコマンドを実行すれば、現在定義されているルートの一覧を確認できます。

<br>

# アプリケーションをCSRF攻撃から守る
~~~html
<form method="POST" action={{ route("profile") }}>
    @csrf
　　............
    ............
</form>
~~~
> アプリケーションで"POST"、"PUT"、"PATCH"、"DELETE" HTMLフォームを定義する際、悪意あるユーザーからのCSRF攻撃を防ぐために <span style="color:red;">@csrf</span> を記述しなけらばいけません。
記述されていない場合はエラーになります。詳しくは[こちら](https://readouble.com/laravel/9.x/ja/csrf.html
)を確認してください。

<br>


* APIルートを定義する
> LaravelでAPIルートを定義する際は、上記での定義方法と少し異なります。まず初めに、APIルートを定義する場合は、example-app/routes/api.php ファイルに定義します。このファイル内には以下のようになっていると思います。
~~~php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
~~~

> すでに定義されているルートの中に埋め込む形でAPIルートを以下のように定義でき、コントローラーへ処理を渡せます。
~~~php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    Route::post('/setvalue', [ApiController::class, 'store'])
    return $request->user();
});
~~~
> 定義したAPIルートへアクセスする場合、web.phpで定義されたルートへは、[http://localhost:8000/setvalue]ですが、APIルートの場合は、[http://localhost:8000/api/setvalue]でアクセスできます。

<br>

# コントローラー
<br>

* 概要
> Laravelでは、ルートのリクエスト処理はすべてコントローラークラスで行います。コントローラークラスは、app/Http/Controllers ディレクトリ内に存在します。コントローラークラスはartisanコマンドで生成しますが、自動的にこのディレクトリに割り振られます。

<br>

* コントローラーの生成
~~~
sail artisan make:controller ExamplController
php artisan make:controller ExamplController
~~~
> 上記のコマンドでコントローラークラスを生成できます。

<br>

* もっと便利なコントローラーを生成するコマンド

~~~php
sail artisan make:controller ExamplController --resource
php artisan make:controller ExamplController --resource
~~~

> 上記のコマンドでコントローラーを生成すると、index, create, store, show, edit, update, destroy メソッドが定義されている状態で生成され、自分で０から定義する必要がなくなります。
また、--resource オプションで生成したコントローラークラスのメソッドの呼び出しルートの定義も便利で、以下のように定義できます。

~~~php
Route::resource('example', ExampleController::class);
~~~

> このように定義した場合、各メソッドへは以下のようにアクセスできます。この例では、ExampleControllerクラスのstoreメソッドを呼び出している。どのような記述でどのメソッドを呼べるかはコードしたのテーブルを参照してください。
~~~html
<form method="POST" action={{ route("example") }}>
    @csrf
　　............
    ............
</form>
~~~



| 動詞      | URI                  | アクション| ルート名 |
|:---:|:---:|:---:|:---:|
| GET       | /photos              | index   | photos.index   |
| GET       | /photos/create       | create  | photos.create  |
| POST      | /photos              | store   | photos.store   |
| GET       | /photos/{photo}      | show    | photos.show    |
| GET       | /photos/{photo}/edit | edit    | photos.edit    |
| PUT/PATCH | /photos/{photo}      | update  | photos.update  |
| DELETE    | /photos/{photo}      | destroy | photos.destroy |

[さらに詳しいルートの説明はこちらの公式ドキュメントを参照してください](https://readouble.com/laravel/9.x/ja/controllers.html)


<br>

* 実際に定義してみる
> 例えば、あるコントローラークラスの class UserController{} 内に以下のコードを記述する。
~~~php
 public function show($id)
{
    return view('user.profile', [
        'user' => User::findOrFail($id)
    ]);
}
~~~
> このメソッドにアクセスする場合は、app/routes/web.php ファイル内に以下のように定義し、

~~~php
Route::get('/user/{id}', [UserController::class, 'show']);
~~~
> blade ファイルから以下のように route メソッドをつかって呼び出せる。
~~~html
<form method="get" action="{{ route("user/3") }}">
    @csrf
~~~

<br>

* Laravelでのマジックメソッドの使い方
> コントローラークラス内に記述できる代表的なマジックメソッドの使用例を記載します。

1. __construct()
>　新たにオブジェクトが 生成される度にこのメソッドをコールされ、そのオブジェクトを使用する前に必要な初期化を行うことができます。要は、new したときに呼ばれます。使用例は、ルーティングでUserController内のメソッドが指定されていれば必ず、オブジェクトの生成がされるので
複数のメソッドで使う、DIだったりミドルウェアの適応等に適しています
~~~php
class UserController extends Controller
{
    public $service;
    public function __construct(\App\Services\UserService $service)
    {
        $this->middleware('auth');

        $this->middleware('log')->only('index');

        $this->service = $service;
    }
}
~~~

<br>

2. __destruct()
>  特定のオブジェクトを参照するリファレンスがひとつもなくなったときにコールされます。 あるいは、スクリプトの終了時にも順不同でコールされます。
要は、オブジェクトが破棄される際や、スクリプトが終わったら実行されます。使用例は、ファイルのポインタ開いたのをを閉じたり、GuzzleとかCurlHttpClient等ではつかってるぽいです
~~~php
class SampleFileOpen
{
    public $handle;
    public function __construct($user,$pass)
    {
        // ファイル開く
        $this->handle = fopen('somefile.txt', 'r');
    }

    public function __destruct()
    {
        fclose($handle);
    }
}

$file = new SampleFileOpen();
echo fgets($file->handle); //1行目が表示

// スクリプト終了したので、ファイルが自動で閉じられる
~~~

<br>

3. __invoke()
> スクリプトがオブジェクトを関数のように呼び出したときに起動します。
~~~php
class User
{
    protected $name = 'hoge';

    public function __invoke()
    {
        echo $this->name;
    }

    public function showName()
    {
        echo $this->name;
    }
}

$user = new User();
$user->showName(); // hoge
$user(); //hoge
~~~
> また、Laravelでは、invokeを利用した、シングルアクションControllerを使用できます。
~~~php
Route::get('user/{id}', 'ShowProfile');
~~~

~~~php
namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class ShowProfile extends Controller
{
    /**
     * 指定ユーザーのプロフィール表示
     *
     * @param  int  $id
     * @return View
     */
    public function __invoke($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
~~~
> [マジックメソッドについてのさらに詳しい情報はこちらからそうぞ](https://atmarkit.itmedia.co.jp/ait/articles/1804/05/news008.html)

<br>

# リダイレクト

## 他のURLへリダイレクト
> ユーザーを他のURLへリダイレクトさせるための定義方法

~~~php
---------------------------------------
Route::get('/dashboard', function () {
    return redirect('test/index');
});
---------------------------------------
public function update(Request $request)
{
    .....
    return redirect()->('test/index', 301);
}
---------------------------------------
~~~
> データと一緒にリダイレクトする

~~~php
return redirect('home')->with('result', '完了');
~~~

<br>

## 直前のURLへリダイレクト
> ユーザーを直前のURLへリダイレクトするときの定義方法

~~~php
return back();
~~~
> データと一緒に直前URLへリダイレクトする
~~~php
return back()->with('result', 'ok！');
return back()->withInput();     // 送信データがセッション内に格納される
~~~

> [上記以外にもいろんな定義方法があります。](https://qiita.com/manbolila/items/767e1dae399de16813fb)

<br>

## その他のリダイレクト

> コントローラアクションに対するリダイレクト

~~~php
return redirect()->action([UserController::class, 'index']);
~~~
> アプリケーション外のドメインへリダイレクト
~~~php
return redirect()->away('https://www.google.com');
~~~

>[上記以外のリダイレクト方法をさらに詳しくはこちら](https://readouble.com/laravel/9.x/ja/responses.html)

<br>

# View

> ビューを生成するファイルは、example-app/resources/views フォルダで管理します。
このフォルダ内に .blade.php 拡張子のついたファイルを配置すれば、ビューを生成できます。<br>
.blade.php拡張子は、ファイルにBladeテンプレートが含まれていることを表現しており、Bladeテンプレートでは、"if"文や"foreach"文、"error"文を簡単に定義できます。<br>
Bladeテンプレートについて詳しくは次項で説明します。

<br>

## 一番シンプルなビュー生成
~~~html
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello People</h1>
    </body>
</html>
~~~

> 上記のようなgreeting.blade.phpファイルのビューを生成する場合は以下のように定義でき、.blade.phpファイルの名前（greeting）を指定すればよい。

~~~php
---------------------------------------
Route::get('/greet', function () {
    return view('greeting');
});
---------------------------------------
public function update(Request $request)
{
    .....
    return view('greeting');
}
---------------------------------------
~~~

<br>

## データと一緒にビュー生成
~~~html
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
~~~

> 上記のようなデータが必要なgreeting.blade.phpファイルのビューを生成する場合は以下のように定義できる。
> 
~~~php
---------------------------------------
Route::get('/greet', function () {
    return view('greeting', ['name' => 'James']);
});
---------------------------------------
public function update(Request $request)
{
    .....
    return view('greeting', ['name' =>  $request->name]);
}
---------------------------------------
~~~

<br>

# Bladeテンプレート
> Bladeは、Laravelに含まれているシンプルでありながら強力なテンプレートエンジンです。<br>
> 前項ですでに述べましたが、Bladeテンプレートファイルは.blade.phpファイル拡張子を使用し、通常はresources/viewsディレクトリに保存します。<br>
> Bladeファイルの表示方法も前項で述べているので、そちらを参考にしてください。

<br>

## ディレクティブ
> BladeはPHPのif文、 foreach文、 switch文のショートカットを提供しています。こうしたショートカットは、非常にクリーンで簡潔なコーディング方法を提供する一方で、慣れ親しんだ同等のPHP構文も生かしています。

<br>

* 生PHP
> 状況によっては、PHPコードをビューに埋め込める。
~~~php
@php
    $counter = 1;
@endphp
~~~

<br>

* コメント
> ビューにコメントを定義することができます。
~~~php
{{-- このコメントはHTMLのなかに存在しない --}}
~~~

<br>

* ifディレクティブ

> @if、@elseif、@else、@endifディレクティブなどや他にも、PHPの構文と同じように機能します。
~~~php
@if (count($records) === 1)
    // １レコードあります。
@elseif (count($records) > 1)
    // 複数レコードあります。
@else
    // レコードがありません。
@endif

@unless (Auth::check())
    // あなたはサインインしていません。
@endunless

@isset($records)
    // $recordsが定義済みで、NULLではない…
@endisset

@empty($records)
    // $recordsは「空」
@endempty
~~~

<br>

* 認証ディレクティブ
> @authおよび@guestディレクティブを使用すると、現在のユーザーが認証済みであるか、ゲストであるかを簡潔に判断できます。
~~~php
@auth
    // ユーザーは認証済み…
@endauth

@guest
    // ユーザーは認証されていない…
@endguest


@auth('admin')
    // ユーザーは認証済み…
@endauth

@guest('admin')
    // ユーザーは認証されていない…
@endguest
~~~

<br>

* Switchディレクティブ
> Switchステートメントは、@switch、@case、@break、@default、@endswitchディレクティブを使用して作成できます。
~~~php
@switch($i)
    @case(1)
        最初のケース…
        @break

    @case(2)
        ２番めのケース…
        @break

    @default
        デフォルトのケース…
@endswitch
~~~

<br>

* 繰り返しディレクティブ
~~~php
@for ($i = 0; $i < 10; $i++)
    現在の値は、{{ $i }}
@endfor

@foreach ($users as $user)
    <p>このユーザーは：{{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>ユーザーはいません。</p>
@endforelse

@while (true)
    <p>無限ループ中です。</p>
@endwhile
~~~

<br>

> ループを使用する場合は、@continueおよび@breakディレクティブを使用して、現在の反復をスキップするか、ループを終了することもできます。
~~~php
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach


@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
~~~

<br>

* ループ変数
> foreachループの反復処理中、ループの内部では$loop変数を利用できます。この変数により、現在のループのインデックスや、ループの最初の反復なのか最後の反復なのか、といった便利な情報にアクセスすることができます。
~~~php
@foreach ($users as $user)
    @if ($loop->first)
        これが最初の繰り返しです。
    @endif

    @if ($loop->last)
        これが最後の繰り返しです。
    @endif

    <p>このユーザーは：{{ $user->id }}</p>
@endforeach
~~~
> また、ネストしたループ内にいる場合は、parentプロパティを介して親ループの$loop変数にアクセスできます。
~~~php
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            これは親ループの最初の繰り返しです。
        @endif
    @endforeach
@endforeach
~~~

<br>

> $loop変数は、他にもいろいろと便利なプロパティを持っています。

| プロパティ | 説明  |
| ----- | --- |
|$loop->index	|現在の反復のインデックス（初期値０）|
|$loop->iteration	|現在の反復数（初期値１）|
|$loop->remaining	|反復の残数。|
|$loop->count	|反復している配列の総アイテム数|
|$loop->first	|ループの最初の繰り返しか判定|
|$loop->last	|ループの最後の繰り返しか判定|
|$loop->even	|今回が偶数回目の繰り返しか判定|
|$loop->odd	|今回が奇数回目の繰り返しか判定|
|$loop->depth	|現在のループのネストレベル|
|$loop->parent	|ループがネストしている場合、親のループ変数|

<br>

* Methodフィールド
> HTMLフォームは<span style="color:red;">put、patch、delete</span>リクエストを作ることができないので、これらのHTTP動詞を偽装するために_Method隠しフィールドを追加する必要があります。@methodBladeディレクティブは、皆さんのためこのフィールドを作成します。
~~~php
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
~~~

<br>

> 上記以外に、Bladeテンプレートにはたくさんの機能があります。より詳しい情報は公式ドキュメントに詳しく掲載されています。[こちらから参照してください。](https://readouble.com/laravel/9.x/ja/blade.html)

<br>

