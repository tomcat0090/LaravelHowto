# アセットバンドル

> これまでLaravelでのアセットバンドルツールはLaravel Mixでした。しかし、Laravelでのフロントエンドのアセットバンドルツールは、Viteが標準になると2022/6/29に公式発表されました。

<br>

## Viteとは
> Vue.jsなどに対応するフロントエンド開発ツールで、ネイティブESMによって要求時にファイルが提供されるのでバンドルが不要であり、HMR（hot module replacement）機能や、TypeScript／JSX／CSSのサポートなど、先進的な機能を備える。

> LaravelとViteの統合にあわせて、Laravel BreezeおよびLaravel Jetstreamも更新されており、Laravelの開発チームは、LaravelとともにViteの使用を開始するにあたって、フロントエンドとバックエンドの認証スキャフォールディング、およびTailwind、Inertia、Viteといったツールが提供されることから、Laravel Breezeの使用を推奨している。

<br>

## 実装方法

> LaravelでのViteの使用方法はまだ始まったばかりで、ネット情報もまだ少ないです。なのでここでは記事の紹介だけにします。2022/7/22現時点では、公式以外のLaravel Viteの使い方をまとめた記事はほとんどなく、あってもLaravel Mixをアンインストールしてから実装する自力感満載の記事だけです。

* [Vite Official Document](https://ja.vitejs.dev)
* [Laravel Official Document for Vite(日本語)](https://readouble.com/laravel/9.x/ja/vite.html)
*  [Laravel Vite Official Document](https://legacy.laravel-vite.dev/guide/usage.html)