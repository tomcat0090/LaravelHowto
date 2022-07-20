# データベース

## 概要
> Laravelは、素のSQL、Fluentクエリビルダ、Eloquent ORMを使用し、サポートしている様々なデータベースの操作をとてもシンプルにしています。現在、Laravelは５つのデータベースのサポートを提供しています。

- MariaDB10.2以上 (バージョンポリシー)
- MySQL5.7以上 (バージョンポリシー)
- PostgreSQL10.0以上 (バージョンポリシー)
- SQLite3.8.8以上
- SQL Server2017以上 (バージョンポリシー)

<br>
<br>

# envファイルの設定

* envファイルとは
> .envファイルを使って開発環境と本番環境を切り替えたり、データベースなどの接続情報の変更を行うことができます。つまりLaravelにとって重要な設定変更.は.envファイルを介することで簡単に行うことができます。例えば開発環境では簡易的に利用できるsqliteのデータベースに接続し、本番環境ではmysqlのDBに接続といった変更が可能です。

<br>

* 設定例
> example-app/.env ファイルの中に記載されている下記項目を設定。
~~~
.......
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=example_app     =>データベースの名前
DB_USERNAME=root             =>元々はsailになっている
DB_PASSWORD=password
.......
~~~

> .env ファイルを上記のように設定したら、一度 sail stop し、もう一度 sail up -d をする。
DB_USERNAME=sail のまま sail up -d していた場合、mysql 内に入っても、ユーザーがsailでは権限がなく、何もできない。
設定を root に変更した状態でコンテナを再起動し、sail mysql コマンドでmysql 内に入る。
create database example-app コマンドで新たにデータベースを作成し、.env ファイルのDB_DATABASE=example_app の名前の部分は作成したデータベース名と一致させる。

<br>

* 繋がっているか確認してみる。
>example_app/resources/view/database.blade.php ファイルを生成し、以下のコードを貼り付けて保存。
~~~html
<strong>Database Connected: </strong>
    <?php
        try {
            \DB::connection()->getPDO();
            echo \DB::connection()->getDatabaseName();
            } catch (\Exception $e) {
            echo 'None';
        }
    ?>
~~~

>example_app/routes/web.php ファイルに下記を追記。

~~~php
Route::get('/database', function(){
    return view('database');
});
~~~

>ブラウザで localhost/database にアクセスし、（sail up -d でコンテナを起動しておいてください）
Database Connected: example_app
と出力されていれば接続OKです。

<br>
<br>

# database.phpの設定
* database.phpとは 
> config/database.phpではconnections 配列で上記のデータベース種類に応じて必要となる接続情報を定義するファイルです。<br>
> また、config/database.phpは、envファイルで定義された環境変数を参照するので、envファイルに設定した値が優先されます。

<br>

* 設定例

> mysqlデータベースを使用するとした設定は以下の通りです。

