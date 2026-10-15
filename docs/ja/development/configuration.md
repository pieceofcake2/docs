# 構成設定

CakePHP アプリケーションの設定は非常に簡単です。CakePHP の
インストールができたら、基本的な Web アプリを作り、それに必要な
データベース設定をセットアップしてみましょう。

もちろん CakePHP の柔軟なアーキテクチャの利点を活用するため、
いろいろなオプションの設定手順も用意されています。CakePHP
コアから継承された機能に簡単な追加を行なうことで、たとえば URL
マッピングルール(route)を追加／設定したり、単語の語尾変化
ルール(inflection)を追加／変更したりできます。

アプリケーションの設定ファイルは、プロジェクトの `config/` ディレクトリに置きます。

- `config/core.php` - デバッグレベル、エラー処理、セッション、セキュリティ用のソルト、
  内部キャッシュなどのコア設定
- `config/bootstrap.php` - アプリケーションの起動処理。キャッシュやログの設定、
  プラグインの読み込み、独自の inflection など
- `config/database.php` - データベース接続 (`config/database.php.default` をコピーして作成します)
- `config/routes.php` - URL のルーティング
- `config/email.php` - メール送信の設定 (`config/email.php.default` をコピーして作成します)
- `config/acl.php`、`config/acl.ini.php` - ACL の設定
- `config/Schema/` - データベースセッションや i18n 用のスキーマファイル

`CONFIG` 定数はこのディレクトリを指します。レガシー構成では同じファイルが `app/Config/`
にあり、プロジェクトのトップレベルに `config/` ディレクトリが無い場合は `CONFIG` がそちらを
指します。両方の構成については [CakePHP のフォルダ構造](/ja/getting-started/cakephp-folder-structure)
を参照してください。

<div class="index">

database.php, database.php.default

</div>

<a id="database-configuration"></a>

## データベース設定

CakePHP では、データベースの詳細設定は `config/database.php`
ファイルに記載されていることを前提としています。
`config/database.php.default` ファイルにデータベースの設定例が
記載されています。設定後の内容は、たとえば以下のようになります。

```php
// config/database.php
class DATABASE_CONFIG
{
    public $default = [
        'datasource' => 'Database/Mysql',
        'persistent' => false,
        'host' => 'localhost',
        'login' => 'cakephpuser',
        'password' => 'c4k3roxx!',
        'database' => 'my_cakephp_project',
        'prefix' => '',
        'encoding' => 'utf8mb4',
    ];
}
```

モデルの中で `$useDbConfig` プロパティが指定されない場合は、
\$default という名の接続用配列が使われます。たとえばあなたの
アプリケーションで、デフォルトのデータベース以外にも従来から
使っている(legacy)データベースを使う必要がある場合、モデルの中で
\$default 配列と同様の形式で新しいデータベース \$legacy を作り、
さらに適切なモデルの中で `public $useDbConfig = 'legacy';`
のように書きます。

設定配列の中のキー／値のペアを、
あなたの要求に合うように適切に設定してください。

