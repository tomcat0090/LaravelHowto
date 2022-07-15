# データベース

## 概要
> Laravelは、素のSQL、Fluentクエリビルダ、Eloquent ORMを使用し、サポートしている様々なデータベースの操作をとてもシンプルにしています。現在、Laravelは５つのデータベースのサポートを提供しています。

- MariaDB10.2以上 (バージョンポリシー)
- MySQL5.7以上 (バージョンポリシー)
- PostgreSQL10.0以上 (バージョンポリシー)
- SQLite3.8.8以上
- SQL Server2017以上 (バージョンポリシー)

<br>

## envファイルの設定

<br>

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

## database.phpの設定

<br>

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




## モデル
//まずここでモデルの説明をし、-m オプションでマイグレージョン亜フィルを同時生成できることを伝え、マイグレージョンファイルのことは次項で説明すると明記する




## マイグレーション

<br>

* マイグレーションとは

> マイグレーションはデータベースのバージョン管理のようなもので、チーム内でのデータベースを管理するうえでの情報共有が円滑になります。<br>
> マイグレーションで何ができるかというと、テーブルの新規作成や削除などができ、そのテーブルを操作した経過手順をファイル単位で管理できます。

<br>

* マイグレーションの生成

> Artisanコマンド(make:migration)を使用して、データベースマイグレーションを生成します。新しいマイグレーションは、database/migrationsディレクトリに配置されます。各マイグレーションファイル名には、Laravelがマイグレーションの順序を管理しやすくするためにタイムスタンプ（生成した日時）を含めています。

~~~php
sail artisan make:migration create_sample_table --create=sample
php artisan make:migration create_sample_table --create=sample
~~~

> 上記のコマンドを実行すると、database/migrationsディレクトリに新しいマイグレーションファイルが生成されているのが確認できます。

>[補足1]<br>
create_sample_tableの命名法はこの形が望ましく、これがファイル名になり、ファイル名には生成日時も含まれるため、「create_sample_table 2022_07_15_095249」となり、「sampleというtableが2022_07_15_095249にcreateされた」と誰が見てもなにをしているファイルか判断しやすいためです。

> [補足2]<br>
> --create=sampleはオプションです。このオプションをつけてマイグレーションファイルを生成すれば、自動でテーブル名とそのテーブルを生成するコードを記載した状態でファイルが生成されます。<br>
> 以下のコードでいうと、up, down関数の中身すべてが該当します。
> オプションで指定したsampleというテーブル名がすでに記述されていることがわかります。<br>
> このオプションを付けずにファイルを生成すると、中身が空のup, downメソッドが記述されている状態になります。

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
        Schema::create('sample', function (Blueprint $table) {
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
        Schema::drop('sample');
    }
};
~~~

> ・upメソッド<br>
> データベースに新しいテーブル、カラム、またはインデックスを追加するために使用します。<br>

> ・downメソッド<br>
> upメソッドによって実行した操作（テーブル生成）を戻すことがでます。ようはupメソッドで生成したテーブルを削除します。

> --create=sample オプションでマイグレージョンファイルを生成したので、upメソッドの中に table->id();　table->timestamps();　が記述されていますが、これはテーブルカラム名でいうと、id, created_at, updated_at をテーブルのカラムとして追加することを表現しています。<br>

> Laravelでは、データベースの値を取得したり操作したりする際に、このid, created_at, updated_at カラムを頼りに多機能で簡単な操作(Eagerロードなど)を可能にしてくれるため、よほどの理由がない限りこのまま定義しておいたほうがいいです。

<br>

* 新たにカラムを定義

> 以下のようにコードを追加するだけです。

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

## シーダー
<br>

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