~~~
'mysql' => [
    'driver' => 'mysql',
    'url' => env('DATABASE_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'forge'),
    'username' => env('DB_USERNAME', 'forge'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
~~~

>上記を以下のように変更する
~~~html
'mysql' => [
    'driver' => 'mysql',
    'url' => env('DATABASE_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', '【DB名】'), 
    'username' => env('DB_USERNAME', '【DBユーザー名】'),
    'password' => env('DB_PASSWORD', '【DBパスワード】'),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
~~~
> 絶対に変更すべき箇所は次の3つです。

- DB_DATABASE
- DB_USERNAME
- DB_PASSWORD

<br>
<br>

# マイグレーション


* マイグレーションとは

> マイグレーションはデータベースのバージョン管理のようなもので、チーム内でのデータベースを管理するうえでの情報共有が円滑になります。<br>
> マイグレーションで何ができるかというと、テーブルの新規作成や削除などができ、そのテーブルを操作した経過手順をファイル単位で管理できます。

<br>

* マイグレーションの生成

> Artisanコマンド(make:migration)を使用して、データベースマイグレーションを生成します。新しいマイグレーションは、database/migrationsディレクトリに配置されます。各マイグレーションファイル名には、Laravelがマイグレーションの順序を管理しやすくするためにタイムスタンプ（生成した日時）を含めています。

~~~php
sail artisan make:migration create_sample_tables
php artisan make:migration create_sample_tables
~~~

> 上記のコマンドを実行すると、database/migrationsディレクトリに新しいマイグレーションファイルが生成されているのが確認できます。

>[補足1]<br>
create_sample_tablesの命名法はこの形が望ましく、これがファイル名になり、ファイル名には生成日時も含まれるため、「create_sample_tables 2022_07_15_095249」となり、「sampleというtablesが2022_07_15_095249にcreateされた」と誰が見てもなにをしているファイルか判断しやすいためです。

>[補足2]<br>
create_sample_tablesを例にすると、テーブル名となる"sample_tables"の部分は例のように必ず複数形にしてください。<br>
Laravelの使用上、テーブル名は複数形にしなければいけません。

>[補足3]<br>
create_sample_tablesのようにファイル名に"create_"を付けることによって、下記セクションのup, downメソッドの中身にテーブル作成と、id, timestampを定義するメソッドが定義された状態でファイルが生成されます。自分で書くコードが減り単純にこちらのほうが楽です。<br>

>[補足4]<br>
create_sample_tablesがファイル名だとすれば、Laravelは"create_"以降の"sample_tables"の部分が定義または作成するテーブル名だと判断し、下記セクションのup, downメソッド内で "Schema::create('sample_tables'..."と自動でテーブル名を定義してくれます。<br>



<br>

* マイグレーションファイルの説明

~~~php
return new class extends Migration
{
    /**
     * マイグレーションの実行
     *
     * @return void
     */
    public function up()
    {
        Schema::create('sample_tables', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * マイグレーションを戻す
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('sample_tables');
    }
};
~~~

> ・upメソッド<br>
> データベースに新しいテーブル、カラム、またはインデックスを追加するために使用します。<br>
> table->id();　table->timestamps();　が記述されていますが、これはテーブルカラム名でいうと、id, created_at, updated_at をテーブルのカラムとして追加することを表現しています。

> ・downメソッド<br>
> upメソッドによって実行した操作（テーブル生成）を戻すことがでます。ようはupメソッドで生成したテーブルを削除します。デフォルトではdrop関数が使用されているが、dropIfExists関数のほうがおすすめかも。

> Laravelでは、データベースの値を取得したり操作したりする際に、このid, created_at, updated_at カラムを頼りに多機能で簡単な操作(Eagerロードなど)を可能にしてくれるため、そのような機能を使用する場合は定義しておいたほうがいいです。

<br>

* 新たにカラムを定義➀（マイグレーション実行前のファイルに対して）

> マイグレーションファイルを作成し、そのファイルがまだマイグレーションされていない場合は以下のようにコードを追加するだけです。マイグレーションした後、既存のテーブルにカラムを追加する場合は、新たにマイグレーションファイルを作成しなければいけません。チーム間などでデータベースの管理をしやすくするためです。

~~~php
$table->id();
$table->string('name')->nullable();
$table->integer('age');
$table->timestamps();
~~~

> カラムのデータ型の指定は例のコードでいうと、string, integerの部分になります。<br>
> [その他にたくさんあるのでこちらを参照してください。](https://qiita.com/Otake_M/items/3c761e1a5e65b04c6c0e)

> ->nullable()はカラムタイプの指定をしています。nullableはカラム値が空（null）でも大丈夫だと表現しています。<br>
> その他のカラムタイプは公式ドキュメントの[このページ](https://readouble.com/laravel/8.x/ja/migrations.html)上で Ctrl+F コマンドで「null」と検索すれば一覧にヒットします。

<br>

* 新たにカラムを定義➁（マイグレーション実行後のテーブルに対して）

> 一度マイグレーションしたファイルを編集してカラムを押すさすることはお勧めしません。テーブル情報の変更を追いにくくなり、管理しにくくなります。なので、カラムを追加する場合は新たにマイグレーションファイルを作成し、そこで新たなカラムを定義します。

~~~
sail artisan make:migration add_email_to_sample_tables
php artisan make:migration add_email_to_sample_tables
~~~

> ファイル名は、そのファイルで行う操作を表現した名前にすべきです。ここでは'email'というカラムをsampleテーブルにaddしています。

~~~php
return new class extends Migration
{
    /**
     * マイグレーションの実行
     *
     * @return void
     */
    public function up()
    {
        Schema::table('sample_tables', function (Blueprint $table) {
            $table->string('email'),
        });
    }

    /**
     * マイグレーションを戻す
     *
     * @return void
     */
    public function down()
    {
       Schema::table('sample_tables', function (Blueprint $table) {
            $table->dropColumn('email'),
        });
    }
};
~~~

> up, downメソッド内で使用している関数がテーブル作成時のものと違うことに注意してください。

<br>

* マイグレーションの実行

> 以下のコマンドを実行すれば、まだ実行されていないマイグレーションファイルをLaravelが探し、そのファイルを実行し、テーブルを作成します。<br>
> なので、もし全てのマイグレーションファイルが既に実行された状態でコマンドを実行しても、「nothing to migrate」と表示され何も起こりません。

~~~
sail artisan migreate
php artisan migrate
~~~

<br>

* 便利なmigrateコマンド

> 前回のマイグレーションの実行結果を無効にする。Undo的な機能。
~~~
sail artisan migrate:rollback
~~~
<br>

>すべてのマイグレーション実行結果を無効にする。
~~~
sail artisan migrate:reset
~~~
<br>

> すべてのマイグレーションの実行を無効にしてから、もう一度グレージョンを実行します。<br>
> データベース全体を効果的に再作成します。
~~~
sail artisan migrate:fresh
~~~

<br>
<br>

# シーダー

* シーダーとは
  
> データベースに初期データを登録するための機能です。<br>
最初から用意しておきたいマスターデータやテスト用のデータを作成するときに使えます。

<br>

* シーダーファイルの生成
>Artisanコマンドを使用して生成します。シーダーファイルはすべてdatabase/seedersディレクトリに設置します。
~~~
sail artisan make:seeder SampleSeeder
~~~

<br>

* 生成するデータの定義方法

> シーダーファイルのrun()メソッドの中に、以下のように定義して下さい。
~~~php
class DatabaseSeeder extends Seeder
{
    /**
     * データベースに対するデータ設定の実行
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => Hash::make('password'),
            'age' => 19,
        ]);
    }
}
~~~
> table('users') でデータを挿入するテーブルを指定します。

<br>

* 予備知識
> ここでは詳しく説明しませんが、大量のデータを生成したい場合は、ファクトリー（Factory）という機能を使うことをおすすめします。<br>
必要であれば調べてみてください。

<br>


* シーダーファイルを実行するための設定
> database/seeders/DatabaseSeeder ファイルのrun()メソッドの中に、先ほどコマンドで生成したシーダーファイルのクラスを以下のように指定します。
~~~php
public function run()
{
    $this->call([
        SampleSeeder::class,
        AnotherSeeder::class,
    ]);
}
~~~ 

<br>

* シーダーの実行
> 以下のコマンドでシーダーを実行し、初期データが定義したテーブルに追加されます。
~~~
sail artsian db:seed
~~~

<br>
<br>

# モデル
> このセクションでは、Artisanコマンドでモデルファイルを作成し、モデルに紐づくテーブル情報の定義方法や、データを特定のテーブルから取得する変数やメソッドの定義方法をまとめます。

<br>

* モデルとテーブルの紐づけ方法➀　（手動で紐づけ）
> 前のセクションではマイグレーションでテーブルとカラムを定義し作成しました。<br>
> これからそのデータベース上のテーブルと、そのテーブルのレコードの追加と取得などのデータ操作をするためにモデルファイルを作成し、作成したモデルとテーブルを紐づけます。<br>
> まずArtisanコマンドでモデルファイルを作成します。

~~~
sail artisan make:model SampleTable
php artisan make:model SampleTable
~~~

> 上記のコマンドで作成されたモデルファイルは、example-app/Modelsフォルダに生成されます。

> 生成されたモデルファイルにprotected $table = 'テーブル名'を書けば、このモデルと指定したテーブルを紐づけられます。<br>
> また、Laravelの使用上、存在するテーブル名（複数形）とモデル名（単数形）が一致したものを自動で紐づけます。その場合は、protected $table = 'テーブル名'は定義しなくても大丈夫です。

~~~php
class SampleTable extends Model
{
     use HasFactory

    /**
     * モデルに関連付けるテーブル
     *
     * @var string
     */
    protected $table = 'sample_tables';
}
~~~

<br>

* モデルとテーブルの紐づけ方法➁　（Laravelが自動で紐づけ）

> モデルファイルをArtisanコマンドで作成する際、-mオプションを付ければ、指定した名前のモデルファイル、そのモデルファイル名の複数形の名前のテーブル作成マイグレーションファイル（いい感じに定義されたup, downメソッド付き）が同時にすべて生成されます。

~~~
sail artisan make:model SampleTable -m
php artisan make:model SampleTable -m
~~~
> 上記のコマンドを実行すれば、-mオプションにより下記のようなモデルファイルとマイグレーションファイルが同時に生成されます。
~~~php
class SampleTable extends Model
{
    use HasFactory;
}
~~~
~~~php
return new class extends Migration
{
    /**
     * マイグレーションの実行
     *
     * @return void
     */
    public function up()
    {
        Schema::create('sample_tables', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * マイグレーションを戻す
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('sample_tables');
    }
};
~~~

<br>

# レコードの追加
* データベースにレコードを追加する準備

>　まず、モデルファイルにレコードの代入が可能なカラムを定義します。<br>
ここでは、前のセクションのマイグレーションファイルで定義したカラムを例に挙げます。

~~~php
class Flight extends Model
{
     use HasFactory;

    /**
     * 複数代入可能な属性
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'age',
    ];
}
~~~
> これで、name, ageカラムに新たなレコードを挿入する準備が整いました。

<br>

* データベースにレコードを追加する
> 新たにレコードを挿入するタイミングは、ユーザーからフォームが送信された時など、何かしらアクションがあった時にコントローラーで行う場合がほとんどだと思います。なので、下記ではコントローラーのメソッド内でレコードを挿入する例を挙げてみます。

~~~php
class ExampleController extends Controller
{
    /**
     * 新しいフライトをデータベースに保存
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $sample_table = new SampleTable;

        $sample_table->name = $request->name;

        $sample_table->age = $request->age;

        $sample_table->save();
    }
}
~~~
> 上記のように、SampleTableをインスタンス化し、name, age変数に新たなモデルを代入し、save()でSampleTableに紐づけられたsample_tablesテーブルにモデルを保存します。<br>
> もしくは、createメソッドを使用して、新しいモデルを保存することもできます。createメソッドは、その挿入したモデルインスタンスを返します。
~~~php
class ExampleController extends Controller
{
    /**
     * 新しいフライトをデータベースに保存
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $sample_table = SampleTable::create([
            'name' => $request->name,
            'age' =>  $request->age,
        ]);

        $sample_table->save();
    }
}
~~~

<br>

# レコードを更新する

> saveメソッドを使用すれば、データベースにすでに存在するモデルを更新することもできます。モデルを更新するには、モデルを取得して、更新する値をセットします。次に、モデルのsaveメソッドを呼び出します。<br>
> 下記で登場するfind(), where(), update()メソッドなどの説明は、後のセクションで説明します。
~~~php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = 'Paris to London';

$flight->save();
~~~
> 複数更新もでき、特定のクエリに一致するモデルに対して更新を実行することもできます。この例では、「アクティブ（active）」でdestinationがSan Diegoのすべてのフライトが遅延（delayed）としてマークされます。
~~~php
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
~~~

<br>

# レコードを削除する

> モデルを削除するには、モデルインスタンスでdeleteメソッドを呼び出してください。
~~~php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
~~~

> 上記の例では、deleteメソッドを呼び出す前にデータベースからモデルを取得しています。しかし、モデルの主キーがわかっている場合は、destroyメソッドを呼び出して、モデルを明示的に取得せずにモデルを削除できます。
~~~php
Flight::destroy(1);

Flight::destroy(1, 2, 3);

Flight::destroy([1, 2, 3]);

Flight::destroy(collect([1, 2, 3]));
~~~

> Eloquentクエリを作成して、クエリの条件に一致するすべてのモデルを削除することもできます。この例では、非アクティブとしてマークされているすべてのフライトを削除します。一括更新と同様に、一括削除では、削除されたモデルのモデルイベントはディスパッチされません。
~~~php
$deleted = Flight::where('active', 0)->delete();
~~~

 <br>

* データベースのレコードをソフトデリートする
> データベースから実際にレコードを削除するだけでなく、モデルを「ソフトデリート」することもできます。モデルがソフト削除されても、実際にはデータベースから削除されません。代わりに、モデルに「deleted_at」属性がセットされ、モデルを「削除」した日時が保存されます。モデルのソフト削除を有効にするには、「Illuminate\Database\Eloquent\SoftDeletes」トレイトをモデルに追加します。
~~~php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
~~~

> まず、データベーステーブルにdeleted_atカラムを追加する必要があります。<br>
> 以下のように、マイグレーションファイルにsoftDeletes();を定義すれば大丈夫です。

~~~php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});

Schema::table('flights', function (Blueprint $table) {
    $table->dropSoftDeletes();
});
~~~

> これで、モデルのdeleteメソッドを呼び出すと、deleted_at列が現在の日付と時刻に設定されます。ただし、モデルのデータベースレコードはテーブルに残ります。ソフト削除を使用するモデルをクエリすると、ソフト削除されたモデルはすべてのクエリ結果から自動的に除外されます。

<br>

# レコードの取得

> まずはシンプルなクエリを実行し、モデルを取得する例を見ます。

~~~php
// すべてのモデルを取得
$flight = Flight::all()
~~~

~~~php
// whereで取得するモデルをactive=1の条件にあてはまるものと指定し、その後さらに取得するモデルを絞っていく
$flight = Flight::where('active', 1)
               ->orderBy('name')
               ->take(10)
               ->get();
~~~

~~~php
// 主キー（id）でモデルを取得
$flight = Flight::find(1);
~~~

~~~php
// クエリの制約に一致する最初のモデルを取得
$flight = Flight::where('active', 1)->first();
~~~

~~~php
// クエリの制約に一致する最初のモデルを取得する別の記法
$flight = Flight::firstWhere('active', 1);
~~~

> 上記で取得したモデルは、PHPの配列型ではなく、LaravelのIlluminate\Database\Eloquent\Collectionのインスタンスが返されます。<br>
> この\Collectionには様々な便利なメソッドがあり、例えば、rejectメソッドを使用して、呼び出されたクロージャの結果に基づいてコレクションからモデルを削除できます。

~~~php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function ($flight) {
    return $flight->cancelled;
});
~~~

> また、PHPのiterableなインターフェイスを実装しているため、コレクションを配列のようにループ処理できます。
~~~php
foreach ($flights as $flight) {
    echo $flight->name;
}
~~~

<br>

* メモリ節約しながらモデルを取得できる chunk(), cursor() メソッド

> まず、分かりやすくするために、SeederとFactoryを利用して、100,000件のMemberモデルとリレーション用にCommentモデルのレコードを作成したとします。

> まずは、普通に get() メソッドで取得してみます。

~~~php
// Memberモデルの取得。getメソッド。
$members = MemberModel::with('comments')->get();
dump(memory_get_usage() / (1024 * 1024)); // 316MB
dump(memory_get_peak_usage() / (1024 * 1024)); // 332MB
~~~

>  chunk() メソッドで取得してみます。

~~~php
MemberModel::with('comments')->chunk(1000, function ($members) {
    // DBからデータ取得後の処理
    foreach ($members as $member) {
        // 処理
    }
});
dump(memory_get_usage() / (1024 * 1024)); // 6MB
dump(memory_get_peak_usage() / (1024 * 1024)); // 9MB
~~~
> chunkメソッドを利用すると、クエリを複数回発行しながら、データを小分けにして取得可能です。
第一引数に取得するレコード数、第二引数に取得毎に行う処理をクロージャで指定出来ます。

<br>

# リレーションによるモデル取得

> 多くの場合、データベーステーブルは相互に関連（リレーション）しています。たとえば、ブログ投稿に多くのコメントが含まれている場合や、注文がそれを行ったユーザーと関連している場合があります。これらの関係の管理と操作を容易にし、少ないコードでとても効率の良いクエリを発行できます。

> 主に以下のようなテーブル関係を管理できます。

* １対１
* １対多
* 多対多
* Has One Through
* Has Many Through
* １対１（ポリモーフィック）
* １対多（ポリモーフィック）
* 多対多（ポリモーフィック）

> 例えば、以下のような１対多のリレーションで、あるユーザーのすべてのコメントを取得する際に、

~~~php
$user = User::find(1);
foreach($user->comments as $comment){
  // リレーション先のcommentsモデルにアクセスできます
}
~~~

> とせずに、以下のように取得できます。

~~~php
$user_comments = User::find(1)->comments;
~~~

<br>

## １対多のリレーションを定義してみる

> 上記の例のシチュエーション(UserモデルとCommentモデルのリレーション)を利用するとします。<br>
> まずUserモデルに以下のようにコードを定義します。

~~~php
class User extends Model
{
    /**
     * ブログポストのコメントを取得
     */
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
~~~

> これでリレーションの定義は終わりです。しかし、このLaravelリレーションの機能を成立させるために条件があります。

> 条件<br>
> Commentテーブルに、この例でいうと「user_id」という名前のカラムの存在が必須で、もちろんそのuser_idの値と実際にリレーション関係に値するUserモデルの「id」は一致している必要があります。<br>
> Laravelでは、リレーション関係になっている子テーブルには、必ず「親テーブルの名前_id」のカラムを含ませてください。

> これで後は、上記の例と同じように以下のコードで取得できます。

~~~php
$user_comments = User::find(1)->comments;
~~~

> このセクションでは、１対多のリレーションのみ紹介します。
> その他の1対1や多対多のリレーション定義は、[こちらの公式ドキュメント](https://readouble.com/laravel/9.x/ja/eloquent-relationships.html#one-to-one)を参照してください。公式ですが、それなりに分かりやすくまとまっています。

<br>

# Eagerロードによるモデル取得

> 