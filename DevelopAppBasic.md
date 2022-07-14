# プロジェクト構築編

## ルートについて

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

## アプリケーションをCSRF攻撃から守る
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

## コントローラーについて

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
