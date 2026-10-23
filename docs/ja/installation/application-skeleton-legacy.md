# 従来のアプリケーションスケルトン

[pieceofcake2/app-legacy](https://github.com/pieceofcake2/app-legacy) は、本家の CakePHP 2 と同じように
クラスがグローバルで、テンプレートが `.ctp` ファイルの CakePHP 2 アプリケーションのためのアプリケーション
スケルトンです。[pieceofcake2/app](application-skeleton) と同じ CakePHP 5 と互換性のあるディレクトリ構成
(`config/`、`src/`、`templates/`、`tests/`、`webroot/`)、同じ設定ファイルと `.env` ファイル、同じホームページ、
レイアウト、エラーページ、スタイルを持っています。違うのはコードの形だけです。

| | pieceofcake2/app-legacy | [pieceofcake2/app](application-skeleton) |
| --- | --- | --- |
| アプリケーションのクラス | グローバル (`AppController`) | `App` 名前空間 (`App\Controller\AppController`) |
| `src/` のオートロード | クラスマップ。クラスのファイルを追加したら `composer dump-autoload` を実行します | PSR-4 (`src/` から `App\`、`tests/` から `App\Test\`) |
| テンプレート | `Layouts/`、`Elements/`、`Emails/`、`Errors/` にある `.ctp` ファイル | `layout/`、`element/`、`email/`、`Error/` にある `.php` ファイル |
| コーディング規約 | `CakePHP2` | `CakePHP2Namespace` |

既存の CakePHP 2 アプリケーションと同じ形のコードにしたいとき、たとえばアプリケーションを少しずつスケルトンに
移すときに、このスケルトンを使ってください。クラスはあとから名前空間に移せます
([名前空間付きのクラスへの移行](#moving-to-namespaced-classes) を参照)。

> [!WARNING]
> CakePHP 2.x は既存アプリケーションのメンテナンスのためのものです。新しいプロジェクトには、
> CakePHP の現在のメジャーバージョンを使ってください。

## インストール {#installation}

Composer の `create-project` コマンドで新しいアプリケーションを作成します。

```bash
composer create-project pieceofcake2/app-legacy my_app
```

動作要件、インストールされるパッケージ、設定の手順は pieceofcake2/app と同じです。
[動作要件](application-skeleton#requirements) と [インストール](application-skeleton#installation) をご覧ください。
要点は、`config/database.php.default` を `config/database.php` にコピーして `config/.env.local` に `DATABASE_URL`
(テスト用データベースは `config/.env.test.local`) を設定し、セキュリティ用のソルトを変更することです
([セキュリティ用のソルトの生成](application-skeleton#generating-the-security-salts) を参照)。

Bake プラグインと DebugKit プラグインは開発用の依存パッケージです。`config/bootstrap.php` はインストールされていれば
これらを読み込むので、すぐに `bin/cake bake` が使え、本番環境では `composer install --no-dev` でも動作します。
DebugKit のツールバーを表示するには、[開発用プラグイン](application-skeleton#development-plugins) のとおりに
`AppController` にコンポーネントを追加します。`namespace` の行は不要です。

## ディレクトリ構成 {#directory-layout}

```text
my_app/
├── bin/
│   └── cake            the console
├── config/             configuration (core.php, bootstrap.php, routes.php, database.php, ...)
├── logs/               log files
├── plugins/            plugins installed by Composer
├── resources/
│   └── locales/        translation files
├── src/                PHP code
│   ├── Console/        shells, tasks and bake templates
│   ├── Controller/
│   ├── Lib/
│   ├── Model/
│   └── View/           helpers
├── templates/          view templates (.ctp)
│   ├── Elements/       elements (Flash/ for the flash messages)
│   ├── Emails/         email templates (html/, text/)
│   ├── Errors/         error pages
│   ├── Layouts/        layouts (Emails/ for the email layouts)
│   └── Pages/          pages of PagesController
├── tests/
│   ├── Fixture/
│   └── TestCase/
├── tmp/                cache, sessions and test files
├── vendor/             Composer packages, including CakePHP
└── webroot/            document root (index.php, test.php, css/, js/, img/)
```

CakePHP がディレクトリを見つける仕組みは [ディレクトリ構成](application-skeleton#directory-layout) と
[CakePHP のフォルダ構造](/ja/getting-started/cakephp-folder-structure) をご覧ください。

## クラスとオートロード {#classes-and-autoloading}

`src/` のクラスはグローバルで、Composer がクラスマップで読み込みます。

```json
"autoload": {
    "classmap": ["src/"]
}
```

クラスマップは `src/` のどのフォルダにあるクラスも見つけますが、`composer dump-autoload` の後でなければ
見つけられません。クラスのファイルを追加するたびに実行してください。CakePHP のクラスは、pieceofcake2/app と同じく
`use` 文でインポートします。

```php
// src/Controller/AppController.php
use Cake\Controller\Controller;

class AppController extends Controller
{
}
```

[オートロード](/ja/development/autoloading) をご覧ください。

## テンプレート {#templates}

テンプレートは、CakePHP 2 のディレクトリ名を使った `.ctp` ファイルです。

- `templates/Layouts/default.ctp`: 上部のナビゲーションバーとコンテンツのコンテナ。`debug` が 2 の間は、
  コンテンツの下に SQL ログが表示されます。
- `templates/Layouts/error.ctp`: 本番環境のエラーページ用の最小限のレイアウト。
- `templates/Layouts/flash.ctp`: `Controller::flash()` が描画するページ。
- `templates/Elements/Flash/`: `default`、`success`、`error`、`warning`、`info` のフラッシュメッセージ。
- `templates/Pages/home.ctp`: `debug` が有効な間 `/` に表示されるホームページ。`debug` を無効にする前に
  独自のページに置き換えてください。
- `templates/Errors/error400.ctp` と `error500.ctp`: エラーページ。

見た目は pieceofcake2/app のテンプレートと同じです。スタイルシートと、エラーページやフラッシュメッセージの
スクリーンショットは [ページとスタイル](application-skeleton#pages-and-styles) をご覧ください。

![ホームページ](/skeleton-home.png)

以前のバージョンのスケルトンから作成したアプリケーションで新しい見た目を使うには、`templates/Layouts/`、
`templates/Elements/Flash/`、`templates/Errors/`、`webroot/css/`、`webroot/font/` と `webroot/img/cake.logo.svg` を
アプリケーションにコピーしてください。

## 設定と開発 {#configuration-and-development}

そのほかは pieceofcake2/app と同じです。

- [設定](application-skeleton#configuration) と [環境変数 (.env)](application-skeleton#environment-variables-env)
- [データベースとメーラーの DSN](application-skeleton#database-and-mailer-dsns)
- [アプリケーションの実行](application-skeleton#running-the-application)
- [テストの実行](application-skeleton#running-tests) と [End-to-End テスト](application-skeleton#end-to-end-tests)。
  `tests/TestCase/` のテストケースと `tests/Fixture/` のフィクスチャはグローバルなクラスです。
- [静的解析](application-skeleton#static-analysis)

`vendor/bin/phpcs` は、[pieceofcake2/phpcs-cakephp2](https://pieceofcake2.github.io/phpcs-cakephp2/ja/) の
`CakePHP2` 規約でコードをチェックします
([アプリケーションの phpcs.xml](/ja/contributing/cakephp-coding-conventions#a-phpcs-xml-for-your-application) を参照)。

## 名前空間付きのクラスへの移行 {#moving-to-namespaced-classes}

このスケルトンから作成したアプリケーションを pieceofcake2/app の形に移すには、
[pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/ja/) の `layout` コマンドと `namespace` コマンドを
実行します。すでにモダン構成のアプリケーションでは、`layout` はテンプレートを CakePHP 5 のディレクトリにある
`.php` ファイルに名前を変え、`namespace` はクラスを `App` 名前空間に移します。
[モダン構成への移行](/ja/appendices/migrating-to-the-modern-layout) をご覧ください。

## 由来 {#origin}

このパッケージは [cakephp/cakephp](https://github.com/cakephp/cakephp) (Packagist では
`cakephp/cakephp`、アップストリームの最終バージョンは 2.10.24) の `app/` ディレクトリをもとに別のパッケージとして
切り出したもので、元の作者は CakePHP コミュニティ (Cake Software Foundation, Inc.) です。
[pieceofcake2](https://github.com/pieceofcake2) が、モダンな PHP で動く CakePHP 2 向けにメンテナンスしています。
オリジナルからの変更点は [コミット履歴](https://github.com/pieceofcake2/app-legacy/commits/2.x) で確認できます。