アプリケーションスケルトンの `config/database.php.default` は、接続設定を環境変数から
読み込むこともできます。[環境変数](#environment-variables) を参照してください。

datasource  
この設定配列が使うデータソースの名前。
組み込みのデータベースドライバーは Database/Mysql、Database/Postgres、
Database/Sqlite、Database/Sqlserver です。いずれも PDO を使うので、対応する PDO 拡張
(`pdo_mysql`、`pdo_pgsql`、`pdo_sqlite`、`pdo_sqlsrv`) がインストールされている必要があります。
プラグインに含まれるデータソースを使う場合は `プラグイン記法`
という書式で指定します。

persistent  
データベースへの永続的接続を使うかどうか。Sqlserver データソースは永続的接続に
対応しておらず、true を指定すると例外が発生します。

host  
データソースサーバのホスト名（またはIPアドレス）

login  
データベースアカウントのユーザ名

password  
データベースアカウントのパスワード

database  
この接続で利用するデータベース名

prefix (*オプション*)  
データベース中のすべてのテーブル名の前にこの prefix を付加する。
prefix を使わない場合は空文字列のままにします。

port (*オプション*)  
サーバに接続するための TCP ポート番号または Unix ソケット。
SQL Server では、接続時にホスト名の後ろにポート番号が付加されます。

encoding  
接続で使用する文字コード。デフォルトはそのデータベースのデフォルト値です。
MySQL で UTF-8 を使いたい場合は、'utf8' または 'utf8mb4'（ハイフンなし）と指定してください。
文字コードは接続時に PDO の DSN で渡されます (MySQL は `charset=...`、PostgreSQL は
`options='--client_encoding=...'`)。そのため `SET NAMES` クエリは発行されません。
SQL Server では 'utf8' を指定するとドライバーの UTF-8 エンコーディングが使われます。

flags (*オプション*)  
PDO のコンストラクタに渡す PDO 属性の配列。PHP 8.1 以降、MySQL と SQLite の PDO
ドライバーは整数型と浮動小数点型のカラムを PHP の int と float で返します。
従来どおり文字列で受け取る必要がある場合は
`'flags' => [PDO::ATTR_STRINGIFY_FETCHES => true]` を指定してください。

schema  
使用するスキーマ。PostgreSQL (デフォルトは `public`) と
SQL Server (デフォルトは `dbo`) で使われます。

sslmode (*オプション*)  
PostgreSQL のみ。DSN に付加する `sslmode` の値です (例: `'require'`)。

options (*オプション*)  
SQL Server のみ。DSN に付加する接続オプションの配列です
(例: `['TrustServerCertificate' => 'yes', 'Encrypt' => 'no']`)。

::: pieceofcake2
Sqlserver データソースは SQL Server 2022 以降を対象としています。
ポートは `port` から読み取り、`schema` のデフォルトは `dbo` で、SSL/TLS の設定は `options`
配列で指定します。[CakePHP 2.10 からの移行](/ja/appendices/migrating-from-cakephp-2-10) を参照してください。
:::

unix_socket  
UNIX のソケットファイル経由での接続をサポートするドライバで使われます。
PostgreSQL を Unix ソケット経由で使いたい場合、ホスト(host)の設定値は
ブランクのままにしておきます。

ssl_key  
SSL キーファイルへのパス。`ssl_cert` と組み合わせて指定します（MySQL のみサポート）

ssl_cert  
SSL 証明書ファイルへのパス。`ssl_key` と組み合わせて指定します（MySQL のみサポート）

ssl_ca  
SSL CA ファイルへのパス（MySQL のみサポート）

settings  
キー／値のペアを持つ配列を指定すると、データベースサーバへの
接続が確立された後、これらが `SET` コマンドとしてサーバに
送られます。現時点では MySQL, Postgres, SQLserver のみでの
サポートです。

::: info Changed in version 2.4
`settings`, `ssl_key`, `ssl_cert`, `ssl_ca` キーは 2.4 で追加されました。
:::

> [!NOTE]
> prefix 設定はテーブルに対して適用されますが、モデルには **適用されません** 。
> たとえばあなたが自分の Apple and Flavor モデル用に JOIN テーブルを
> 作る場合、そのテーブル名は（prefix_apples_prefix_flavors **ではなく** ）
> prefix_apples_flavors とし、prefix 設定は 'prefix\_' としてください。

ここで、一度 [CakePHP の規約](../getting-started/cakephp-conventions)
を見ておくことをおすすめします。テーブル（およびいくつかのカラム）
に関する正確なネーミングルールを適用することで、自由に Cake の
能力を引き出すことができ、またいくつかの設定作業を省略できます。
たとえば、あなたのデータベーステーブルの名前を big_boxes 、
モデル名を BigBox 、コントローラー名を BigBoxesController
とすることで、すべての機能が自動的に有効になります。慣習として、
データベーステーブル名はアンダースコア、小文字、複数形にします。
たとえば bakers, pastry_stores, savory_cakes のようにします。

<a id="environment-variables"></a>

## 環境変数

デバッグレベル、セキュリティ用のソルト、データベースの認証情報など、環境ごとに異なる設定は、
設定ファイルに書く代わりに `env()` で環境変数から読み込めます。本番環境では、Web サーバー、
コンテナ、プロセスマネージャーなどの設定で環境変数をセットします。ローカルの開発環境では、
アプリケーションスケルトンが `.env` ファイルから環境変数を読み込めます。

<a id="loading-env-files"></a>

### .env ファイルの読み込み

[pieceofcake2/app](https://pieceofcake2.github.io/app/) スケルトンから作成したアプリケーションは、
`config/core.php` の先頭で [josegonzalez/dotenv](https://github.com/josegonzalez/php-dotenv)
を使って `.env` ファイルから環境変数を読み込みます。`config/core.php` は最初に読み込まれるので、
残りの設定、コンソール、テストで環境変数を使えます。

これは CakePHP 3〜5 のアプリケーションスケルトンにある `.env` サポートを CakePHP 2 に
バックポートしたものです。[cakephp/app](https://github.com/cakephp/app) は、同じライブラリを使って
`config/bootstrap.php` で `config/.env` を読み込みます。CakePHP 5 と同じく、フレームワーク自体は
`.env` ファイルを読み込まず、アプリケーションが読み込みます。

::: pieceofcake2
`.env` の読み込みは v2.4.0 から `pieceofcake2/app` スケルトンに含まれています。ほかの
アプリケーションについては、[既存のアプリケーションに .env サポートを追加する](#adding-env-support-to-an-existing-application)
を参照してください。
:::

スケルトンは、以下のファイルが存在すれば読み込みます。

1. `config/.env`。CakePHP 3〜5 のアプリケーションスケルトンが使う場所です。
2. プロジェクトルートの `.env`

優先順位は、実際の環境変数 > `config/.env` > プロジェクトルートの `.env` です。
`config/.env` を先に、ルートの `.env` を後に読み込み、実際の環境 (Web サーバー、シェル、
コンテナ、CI) や `config/.env` ですでにセットされている環境変数が上書きされることはありません。

読み込んだ値は文字列として `putenv()`、`$_ENV`、`$_SERVER` に書き込まれるので、`env()`、
`getenv()`、`$_ENV`、`$_SERVER` のどれでも読み出せます。

```php
// config/core.php
Configure::write('Security.salt', env('SECURITY_SALT') ?: 'a-default-salt');
```

まず、スケルトンに含まれるサンプルファイルをコピーして編集します。

```bash
cp config/.env.example config/.env
```

`config/.env` と `/.env` は、どちらもスケルトンの `.gitignore` に記載されています。`.env`
ファイルはバージョン管理に含めず、代わりに `config/.env.example` のようなサンプルファイルを
コミットしてください。

`.env` ファイルは開発用です。本番環境では、Web サーバー、コンテナ、プロセスマネージャーなどの
設定で実際の環境変数をセットしてください。実際の環境変数は常にファイルより優先されます。

<a id="env-file-syntax"></a>

### .env ファイルの書式

`.env` ファイルには、1 行に 1 つずつ `NAME="value"` の形式で値を書きます。Symfony や Laravel
でも使われている一般的な dotenv の書式です。`#` で始まる行はコメントです。パーサーは前に付けた
`export ` も受け付けますが、付ける必要はありません。

```dotenv
# config/.env
DEBUG="2"
SECURITY_SALT="a-long-random-string"
DB_HOST="127.0.0.1"
```

値は必ずクォートしてください。josegonzalez/dotenv は、クォートしない値を真偽値、数値、`null`
に変換します。`007` は `7` になり、暗号化シードのような長い数字の列はオーバーフローして別の数値に
なります。クォートしない値では、空白の後の `#` 以降がコメントになります。クォートした値はそのまま
使われます。ダブルクォートの中の `${NAME}` は、ファイル内でそれより前に定義された変数の値に置き換え
られます。

<a id="adding-env-support-to-an-existing-application"></a>

### 既存のアプリケーションに .env サポートを追加する

レガシーの `app/` 構成のアプリケーションや、以前のバージョンのスケルトンから作成した
アプリケーションにも、同じ読み込み処理を追加できます。まずライブラリをインストールします。

```bash
composer require josegonzalez/dotenv
```

次に、`app/Config/core.php` (モダン構成では `config/core.php`) の先頭に以下のコードを
追加します。`use` 文はファイル冒頭のコメントの後に、ループは最初の `Configure::write()`
の前に置きます。レガシー構成では `CONFIG` は `app/Config/`、`ROOT` は `app/` を含む
ディレクトリなので、読み込むファイルは `app/Config/.env` とプロジェクトルートの `.env` です。

```php
// app/Config/core.php

use josegonzalez\Dotenv\Loader;

foreach ([CONFIG . '.env', ROOT . DS . '.env'] as $dotenvFile) {
    if (!is_file($dotenvFile)) {
        continue;
    }
    $dotenv = new Loader([$dotenvFile]);
    foreach ($dotenv->parse()->toArray() as $key => $value) {
        if (getenv($key) !== false || isset($_ENV[$key]) || isset($_SERVER[$key])) {
            continue;
        }
        // The parser converts some values to booleans, numbers or null; keep them as strings.
        if (is_bool($value)) {
            $value = $value ? 'true' : 'false';
        }
        $value = (string)$value;
        putenv($key . '=' . $value);
        $_ENV[$key] = $value;
        $_SERVER[$key] = $value;
    }
    unset($dotenv, $key, $value);
}
unset($dotenvFile);
```

`.env` ファイルを `.gitignore` に追加し、次の節のように設定の中で `env()` を使って
環境変数を読み込んでください。

<a id="environment-variables-of-the-application-skeleton"></a>

### アプリケーションスケルトンの環境変数

`pieceofcake2/app` スケルトンの `config/core.php` と `config/database.php.default` は、
以下の環境変数を読み込みます。これらは `config/.env.example` に記載されています。

| 環境変数 | 設定 | セットされていない場合 |
|----------|------|------------------------|
| `DEBUG` | `debug`: `0`、`1`、`2` (`true` は 2、`false` は 0) | `2` |
| `SECURITY_SALT` | `Security.salt` | `config/core.php` の値 |
| `SECURITY_CIPHER_SEED` | `Security.cipherSeed` (1〜4294967295。[cipherSeed](#cipher-seed) を参照) | `config/core.php` の値 |
| `COOKIE_KEY` | `Cookie.key`。`'aes'` タイプの `CookieComponent` のキー ([`'aes'` への移行](#migrating-to-aes) を参照) | `null`。`CookieComponent` は `Security.salt` を使います |
| `DB_DATASOURCE`、`DB_HOST`、`DB_PORT`、`DB_USERNAME`、`DB_PASSWORD`、`DB_DATABASE` | `default` 接続の `datasource`、`host`、`port`、`login`、`password`、`database` | `config/database.php` の値 |
| `DB_TEST_DATASOURCE`、`DB_TEST_HOST`、`DB_TEST_PORT`、`DB_TEST_USERNAME`、`DB_TEST_PASSWORD`、`DB_TEST_DATABASE` | `test` 接続の同じ設定 | `config/database.php` の値 |

空の環境変数はセットされていないものとして扱われます。ただし、`DB_PASSWORD` と
`DB_TEST_PASSWORD` は空にできます。`.env` ファイルも環境変数も無い場合、アプリケーションは
設定ファイルの値を使います。スケルトンがこれらの環境変数を読み込むようになる前に
`config/database.php` を作成した場合は、`config/database.php.default` の `__construct()` と
`applyEnv()` メソッドをコピーしてください。

たとえば、開発用の `config/.env` は次のようになります。

```dotenv
# config/.env
DEBUG="2"
DB_HOST="127.0.0.1"
DB_USERNAME="my_app"
DB_PASSWORD="secret"
DB_DATABASE="my_app"
```

## クラスの読み込みと追加のクラスパス

CakePHP フレームワークのクラスは Composer のオートローダーで読み込まれるので、
`App::uses()` を書く必要はありません。アプリケーションのクラスも、
[pieceofcake2/app](https://pieceofcake2.github.io/app/) スケルトンのように
`composer.json` に classmap を追加すれば同じようにオートロードできます。

```json
"autoload": {
    "classmap": ["src/"]
}
```

こうしておけば、`src/` にあるコントローラー、モデル、ヘルパーなどのクラスに対する
`App::uses()` は削除できます。新しいクラスファイルを追加したら、classmap に含まれるように
毎回 `composer dump-autoload` を実行してください。Composer のオートロードに対応していない
プラグインでは、引き続き `App::uses()` が必要です
(例: `App::uses('MyPluginHelper', 'MyPlugin.View/Helper')`)。

::: pieceofcake2
フレームワークは include path ではなく Composer の classmap で
読み込まれるようになり、フレームワーク内の `App::uses()` は削除されました。
`App::uses()` を使っているアプリケーションのコードはそのまま動きます。
:::

`App::uses()` と `App::build()` は引き続き使えます。`App::uses()` を使った場合、
CakePHP はアプリケーションディレクトリ (`src/`、レガシー構成では `app/`) からクラスを探します。
テンプレートは `templates/`、ロケールは `resources/locales/`、プラグインは `plugins/` からも探します。

同一システム上の複数のアプリケーション間で MVC クラスを共有できると
有用な場合があります。両方のアプリケーションで同一のコントローラーを
持たせたい場合、CakePHP の `config/bootstrap.php` を使ってそれらの追加クラスを
読み込ませることができます。

`config/bootstrap.php` の中で `App::build()` を使うと、CakePHP
がクラスを検索する際の追加パスを定義できます。

```php
App::build([
    'Model' => [
        '/path/to/models',
        '/next/path/to/models',
    ],
    'Model/Behavior' => [
        '/path/to/behaviors',
        '/next/path/to/behaviors',
    ],
    'Model/Datasource' => [
        '/path/to/datasources',
        '/next/path/to/datasources',
    ],
    'Model/Datasource/Database' => [
        '/path/to/databases',
        '/next/path/to/database',
    ],
    'Model/Datasource/Session' => [
        '/path/to/sessions',
        '/next/path/to/sessions',
    ],
    'Controller' => [
        '/path/to/controllers',
        '/next/path/to/controllers',
    ],
    'Controller/Component' => [
        '/path/to/components',
        '/next/path/to/components',
    ],
    'Controller/Component/Auth' => [
        '/path/to/auths',
        '/next/path/to/auths',
    ],
    'Controller/Component/Acl' => [
        '/path/to/acls',
        '/next/path/to/acls',
    ],
    'View' => [
        '/path/to/views',
        '/next/path/to/views',
    ],
    'View/Helper' => [
        '/path/to/helpers',
        '/next/path/to/helpers',
    ],
    'Console' => [
        '/path/to/consoles',
        '/next/path/to/consoles',
    ],
    'Console/Command' => [
        '/path/to/commands',
        '/next/path/to/commands',
    ],
    'Console/Command/Task' => [
        '/path/to/tasks',
        '/next/path/to/tasks',
    ],
    'Lib' => [
        '/path/to/libs',
        '/next/path/to/libs',
    ],
    'Locale' => [
        '/path/to/locales',
        '/next/path/to/locales',
    ],
    'Vendor' => [
        '/path/to/vendors',
        '/next/path/to/vendors',
    ],
    'Plugin' => [
        '/path/to/plugins',
        '/next/path/to/plugins',
    ],
]);
```

> [!NOTE]
> 追加パスの設定は、すべて `bootstrap.php` の先頭で行ってください。
> こうすることで、アプリケーションのそれ以降の処理でこれらのパスが
> 有効になります。

<div class="index">

core.php, configuration

</div>

## コア（中核部）設定

CakePHP の各アプリケーションには、CakePHP 内部の振る舞いを決めるための
設定ファイルが含まれます。それは `config/core.php` です。
このファイルは設定用クラス変数や、あなたのアプリケーションが
どう振る舞うかを決めるための定数の定義の集まりです。それぞれの
変数を見て行く前に、まずは CakePHP の設定レジストリである
`Configure` を押さえておく必要があります。

### CakePHP のコア設定

`Configure` クラスは、CakePHP コアの設定用変数群を
管理するために使われます。これらの変数は `config/core.php`
にあります。それぞれの変数の定義や、それらがあなたの CakePHP
アプリケーションにどう影響するのかを以下に示します。

debug  
CakePHP のデバッグ出力を制御します。
0 = 本番モード。出力を行いません。
1 = エラーや警告を出力します。
2 = エラーや警告、および SQL を出力します。
\[ SQL のログは、ビューやレイアウト内で
\$this-\>element('sql_dump') をしている場合にのみ表示されます。 \]

アプリケーションスケルトンは、デバッグレベルを環境変数 `DEBUG` から読み込みます。
[環境変数](#environment-variables) を参照してください。

Error  
あなたのアプリケーションのエラーを処理するためのエラーハンドラー
を設定します。デフォルトでは `ErrorHandler::handleError()`
が使われます。これは、debug \> 0 の場合は `Debugger` を使ってエラーを表示し、
debug = 0 の場合は `CakeLog` でエラーをログに出力します。

サブキー:

- `handler` - callback - エラーを処理するためのコールバック関数。
  これには無名関数を含む、あらゆるタイプのコールバックを指定可能です。
- `level` - int - 捕捉したいエラーのレベル。
- `trace` - boolean - ログファイルにエラーのスタックトレースを出力。

Exception  
捕捉されていない例外に対して使われる、例外ハンドラーを設定します。
デフォルトでは ErrorHandler::handleException() が使われます。
これは例外の HTML ページを表示し、さらに debug \> 0 の場合であれば、
コントローラーがないといったフレームワークのエラーも表示します。
debug = 0 の場合、フレームワークのエラーは強制的に一般的な HTTP
レベルのエラーに変換されます。例外ハンドリングに関する詳細は
[Exceptions](exceptions) セクションを参照してください。

<div id="core-configuration-baseurl">

App.baseUrl  
もしあなたのサーバで mod_rewrite （や、その他の互換モジュール）
を使いたくない、もしくは使えない場合、CakePHP 組み込みの優れた
URL のしくみを使うことになります。 `config/core.php`
で以下のような行のコメントを外して有効にしてください。

```php
Configure::write('App.baseUrl', env('SCRIPT_NAME'));
```

さらに、`webroot/.htaccess` ファイルを削除します。レガシー構成の場合は、
以下の `.htaccess` ファイルを削除します。

    /.htaccess
    /app/.htaccess
    /app/webroot/.htaccess

ドキュメントルートは `webroot/` (`app/webroot/`) に設定したままにしてください。
`.htaccess` ファイルがない状態でドキュメントルートがプロジェクトのディレクトリを指していると、
設定やログ、ソースコードが公開されてしまいます。[デプロイ](../deployment#set-document-root) を参照してください。

これであなたの URL は、たとえば
www.example.com/controllername/actionname/param
ではなく
www.example.com/index.php/controllername/actionname/param
として扱われます。

もし CakePHP を Apache 以外の環境にインストールしようとしている
場合、それぞれのサーバで URL 書き換えを行なう方法について、
[Web サーバーの設定](../installation/url-rewriting) に記載があります。

App.fullBaseUrl  
アプリケーションのルートへの完全修飾ドメイン名 (プロトコルを含む)。
アプリケーション内の任意の URL 生成に特定のドメイン URL を使用するように
CakePHP を設定するには、この設定変数をセットしてください。
これにより、完全なベース URL の自動検出が無効になり、CLI （メール送信など）
からのリンクの生成に役立ちます。アプリケーションがサブフォルダーで実行されている場合は、
`App.base` も設定する必要があります。

App.base  
app のあるベースディレクトリー。アプリケーションがサブフォルダーで実行され、
`App.fullBaseUrl` が設定されている場合に使用する必要があります。

App.encoding  
あなたのアプリケーションで使用しているエンコーディングを定義します。
このエンコーディングは、レイアウトの中の charset やエンティティ
のエンコードを生成するのに使われます。この指定はあなたのデータベース
で使用しているエンコーディングと一致させる必要があります。

Routing.prefixes  
admin のような CakePHP の接頭辞(prefix)ルーティングを活用
したい場合はコメントを外して有効にします。この変数には使用したい
ルーティングの接頭辞名の配列を指定します。詳細は後述します。

Cache.disable  
true にすると、サイト全体のキャッシュを無効にします。
これを行なうと、 `Cache` に対するすべての
read/write は失敗します。

Cache.check  
true にすると、ビューのキャッシュを有効にします。さらにコントローラー
の中でも指定する必要がありますが、この変数はこれらの設定の検出
を行います。

Session  
セッション機能に関して使われる設定の配列です。デフォルトのプリセット
定義として 'defaults' キーが使われますが、ここで宣言されている設定は
デフォルトの設定を上書きします。

サブキー

- `cookie` - 使用するクッキーの名前。デフォルトは 'CAKEPHP'。

- `timeout` - セッションの有効期間（単位：分）。
  この timeout は CakePHP が制御します。

- `cookieTimeout` - セッションクッキーを有効にしたい期間（単位：分）。

- `checkAgent` - セッションの開始時にユーザーエージェント文字列を
  チェックするかどうか。セッションの途中でユーザーエージェント文字列が
  変わるクライアントに対応する場合は false にしてください。

- `defaults` - あなたのセッションの基本構成として使用する
  デフォルト設定。php, cake, cache, database という４種類の
  ビルトイン機構があります。

- `handler` - ユーザー独自のセッションハンドラーとして使えます。
  `session_save_handler` と一緒に使える callable の配列か、
  `CakeSessionHandlerInterface` を実装したクラス名を `engine` キーに持つ配列を指定します。
  このオプションを有効にすると、ini 配列に `session.save_handler` が自動的に追加されます。

- `autoRegenerate` - この設定を有効にすると、セッションを毎回
  自動生成し、セッション ID が頻繁に変わります。詳細は
  `CakeSession::$requestCountdown` を参照してください。

- `cacheLimiter` - セッションクッキーで使用するキャッシュ制御ヘッダー。
  指定できる値は [session_cache_limiter()](https://www.php.net/session_cache_limiter)
  を参照してください。

- `ini` - ini の値として追加で設定したい連想配列。

ビルトインのデフォルトは以下の通りです。

- 'php' - あなたの `php.ini` で定義されている設定を使う
- 'cake' - セッションファイルを CakePHP の `tmp/sessions` ディレクトリに保存する
- 'database' - CakePHP のデータベースセッションを使う
- 'cache' - セッションの保存に Cache クラスを使う

カスタムセッションハンドラーを定義する場合、
`src/Model/Datasource/Session/<name>.php` に記載します。
そのクラスは `CakeSessionHandlerInterface` を
実装している必要があります。
また `Session.handler.engine` を \<name\> に設定してください。

データベースセッションを使うには、cake のシェルコマンド
`bin/cake schema create Sessions` を使って、
`config/Schema/sessions.php` のスキーマからテーブルを作成します。

詳しくは [セッション](sessions) を参照してください。

Security.salt  
セキュリティ関連のハッシュ処理で使われるランダムな文字列。CSRF トークンやフォーム改ざん防止用
トークンの署名、`CookieComponent` のデフォルトの暗号化キーにも使われます。
アプリケーションごとに一意な値に変更し、秘密にしておいてください。
スケルトンのデフォルト値のままだと、CakePHP が警告を表示します。
スケルトンは、環境変数 `SECURITY_SALT` が設定されていればその値を使います。
生成方法と、変更したときの影響は [セキュリティ用のソルトの生成](#security-salts) を参照してください。

::: pieceofcake2
`SecurityComponent` が発行する CSRF トークンは
`Security.salt` を使った HMAC で署名されるようになりました (CVE-2020-15400)。
ソルトを変更すると、発行済みのトークンは無効になります。
[脆弱性への対応](/ja/security/fixes) を参照してください。
:::

Security.cipherSeed  
`Security::cipher()` で文字列を暗号化／復号化するのに使われるランダムな文字列（数字のみ）。
このメソッドを使うのは `CookieComponent` の暗号化タイプ `'cipher'` だけです。
このタイプは脆弱で、後方互換性のためだけに残されています。Cookie には `'aes'` タイプを使い、
`Security.useOpenSsl` が `true` になっていることを確認してください (app スケルトンでは設定済みです)。
スケルトンは、環境変数 `SECURITY_CIPHER_SEED` が設定されていればその値を使います。
意味があるのは 1 から 4294967295 までの数値だけです。本家の `config/core.php` にある 29 桁のデフォルト値を含め、これより長い値はすべて同じ
シード `0` になります。どの値にしても `'cipher'` タイプは安全になりません。
[cipherSeed と Cookie の 'cipher' タイプ](#cipher-seed) を参照してください。

Security.useOpenSsl  
true にすると、`Security::encrypt()` と `Security::decrypt()` (つまり Cookie の暗号化タイプ
`'aes'`) が mcrypt ではなく OpenSSL を使います。mcrypt 拡張は PHP 7.2 以降同梱されていないので、
`config/core.php` で有効にしてください。

```php
Configure::write('Security.useOpenSsl', true);
```

デフォルトは false です。[pieceofcake2/app](https://pieceofcake2.github.io/app/) のスケルトン (v2.3.1 以降) から
作成したアプリケーションにはこの行が含まれています。既存のアプリケーションでは追加してください。

Asset.timestamp  
適切なヘルパーを使っている場合、静的ファイルの URL
(CSS, JavaScript, 画像) の末尾に個々のファイルの最終更新時刻
を表すタイムスタンプを追加します。
有効な値は以下の通りです。
(boolean) false - 何もしない（デフォルト）
(boolean) true - debug \> 0 の場合にタイムスタンプを付加
(string) 'force' - debug \>= 0 の場合にタイムスタンプを付加

Acl.classname, Acl.database  
CakePHP のアクセス制御リスト(ACL)機能で使われる定数。
詳細はアクセス制御リストの章を参照してください。

</div>

> [!NOTE]
> キャッシュの設定も `core.php` にあります。これについては後述します。

`Configure` クラスは、コアの設定をその場で読み書き
するのに使われます。これは、たとえばあなたのアプリケーションの
限られた範囲でデバッグ設定を有効にしたい場合などに特に便利です。

### セキュリティ用のソルトの生成 {#security-salts}

`Security.salt` は、アプリケーションごとに一意で秘密の値にする必要があります。
暗号論的に安全な乱数生成器で生成してください。自分で考えた値や、サンプルにある値を使ってはいけません。
以下のコマンドは、Linux と macOS の POSIX シェル (`sh`、`bash`、`zsh`) で動作します。

`Security.salt` には、32 バイトの乱数を 64 文字の 16 進数で表した値を使います。

::: code-group

```bash [PHP]
php -r 'echo bin2hex(random_bytes(32)), PHP_EOL;'
```

```bash [OpenSSL]
openssl rand -hex 32
```

:::

ソルトは 32 文字以上にしてください。ソルトは `CookieComponent` のデフォルトの暗号化キーで、
暗号化タイプ `'aes'` (`Security::encrypt()`) は 32 バイト未満のキーに対して例外を投げます。

`Security.cipherSeed` を使うのは古い Cookie の `'cipher'` タイプだけで、値は 1 から 4294967295 までの
数値にする必要があります。本家の `config/core.php` にある 29 桁のデフォルト値を含め、これより長い値はすべて同じシードになり、どの値にしても
`'cipher'` は安全になりません。設定する前に [cipherSeed と Cookie の 'cipher' タイプ](#cipher-seed) を
読んでください。

Windows では、コマンドプロンプトや PowerShell で PHP のコマンドを実行するとき、シングルクォートの代わりに
ダブルクォートを使ってください。たとえば `php -r "echo bin2hex(random_bytes(32)), PHP_EOL;"` のようにします。

`pieceofcake2/app` のスケルトンから作成したアプリケーションでは、これらの値を
[環境変数](#environment-variables) `SECURITY_SALT` と `SECURITY_CIPHER_SEED` で設定します。
開発環境では、次のコマンドでソルトと範囲内のシードを `config/.env` に追記できます (先に `config/.env.example` を
コピーしておいてください。同じ変数が 2 回設定されている場合は、後の行が有効になります)。

::: code-group

```bash [PHP]
php -r 'printf("SECURITY_SALT=\"%s\"\nSECURITY_CIPHER_SEED=\"%d\"\n", bin2hex(random_bytes(32)), random_int(1, 4294967295));' >> config/.env
```

```bash [Shell]
printf 'SECURITY_SALT="%s"\nSECURITY_CIPHER_SEED="%s"\n' "$(openssl rand -hex 32)" "$(od -An -N4 -tu4 /dev/urandom | tr -dc 0-9)" >> config/.env
```

:::

値は秘密にし、環境ごとに分けてください。

- 環境 (開発、ステージング、本番) ごと、アプリケーションごとに異なる値を使います。
- 値をコミットしないでください。スケルトンでは `config/.env` と `.env` は Git で無視されます。
  本番環境では `.env` ファイルではなく、実際の環境変数を設定してください。値を `config/core.php`
  に書く場合は、そのファイルを公開リポジトリに置かないでください。
- ソルトが漏れた場合 (コミットした、共有した、サンプル値のままだった場合など) は、値を変更してください。

#### ソルトに依存するもの

`Security.salt` を変更すると、次のものに影響します。

- **ソルトを使ってハッシュ化したパスワード。** `FormAuthenticate` と `BasicAuthenticate` のデフォルトの
  パスワードハッシャーである `SimplePasswordHasher` と、非推奨の `AuthComponent::password()` は、
  パスワードの前にソルトを付けてハッシュ化します (`$salt = true` での `Security::hash()`)。
  保存済みのハッシュが一致しなくなり、ユーザーはログインできなくなります。bcrypt のハッシュ
  (`BlowfishPasswordHasher`、`Security::hash($password, 'blowfish')`) はソルトを使わないので影響を受けません。
- **暗号化した Cookie。** `CookieComponent` は、`$this->Cookie->key` を設定しない限りソルトをキーに使います。
  また、`'aes'` タイプは専用のキーを設定してもソルトをキーに組み合わせます。そのため、専用のキーを設定した
  `'cipher'` と `'rijndael'` の Cookie を除き、変更前に書き込んだ Cookie は読めなくなります。
- **`Security::encrypt()` で暗号化したデータ。** `$hmacSalt` を指定しない場合はソルトが暗号化キーの一部になるため、
  変更前に暗号化した値に対して `Security::decrypt()` は `false` を返します。
- **セッション。** `Session.checkAgent` が `false` でない場合 (デフォルト)、セッションにはユーザーエージェントと
  ソルトのハッシュが保存されているため、既存のセッションは破棄され、ユーザーはログアウトされます。
- **CSRF トークンとフォーム改ざん防止用トークン。** `SecurityComponent` と `FormHelper` が発行するトークンは
  ソルトで署名されています。変更前に表示したフォームは一度だけ拒否されます。

`Security.cipherSeed` を使うのは `Security::cipher()` だけです。変更すると `Security::cipher()` で暗号化した値が
読めなくなります。実際には、`CookieComponent` のデフォルトの暗号化タイプ `'cipher'` で書き込んだ Cookie が該当します。
[cipherSeed と Cookie の 'cipher' タイプ](#cipher-seed) を参照してください。

#### ソルトの変更手順

セッション、CSRF トークン、フォームのトークンは自然に回復します。ユーザーがログインし直し、フォームを
送信し直せばよいので、利用の少ない時間帯にソルトを変更してください。変更する前に、長く保存される値を
ソルトに依存しないようにしておきます。

1. ソルトを使ったパスワードのハッシュを bcrypt (`'passwordHasher' => 'Blowfish'`) に移行します。
   ハッシュは変換できないので、ユーザーのログイン時に古いハッシュで確認し、パスワードの bcrypt ハッシュを
   保存します。ソルトを使ったハッシュがなくなってからソルトを変更するか、残ったユーザーにパスワードの
   再設定を求めてください。
2. デフォルトの `$hmacSalt` で `Security::encrypt()` を使って暗号化した保存済みの値は、古いソルトを
   `Security::decrypt()` に渡して暗号化し直します。`Security.salt` と一緒に変わらない専用の `$hmacSalt`
   を渡すようにすると、より安全です。

   ```php
   $plain = Security::decrypt($encrypted, $key, $oldSalt);
   $encrypted = Security::encrypt($plain, $key, $hmacSalt);
   ```

3. 暗号化した Cookie は引き継げません。`'aes'` タイプはキーに `Security.salt` を組み合わせるため、
   ソルトを変更すると Cookie は読めなくなり、ログイン状態を保持する Cookie を持っていたユーザーは
   一度ログインし直す必要があります。まだ `'cipher'` タイプを使っている場合は、同時に専用のキーを設定した
   `'aes'` に移行すれば、Cookie が失われるのは一度だけで済みます。その際は `Security.useOpenSsl` が
   `true` になっていることを確認してください (app スケルトンでは設定済みです。
   [`'aes'` への移行](#migrating-to-aes) を参照してください)。

#### その他の秘密の値

- `Security::encrypt()` と `Security::decrypt()` に渡すキーや、`CookieComponent` のキーは 32 バイト以上に
  してください。`Security.salt` と同じ方法で生成し、`COOKIE_KEY` から `Cookie.key` に読み込むように、環境変数から設定に読み込みます
  ([`'aes'` への移行](#migrating-to-aes) を参照してください)。
- データベースやメールのパスワード、API キーは、コミットするコードではなく、環境変数や、
  `config/database.php` と `config/email.php` のような Git で無視されるファイルに置いてください。

### cipherSeed と Cookie の 'cipher' タイプ {#cipher-seed}

`Security.cipherSeed` を使うのは `Security::cipher()` だけです。実際には、デフォルトの暗号化タイプが
`'cipher'` である `CookieComponent` がこれに当たります。`'cipher'` タイプは以前から安全でないと
説明されており、後方互換性のためだけに残されています。この節では、シードの既知の弱点と、
それがアプリケーションに与える影響、`'aes'` タイプへの移行方法を説明します。

#### シードの使われ方

`Security::cipher()` は `mt_srand((int)(float)$seed)` で PHP のメルセンヌ・ツイスタを初期化し、
`mt_rand()` で得た数値と値を XOR します。キー (`CookieComponent` のキーを設定していなければ
`Security.salt`) も組み合わされます。

`mt_srand()` のシードは 32 ビットなので、意味があるのは 1 から 4294967295 までの数値だけです。
これより長い値にしても何も加わりません。

- 19 桁を超える値は PHP の整数に収まらず、変換でビットが失われます。
- 27 桁以上の値と、26 桁の値の約 95% は、すべてシード `0` になります。

シード `0` になるよく知られた値には、次のものがあります。

- `76859309657453542496749683645`: 本家 CakePHP 2 と、v2.12.5 までのこのフォークの `config/core.php` のデフォルト値。
  `pieceofcake2/app` スケルトンの以前のバージョンから作成したアプリケーションでも、環境変数がない場合の
  値として使われています
- `7485712659625147843639846751`: 本家のドキュメントのブログチュートリアルにあるサンプル値
- `pieceofcake2/app` スケルトンの以前のバージョンの `config/.env.example` に書かれていたコマンドで
  生成した 30 桁の値

#### 範囲内のシードでも十分ではありません

1 から 4294967295 までのシードにすれば `0` になることは避けられますが、`'cipher'` が安全になるわけでは
ありません。

- シードは約 43 億通りしかないため、すべて試すことができます。
- キーストリームは暗号用ではないメルセンヌ・ツイスタで生成され、値はそれと XOR されるだけです。
- `Security.salt` (または `CookieComponent` のキー) も組み合わされますが、それで本当の暗号化になる
  わけではありません。

シードがどの値であっても、値を秘密にするために `'cipher'` に頼らないでください。

#### 影響

- 影響を受けるのは `Security::cipher()` で暗号化した値だけです。つまり、`CookieComponent` が
  `'cipher'` タイプ (デフォルト) で書き込む Cookie と、アプリケーションが直接呼び出す
  `Security::cipher()` です。Cookie の `'aes'` と `'rijndael'` タイプ、`Security::encrypt()`、
  パスワードのハッシュ、セッション、CSRF トークンはシードを使いません。
- 上記のいずれかの値や、その他の 27 桁以上の値を使っているアプリケーションでは、シードは秘密として
  まったく役に立たず、そうしたアプリケーションはすべて同じシードを使っています。
- シードをほかの値に変更すると、`'aes'` に切り替えた場合と同じように、既存の `'cipher'` の Cookie は
  読めなくなります。

#### どうすればよいですか

- **新しいアプリケーション:** [`'aes'` への移行](#migrating-to-aes) のとおり、専用のキーを設定した
  `'aes'` タイプを使い、`'cipher'` は使わないでください。`Security.useOpenSsl` は `true` にします
  (app スケルトンでは設定済みです)。
- **`'cipher'` の Cookie を使っている既存のアプリケーション:** `'aes'` に移行してください。
  既存の Cookie の値は一度だけ失われます。たとえば、ログイン状態を保持する Cookie でログインしていた
  ユーザーはログインし直す必要があり、Cookie に保存した設定は元に戻ります。手順は
  [`'aes'` への移行](#migrating-to-aes) を参照してください。
- **移行期間中も古い Cookie を読み続ける必要があるアプリケーション:** 影響なく移行する方法は
  ありません。古い Cookie を読めるのは古いシードを使い続けている間だけで、その間は弱点も残ります。
  範囲内のシードに変更するとキーストリームも変わるため、結局古い Cookie は読めなくなり、
  変更後も `'cipher'` は安全ではありません。Cookie が一度失われる日を決めておき、シードだけを
  変更するのではなく、その日に `'aes'` に切り替えてください。
- **`CookieComponent` も `Security::cipher()` も使っていないアプリケーション、**
  または `'aes'` タイプだけを使っているアプリケーション: `Security.cipherSeed` を 1 から 4294967295
  までの有効な値にすること以外に、することはありません。

#### `'aes'` への移行 {#migrating-to-aes}

1. 32 バイトの乱数で Cookie のキーを生成します。`Security.salt` を使い回さないでください。

   ::: code-group

   ```bash [PHP]
   php -r 'echo bin2hex(random_bytes(32)), PHP_EOL;'
   ```

   ```bash [OpenSSL]
   openssl rand -hex 32
   ```

   :::

   生成した値を環境変数 `COOKIE_KEY` に設定します。開発環境では `config/.env` に書き、
   本番環境では実際の環境変数として設定してください。

2. `Security.useOpenSsl` が `true` になっていることを確認します (app スケルトンでは設定済みです)。
   `'aes'` タイプが使う `Security::encrypt()` と `Security::decrypt()` は、この設定が `true` のときだけ
   OpenSSL を使います。そうでない場合は PHP 8 には存在しない `mcrypt` を呼び出すため、`'aes'` の
   Cookie は失敗します。古いアプリケーションにはこの行がないことがあるので、`config/core.php` に
   追加してください ([OpenSSL を有効にする](/ja/appendices/migrating-from-cakephp-2-10#enable-openssl) を参照してください)。

   ```php
   Configure::write('Security.useOpenSsl', true);
   ```

3. `CookieComponent` のタイプとキーを設定します。方法は `pieceofcake2/cakephp` のバージョンによって
   異なります。下記を参照してください。

::: pieceofcake2
v2.12.6 以降では、`Cookie.type` と `Cookie.key` を設定で指定できます。

**v2.12.6 以降:** `config/core.php` のソルトの近くで設定します。コントローラーを変更する必要は
ありません。`pieceofcake2/app` スケルトンの新しいバージョンにはこれらの行があります。
`config/core.php` にない場合は追加してください。

```php
// config/core.php
/**
 * Default encryption type and key of CookieComponent. The key is a random
 * value of at least 32 bytes.
 *
 * The key can be set with the COOKIE_KEY environment variable.
 */
Configure::write('Cookie.type', 'aes');
Configure::write('Cookie.key', env('COOKIE_KEY') ?: null);
```

`CookieComponent` のタイプは、`'type'` の設定または `$this->Cookie->type()` が最優先で、次に
`Cookie.type`、どちらもなければ `'cipher'` です。キーは、`'key'` の設定または `$this->Cookie->key` が
最優先で、次に `Cookie.key`、どちらもなければ `Security.salt` です。

**v2.12.5 以前:** `CookieComponent` は `Cookie.type` と `Cookie.key` を読まないため、上の設定は
効果がありません。`config/core.php` の `Cookie.key` の行は残し、タイプとキーはコントローラーで渡します。
タイプはコンポーネントの設定で指定し、キーは Cookie を読む前に `beforeFilter()` で設定します。
`AppController`、またはこのコンポーネントを使うすべてのコントローラーで設定してください。

```php
// src/Controller/AppController.php
public $components = [
    'Cookie' => ['type' => 'aes'],
];

public function beforeFilter()
{
    parent::beforeFilter();
    $key = Configure::read('Cookie.key');
    if ($key) {
        $this->Cookie->key = $key;
    }
}
```

`null` を代入するとデフォルトのキーが置き換えられて暗号化に失敗するため、キーは値が設定されている
場合だけ代入してください。`'type'` の設定は v2.12.3 から反映されます。それより古いバージョンでは
`beforeFilter()` で `$this->Cookie->type('aes');` を呼び出してください。明示的な設定が優先される
ため、このコードは v2.12.6 以降もそのまま動作します。
:::

`'aes'` タイプには 32 バイト以上のキーが必要です。`COOKIE_KEY` が設定されていない場合、`Cookie.key`
は `null` になり、キーには `Security.salt` が使われます。ソルトが 32 バイト未満の場合、暗号化した
Cookie を書き込んだり読んだりするたびに、`Security::encrypt()` と `Security::decrypt()` が
`CakeException` を投げます。

4. 必要に応じて `$this->Cookie->name` で Cookie の名前を変更します。そうすると、古い `'cipher'` の
   Cookie は新しいタイプで復号されずに無視され、そのまま期限切れになります。

5. デプロイします。ログイン状態を保持する Cookie を持っていたユーザーは、一度ログインし直します。

`'aes'` タイプが使う `Security::encrypt()` は、キーに `Security.salt` も組み合わせます。
後で `Security.salt` を変更すると、`'aes'` の Cookie も読めなくなります。

#### cipherSeed の生成

`'cipher'` をまだ使っている場合に限り、`Security.cipherSeed` を 1 から 4294967295 までの数値にします。
これで `0` になることは避けられますが、`'cipher'` は安全でないままです。また、シードを変更すると
既存の `'cipher'` の Cookie は読めなくなります。

::: code-group

```bash [PHP]
php -r 'echo random_int(1, 4294967295), PHP_EOL;'
```

```bash [Shell]
od -An -N4 -tu4 /dev/urandom | tr -dc 0-9; echo
```

:::

#### v2.12.6 以降の警告 {#cipher-warnings}

::: pieceofcake2
v2.12.6 以降の `pieceofcake2/cakephp` は、動作を変えずに次の問題を報告します。

- `'cipher'` は非推奨です。プロセス (1 回のリクエストや 1 回のコンソール実行) の中で最初に
  `'cipher'` タイプまたは `Security::cipher()` を使うと、デバッグモードがオフでも `warning` を
  ログに記録します。デバッグモードがオンのときは、同じメッセージを `E_USER_DEPRECATED` としても
  発生させます。メッセージには呼び出し元と対処方法 (`Security.useOpenSsl` を有効にして、
  `Cookie.type` を `'aes'` に、`Cookie.key` を 32 バイト以上のランダムな値にすること) が含まれます。
- `Security.cipherSeed` が 1 から 4294967295 までの数字だけの値でない場合は、プロセスの中で最初に
  `Security::cipher()` を呼び出したときに `warning` もログに記録します。シードの値そのものは
  記録しません。デバッグモードがオンのときは、`Debugger::checkSecurityKeys()` (`AuthComponent`
  から呼ばれます) がこのようなシードに対して `E_USER_NOTICE` も発生させます。
- アプリケーションに `config/core.php` がない場合にだけ使われるコアの代替の `config/core.php` は、
  範囲内のシード `'2659758734'` (app スケルトンと同じデフォルト値) を設定し、`Security.useOpenSsl` を
  有効にします。デフォルトの `Security.cipherSeed` の変更を求める通知は、以前のデフォルト値と
  この新しいデフォルト値の両方で表示されます。
- `Cookie.type` と `Cookie.key` で `CookieComponent` のデフォルトのタイプとキーを設定できます
  ([`'aes'` への移行](#migrating-to-aes) を参照してください)。

値の暗号化と復号は以前とまったく同じなので、既存の Cookie はそのまま読め、例外も投げられません。
警告を消すためだけに `Security.cipherSeed` を変更しないでください。シードを変えると既存の `'cipher'`
の Cookie が読めなくなり、どのシードでも `'cipher'` は安全ではありません。代わりに `'aes'` に
移行してください。

テストで非推奨の警告や通知を例外にしている場合、テスト実行中の最初の `'cipher'` の使用や、
デバッグモードがオンで範囲外のシードのときの `AuthComponent` は失敗するようになります。
アプリケーションを `'aes'` に移行したら、`IntegrationTestTrait` の `cookieEncrypted()` /
`assertCookieEncrypted()` のモードに `'aes'` を渡してください。

v2.12.5 以前では何も報告されませんが、弱点は同じです。
:::

### 構成用定数

ほとんどの構成用設定は Configure によって処理されますが、実行時に
CakePHP が参照する定数がいくつか存在します。

`constant` **LOG_ERROR**

エラー定数。エラーログ出力とデバッグを区別するのに使われます。
現在 PHP は LOG_DEBUG をサポートします。

### コアのキャッシュ設定

CakePHP は内部的に `_cake_model_` と `_cake_core_`
という２つのキャッシュ設定を使います。`_cake_core_` はファイルの
パスやオブジェクトの格納位置を保存するのに使われ、`_cake_model_`
はスキーマの定義やデータソースのソース一覧を保存するのに使われます。
これらはリクエストのたびに参照されるので、APCu や Memcached のような
高速なキャッシュストレージを使うことをおすすめします。デフォルト
では debug \> 0 の場合、いずれの設定も 10 秒おきに有効期限が切れます。
どちらも `config/core.php` の末尾で定義されています。

使用できるキャッシュエンジンは File、Apc (APCu 拡張が読み込まれていれば APCu を使います)、
Memcached、Memcache、Redis です。

::: pieceofcake2
Xcache と Wincache のキャッシュエンジンは削除されました。
これらを使っている設定は、上記のいずれかのエンジンに切り替えてください。
:::

`Cache` に保存されたすべてのキャッシュデータと同じく、
`Cache::clear()` を使ってクリアできます。

## Configure クラス {#configure-class}

`class` **Configure**

CakePHP では設定が必要な項目はほとんどありませんが、あなたの
アプリケーション固有の設定ルールを作ることが有用な場合もあります。
あなたは過去に、いくつかのファイルの中で変数や定数を定義することで、
独自の設定値を持たせていたようなケースがあったかもしれません。
この方法では、これらの値を使うために毎回設定ファイルを
インクルードする必要がありました。

CakePHP の Configure クラスは、アプリケーションや実行時固有の値を
保存したり取り出したりするのに使います。注意していただきたいのですが、
このクラスは、その中にいかなるものでも格納することができ、さらにあなたの
コードのいかなる箇所でもそれらを使うことができてしまいます。
これにより、CakePHP のデザインの元になっている MVC パターンを
壊してしまうことになりかねません。 Configure クラスのそもそもの目的は、
多数のオブジェクト間で共有できる変数群の集中管理です。
"convention over configuration（設定より規約）"の世界の中で生きようと
する努力を怠らず、せっかく私たちが整えた MVC 構造という環境を壊すことの
ないようにしてください。

このクラスは、あなたのアプリケーション中の静的なコンテキストにおいて、
いかなるところからでも呼び出せるようになっています。

```php
Configure::read('debug');
```

`static` Configure::**write**($key, $value)

- `string $key`: 書き込むキー。 `ドット記法` が使えます。
- `mixed $value`: 保存する値

アプリケーションの設定の中で `write()` を使ってデータを格納します。

```php
Configure::write('Company.name', 'Pizza, Inc.');
Configure::write('Company.slogan', 'Pizza for your body and soul');
```

> [!NOTE]
> `$key` パラメーターで使われる `ドット記法` により、
> あなたの構成設定値を論理的にグループ化できます。

直前の例は、以下のような単一の呼び出しとしても書けます。

```php
Configure::write(
    'Company',
    [
        'name' => 'Pizza, Inc.',
        'slogan' => 'Pizza for your body and soul',
    ],
);
```

`Configure::write('debug', $int)` を使えば、その場で動的に
デバッグモードと本番モードを行ったり来たりできます。特に、
JSON や XML のレスポンスではデバッグ情報がパースエラーの原因と
なりがちなので、それを防ぐのに有効です。

`static` Configure::**read**($key = null)

- `string $key`: 読み込むキー。 `ドット記法` が使えます。

アプリケーションから設定データを読み込むのに使います。デフォルトは
CakePHP で重要な debug 値です。何らかのキーが指定されたらその
データを返します。前述の write() の例を元に、そのデータを呼び出して
みましょう。

```php
Configure::read('Company.name'); // 'Pizza, Inc.' が返されます
Configure::read('Company.slogan'); // 'Pizza for your body and soul' が返されます

Configure::read('Company');

// 返される値:
['name' => 'Pizza, Inc.', 'slogan' => 'Pizza for your body and soul'];
```

\$key を指定しない場合、 Configure にあるすべての値を返します。指定された \$key に関連する
値が存在しない場合、 null を返します。

`static` Configure::**consume**($key)

- `string $key`: 読み込むキー。 `ドット記法` が使えます。

Configure からキーを読み込み、同時に削除します。値の読み込みと削除を
1 回の操作で行いたい場合に便利です。

`static` Configure::**check**($key)

- `string $key`: チェックするキー。

key/path が存在し、かつ null でない値を持つかどうかをチェックします。

::: info Added in version 2.3
`Configure::check()` は 2.3 で追加されました。
:::

`static` Configure::**delete**($key)

- `string $key`: 削除するキー。 `ドット記法` が使えます。

アプリケーションの設定から情報を削除します。

```php
Configure::delete('Company.name');
```

`static` Configure::**version**()

現在のアプリケーションで動いている CakePHP のバージョンを返します。

`static` Configure::**config**($name, $reader)

- `string $name`: 接続中のリーダー（読み込み機構）の名前
- `ConfigReaderInterface $reader`: 接続されるリーダーのインスタンス

Configure に設定リーダーを接続します。接続されたリーダーは、
設定ファイルをロードするのに使われます。設定ファイルを読み込む
方法の詳細は [設定ファイルのロード](#loading-configuration-files) をご覧ください。

`static` Configure::**configured**($name = null)

- `string $name`: チェックするリーダーの名前。指定しない場合、
  接続されているすべてのリーダーの一覧が返されます。

指定された名前のリーダーが接続されているかをチェックするか、
または接続されたリーダーの一覧を取得します。

`static` Configure::**drop**($name)

接続されているリーダーオブジェクトを削除します。

## 設定ファイルを読み書きする

CakePHP には２つのビルトインの設定ファイル用リーダーが備わっています。
`PhpReader` は PHP の設定ファイルを読むことができますが、
Configure も歴史的にこれと同じフォーマットのファイルを読めるように
なっています。 `IniReader` は ini 形式の設定ファイルを
読むことができます。ini ファイルの仕様については
[PHP documentation](https://www.php.net/parse_ini_file) を参照してください。
コアの設定ファイルリーダーを使うには、そのリーダーを
`Configure::config()` で Configure に接続してください。

```php
// CONFIG (config/) から設定ファイルを読み込む
Configure::config('default', new PhpReader());

// 別のパスから設定ファイルを読み込む
Configure::config('default', new PhpReader('/path/to/your/config/files/'));
```

どちらのリーダーも、デフォルトでは `CONFIG` ディレクトリから読み込みます。
`Configure::load('MyPlugin.my_file')` のようにプラグイン記法を使うと、
プラグインの `Config/` ディレクトリから読み込みます。

Configure に複数のリーダーを接続することができます。それぞれの
リーダーは異なった種類の設定ファイル、もしくは異なったタイプの
ソースを読みます。Configure に備わっている他のメソッドを使って
接続されたリーダーとやりとりすることもできます。
`Configure::configured()` を使って接続されている
リーダーの別名を取得できます。

```php
// 接続されたリーダーの別名の配列を取得する
Configure::configured();

// 特定のリーダーが接続されているかをチェック
Configure::configured('default');
```

接続されているリーダーを取り外すこともできます。
`Configure::drop('default')` はデフォルトリーダーの別名を
取り外します。これ以降、そのリーダーを介した設定ファイルの読み込みは
失敗します。

<a id="loading-configuration-files"></a>

### 設定ファイルのロード

`static` Configure::**load**($key, $config = 'default', $merge = true)

- `string $key`: ロードする設定ファイルの識別子
- `string $config`: 設定されたリーダーの別名
- `boolean $merge`: 読まれたファイルの中身をマージするか、
  それとも既存の値を上書きするか。

Configure に設定リーダーを接続して、設定ファイルを読み込みます。

```php
// リーダーオブジェクト 'default' を使って config/my_file.php をロードする
Configure::load('my_file', 'default');
```

ロードされた設定ファイルは、そのデータを Configure にある既存の
実行時設定とマージします。また、既存の実行時設定を上書きして
新しい値を追加することもできます。 `$merge` を false にすると、
値は既存の設定には上書きされません。

### 設定ファイルを生成／変更する

`static` Configure::**dump**($key, $config = 'default', $keys = [])

- `string $key`: ファイル名、または生成されるストレージ設定
- `string $config`: データを格納するリーダーの名前
- `array $keys`: 保存したいトップレベルのキーのリスト。デフォルトは
  すべてのキー。

Configure にある全部または一部のデータをダンプして、ファイルまたは
設定リーダーでサポートされているストレージシステムに格納します。
シリアライズのフォーマットは \$config として接続されている設定
リーダーにより決定されます。たとえば 'default' のアダプターが
`PhpReader` なら、生成されたファイルは
`PhpReader` で読み込みが可能な PHP の設定ファイルになります。

'default' リーダーが PhpReader のインスタンスだとして、Configure
にあるすべてのデータを `my_config.php` というファイルに保存します。

```php
Configure::dump('my_config.php', 'default');
```

エラーハンドラー設定だけを保存します。

```php
Configure::dump('error.php', 'default', ['Error', 'Exception']);
```

`Configure::dump()` は `Configure::load()` で読める形式の
設定ファイルを変更または上書きするのにも使えます。

::: info Added in version 2.2
`Configure::dump()` は 2.2 で追加されました。
:::

### 実行時設定を保存する

`static` Configure::**store**($name, $cacheConfig = 'default', $data = null)

- `string $name`: キャッシュファイルのストレージキー
- `string $cacheConfig`: 設定データを保存するキャッシュ設定の名前
- `mixed $data`: 保存するデータ、null にすると Configure 中のすべての
  データを保存。

将来取り出して使えるように、実行時設定の値を保存することもできます。
configure は現在のリクエストの値しか記憶しないので、何らかの変更した
値を後続するリクエストでも使いたい場合は、それらを保存しておく
必要があります。

```php
// 'default' キャッシュにある 'user_1234' キーの中の現在の設定を保存する
Configure::store('user_1234', 'default');
```

保存された設定データは `Cache` クラス内で永続的データになります。
これにより、 `Cache` が扱えるストレージエンジンにおいて、
設定情報を保存できます。

### 実行時設定を呼び出す

`static` Configure::**restore**($name, $cacheConfig = 'default')

- `string $name`: ロードするストレージキー
- `string $cacheConfig`: データをロードするキャッシュ設定

実行時設定を保存した後は、おそらくそこに再度アクセスして値を取り出す
ことになります。これは `Configure::restore()` により行います。

```php
// キャッシュから実行時設定を取り出す
Configure::restore('user_1234', 'default');
```

設定情報を取り出す際は、保存した時に使ったものと同じキャッシュから、
同じキーを使って取り出すことが重要です。取り出された情報は、
既存の実行時設定の上にマージされます。

## 独自の設定用リーダーを作成する

設定用リーダーは CakePHP の中で拡張可能な部品です。このため、
あなたのアプリケーションとプラグインの中で、設定用リーダーを
新たに作成することも可能です。設定用リーダーは
`ConfigReaderInterface` を実装する必要があります。
このインターフェースでは `read()` と `dump()` メソッドが定義されています。
たとえば、あなたが本当に XML ファイルが好きなら、あなたのアプリケーションのために
シンプルな XML 設定リーダーを作ることもできます。

```php
// src/Lib/Configure/MyXmlReader.php
class MyXmlReader implements ConfigReaderInterface
{
    protected $_path;

    public function __construct($path = null)
    {
        if (!$path) {
            $path = CONFIG;
        }
        $this->_path = $path;
    }

    public function read($key)
    {
        $xml = Xml::build($this->_path . $key . '.xml');

        return Xml::toArray($xml);
    }

    /**
     * 2.3 以降は dump() メソッドも必須です
     */
    public function dump($key, $data)
    {
        // データをファイルに書き出すコード
    }
}
```

`config/bootstrap.php` でこのリーダーを接続して利用できます。

```php
Configure::config('xml', new MyXmlReader());
// ...

Configure::load('my_xml');
```

`src/` を Composer の classmap でオートロードしていれば、`composer dump-autoload`
の後はリーダークラスが自動的に読み込まれます。classmap を使っていない場合は、
先に `App::uses('MyXmlReader', 'Configure');` でクラスを読み込んでください。

> [!WARNING]
> 独自の設定クラスに `XmlReader` という名前を付けるのはおすすめしません。
> PHP には同じ名前の組み込みクラス
> [XMLReader](https://www.php.net/manual/ja/book.xmlreader.php) があるからです。

設定リーダーの `read()` メソッドは、 `$key` という名前のリソースが
保持する設定情報の配列を返さなければなりません。

`interface` **ConfigReaderInterface**

`Configure` で設定データを読んだり保存したりするクラスで
使われるインターフェースを定義します。

`method` ConfigReaderInterface::**read**($key)

`method` ConfigReaderInterface::**dump**($key, $data)

::: info Added in version 2.3
`ConfigReaderInterface::dump()` は 2.3 で追加されました。
:::

`exception` **ConfigureException**

設定データをロード／保存／復元する際にエラーが発生すると、この例外が投げられます。
`ConfigReaderInterface` の実装は、エラーが発生した場合にこの例外を投げる必要があります。

### ビルトインの設定リーダー

`class` **PhpReader**

`class` **IniReader**

<a id="inflection-configuration"></a>

## Inflection(語尾変化)設定

CakePHP の命名規約は実に優れています。あなたのデータベーステーブルに
big_boxes 、モデルに BigBox 、コントローラーに BigBoxesController
と名前をつけてもらえれば、これですべてが自動的に動作します。
CakePHP はそれぞれの部品を関連付けるにあたり、それぞれの単語の
単数形と複数形を *inflecting(語尾の変化)* によって把握します。

ここで、CakePHP の `Inflector` （複数形、単数形、
キャメルケース、アンダースコアへの相互変換を司るクラス）が
思ったように動かないケースを想定してみましょう。これは特に、
英語話者ではない人たちにとって起こりがちです。もし CakePHP
があなたの Foci （訳注：focus（焦点）の複数形）や Fish （訳注：
同じ種類の魚の場合は単複同型）を認識してくれない場合、CakePHP
にあなたの特別なケースについて教えることができます。

### カスタム inflection のロード

カスタム inflection をロードするには `config/bootstrap.php`
ファイルの中に `Inflector::rules()` を書きます。

```php
Inflector::rules('singular', [
    'rules' => [
        '/^(bil)er$/i' => '\1',
        '/^(inflec|contribu)tors$/i' => '\1ta',
    ],
    'uninflected' => ['singulars'],
    'irregular' => ['spins' => 'spinor'],
]);
```

または

```php
Inflector::rules('plural', ['irregular' => ['phylum' => 'phyla']]);
```

これらのコードは、指定されたルールをフレームワークの `src/Cake/Utility/Inflector.php`
(`vendor/pieceofcake2/cakephp/src/Cake/Utility/Inflector.php`) で定義されている
inflection セットにマージし、コアのルールより優先度の高いルールとして追加します。

## CakePHP の起動(bootstrap)

もし何らかの追加設定が必要な場合、`config/bootstrap.php`
にある CakePHP の bootstrap ファイルを使います。このファイルは
CakePHP のコアな起動処理の直後に実行されます。

このファイルは起動時における多くの共通的な処理に適しています。

- 便利な関数群を定義
- グローバル定数群の登録
- 追加のモデル、ビュー、コントローラーへのパスを定義
- キャッシュ設定の作成
- inflection の構成
- 設定ファイルの読み込み

何かを bootstrap ファイルに追加する場合は、MVC ソフトウェアの
デザインパターンを守るよう注意してください。コントローラーで使うための
独自のフォーマット関数をここに入れたくなるかもしれません。

ぜひその誘惑と戦ってください。その先に、必ず良いことがありますよ。

また、何かを `AppController` クラスに入れることを考慮する
必要があるかもしれません。このクラスはあなたのアプリケーションの
すべてのコントローラーの親クラスです。 `AppController`
はあなたのコントローラーすべてにおいて使われるコールバックや
メソッドを定義するための便利な場所です。
