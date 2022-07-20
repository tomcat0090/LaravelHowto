# サービスコンテナ

> Laravelサービスコンテナは、クラスの依存関係を自動で管理したり、シングルトンクラスを簡単に使えるような仕組みを持つ強力なツールであり、Laravelのコアとなる機能で便利で魔法のような仕組みを実現してくれているものです。<br>
一言でいうと「使いたいクラスをインスタンス化してくれるマシーン」というイメージです。

> もう少し詳しく説明すると、コードアーキテクチャのデザインパターンである「Dependency Injection」の仕組みを使い、Laravelがコンポーネント間（interfaceとそれに依存する各class）の依存関係をプログラムのソースコードから排除し、使用すると定めた機能を持つclassを使いやすくしてくれています。

> なので、このセクションでLaravelサービスコンテナを学ぶ前にまず、Dependency Injectionを学びましょう。サービスコンテナの概念が理解できる近道だと思います。急がば回れ。下記のURLがパッとみた感じわかりやすいと思いました。

<br>

* Dependency Injectionの説明

1. [依存性の注入-Wikipedia](https://ja.wikipedia.org/wiki/依存性の注入)

2. [やはりあなた方のDependency Injectionはまちがっている。](http://blog.a-way-out.net/blog/2015/08/31/your-dependency-injection-is-wrong-as-I-expected/)

<br>

> サービスコンテナの仕組みや使い方を理解するのは大変ですが、まとまった説明をしてくれている記事です。<br>
> そんなに長い記事ではないので、順番に読んでみてください。


1. [サービスコンテナとは？２つの強力な武器を持ったインスタンス化マシーン。簡単に解説。](https://qiita.com/minato-naka/items/afa4b930a2afac23261b)

2. [Laravel サービスコンテナの理解を深める](https://reffect.co.jp/laravel/laravel-service-container-understand)

3. [Laravel 9.x サービスコンテナ](https://readouble.com/laravel/9.x/ja/container.html)

<br>

> 上記の記事たちを読んでいる方に向けて、[公式ドキュメント](https://readouble.com/laravel/9.x/ja/container.html)を参考にしながら自分なりにまとめてみます。

> まずは下記のサンプルコードを使って説明します。
~~~php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;
use App\Models\User;

class UserController extends Controller
{
    /**
     * Userリポジトリの実装
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * 新しいコントローラインスタンスの生成
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * 指定ユーザーのプロファイル表示
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
~~~
> この例では、UserControllerは$usersを取得する必要があるため、ユーザーを取得できるサービスを注入します。このようにしてサービスを注入できるということは、共に簡単に別のサービス（別の機能を持つUserRepository）に交換できます。この機能は強力で、大規模なアプリケーションを構築するために不可欠です。

> 別のコードパターンでさらに説明をします。<br>
> fileを保存するクラウド上のストレージサービスの選択肢が以下の2種類あるとします。<br>
> 何かのタイミングでfileを管理してくれるサービスをAzure Blob(azure)からS3(aws)へ変更するとします。<br>
> もし、Dependency Injectionまたは、サービスコンテナを利用しないとすれば、create_file()含めfileに対する各機能を持つ関数やクラスでAzure Blob(azure)を参照している変数などをすべて手作業で書き換えていかなければいけません。<br>
> しかし、サービスコンテナの機能を利用すると以下のようにコードを構築でき、サービス対象をS3(aws) へ変更する場合は、$this->app->bind(IStorageService::class, S3FileService::class); の一行を変更するだけで済みます。

- S3(aws) 
- Azure Blob(azure)

~~~php
interface IStorageService {
  createFile(stirng name);
  renameFile(string oldName, string newName);
  deleteFile(string name);
}
​
class S3FileService: IStorageService {
  createFile(stirng name) { ... }
  renameFile(string oldName, string newName) { ... }
  deleteFile(string name){ ... }
}
class AzureBlobFileService: IStorageService {
  createFile(stirng name) { ... }
  renameFile(string oldName, string newName) { ... }
  deleteFile(string name){ ... }
}
​
// IStorageSerive として、S3FileService を登録
$this->app->bind(IStorageService::class, S3FileService::class); 

// IStorageSerive として、AzureFileService を登録する場合
// $this->app->bind(IStorageService::class, AzureFileService::class); 
​

public class FileManager {
  private $_strageService;
​
  // コンストラクターインジェクション
  public function __consturct(IStorageService $storageServce)
  {
    $this->_strageService = $storageServce;
  }
​
  public function create_file()
  {
    $this->_strageService->createFile('new_file.txt');
    ...
  }
}
​
~~~
> この例のように、IStorageServiceがS3(aws)とAzure Blob(azure)に依存している場合、サービスコンテナの便利さがわかるかと思います。<br>
> 逆を言えば、このように依存しているサービスが複数個もなく、一対一の関係で依存している場合はサービスコンテナを使用する必要はありません。単に指定したクラスそのものをnewするだけで良い場合のことです。

<br>

## サービスコンテナを登録する

> サービスコンテナを登録する際は、サービスプロバイダーを利用します。<br>
> Laravelではサービスコンテナに登録されているサービスを利用してアプリケーションの開発を行なっていきます。サービスコンテナはサービスを入れる入れ物の役割をもっており、サービスを利用するためには、サービスコンテナに事前にサービスを登録しておく必要があります。そのサービスを登録する役目を持つものがサービスプロバイダーです。
> また、ほとんどのサービスプロバイダは、registerとbootメソッドを持っています。registerメソッドの中ではサービスコンテナへの登録だけを行わなくてはなりません。

> 以下のコマンドで新しいプロバイダを生成してください。example-app/Providersフォルダに生成されます。

~~~
sail artisan make:provider RiakServiceProvider
php artisan make:provider RiakServiceProvider
~~~

> では、基本的なサービスプロバイダを見てみましょう。サービスプロバイダメソッド中であれば、いつでも$appプロパティを利用でき、サービスコンテナへアクセスできます。(このセクションでは説明していませんが、下記のコードのようにsingletonとして登録できます。)

~~~php
namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * 全アプリケーションサービスの登録
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
}
~~~
> このサービスプロバイダはregisterメソッドのみを定義し、このメソッドを使用してサービスコンテナ内のApp\Services\Riak\Connectionの実装を定義します。

<br>

## プロバイダの登録

> すべてのサービスプロバイダは、config/app.php設定ファイルで登録されています。このファイルには、サービスプロバイダの名前をリストしてあるproviders配列が含まれています。この配列にはデフォルトとして、メール送信、キュー、キャッシュなどのLaravelコアのサービスプロバイダが登録されています。

>プロバイダを登録するには、この配列に追加します。

~~~
'providers' => [
    // Other Service Providers

    App\Providers\ComposerServiceProvider::class,
],
~~~

> また他の記事ですが、[こちら](https://reffect.co.jp/laravel/laravel-service-provider-understand)を読んでいただければ、サービスコンテナとサービスプロバイダーの関係性や実装方法がわかるかと思います。