# バリデェーション

> バリデーションとは、入力データが正しいかどうかをチェックする機能です。Laravelでは、formなどからの受信データをバリデーションするための方法がいくつかあります。このセクションではもっとも一般的なvalidateメソッドを使用する方法を紹介します。また、validateメソッドはすべての受信HTTPリクエストで使用可能です。

<br>

## バリディレーションを定義する準備

> まず、routes/web.phpファイルに以下のルートを定義してあるとします。<br>
> GETのルートは新しいブログポストをユーザーへ表示し、POSTルートで新しいブログポストをデータベースへ保存します。
~~~php
Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
~~~

> 次に、これらのルートへの受信リクエストを処理する単純な以下のようなコントローラが定義してあるとします。今のところ、storeメソッドは空のままにしておきます。

~~~php
class PostController extends Controller
{
    public function create()
    {
        return view('post.create');
    }

    public function store(Request $request)
    {
        // ブログポストのバリデーションと保存コード…
    }
}
~~~

<br>

## バリデーションを定義する

> 上記のstoreメソッド内でバリデーションを定義します。<br>
> 'required|unique:posts|max:255' の部分がバリデートする条件になります。
> この指定した条件を満たさなかった値が1つでもあれば、エラーとなります。
~~~php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // ブログポストは有効
}
~~~

> 他にもいろいろバリデーションルールがあるので、[ことらのサイトを参照してみてください。]('required|unique:posts|max:255')

<br>

* エラー時の処理例1

> 受信リクエストがバリデーションルールにパスしない場合、Laravelはユーザーを直前の場所へ自動的にリダイレクトします。さらに、すべてのバリデーションエラーとリクエスト入力は自動的にセッションに一時保持保存されます。<br>
バリデーション機能でエラーになった場合、直前のビュー(blade.phpファイル)内で<span style = "color: red;">$errors</span>変数を定義し使用できます。<br>
この例では、バリデーションに失敗すると、エラーメッセージをビューで表示できるように、コントローラのcreateメソッドへリダイレクトされることになります。

~~~html
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Postフォームの作成 -->
~~~

<br>

  * エラー時の処理例2

> 以下の例では、@errorBladeディレクティブを使用して、特定の属性にバリデーションエラーメッセージが存在するかどうかを簡単に判断できます。@errorディレクティブ内で、$message変数をエコーし​​てエラーメッセージを表示できます。

~~~html
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
~~~

<br>

* エラー時の処理例3

> バリデーションによってエラーになり直前のビューに戻った際、フォームへ入力していた値がすべて消えていたら、ユーザーは再度入力しなくてはならず不便です。<br>
> そこでLaravelでは、oldメソッドを使用し入力されていたフォームの値を再取得できます。<br>
> 以下のように定義でき、エラーで直前のビューに戻っても、エラーではなかった値はフォームに入力された状態を保ちます。
~~~html
<input type="text" name="title" value="{{ old('title') }}">
~~~

> バリデーションの基本的な使用方法は以上です。他にもたくさん機能があるので気になる方は[こちらの公式ドキュメントを参照してください。](https://readouble.com/laravel/9.x/ja/validation.html)

<br>