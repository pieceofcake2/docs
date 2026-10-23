# モダン構成への移行

Piece of Cake 2 フォークは、2 種類のアプリケーション構成に対応しています。

- **レガシー構成**: `app/` ディレクトリを使う従来の CakePHP 2.x の構成です。
- **モダン構成**: プロジェクト直下に `config/`、`src/`、`templates/`、`tests/` などを置く
  CakePHP 5.x 風の構成です。[アプリケーションスケルトン](/ja/installation/application-skeleton)
  はこの構成を使っています。

どちらの構成も同じバージョンのフレームワークで動作します。レガシー構成のまま使い続けることも、
段階的にモダン構成へ移ることもできます。モダン構成では、テンプレートを CakePHP 5 のディレクトリ
(`layout/`、`element/`、`email/`、`Error/`) にある `.php` ファイルにし、アプリケーションのクラスを CakePHP 5 と
同じ `App` 名前空間に移すこともできます。将来 CakePHP 5 にアップグレードする予定があるなら、CakePHP 2 のまま
これらの変更をしておくことで、今のうちに準備できます。

これらの変更は [pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/ja/) の `layout` コマンドと
`namespace` コマンドで行えます ([pieceofcake2/upgrade で移行する](#migrating-with-pieceofcake2-upgrade) を参照)。
クラスをグローバルのままにするなら、`layout` だけを実行します。2 つのスケルトンが移行後の形を示しています。

- [pieceofcake2/app](/ja/installation/application-skeleton): モダン構成、`.php` のテンプレート、`App` 名前空間の
  クラス。
- [pieceofcake2/app-legacy](/ja/installation/application-skeleton-legacy): モダン構成で、グローバルなクラスと
  `.ctp` のテンプレート。

## CakePHP 5 への段階的な移行 {#gradual-migration-to-cakephp-5}

CakePHP 2.x から 5.x へ一度に移行すると、コード、フォルダー構成、API をすべて同時に変えることに
なります。モダン構成を使うと、これを小さなステップに分けられます。

```text
Step 1: 従来の構成の CakePHP 2.x
        ↓（フォルダー構成だけをモダン化: upgrade layout）
Step 2: CakePHP 5.x 風の構成の CakePHP 2.x  ← ここで止めてもよい
        ↓（クラスを名前空間に移す: upgrade namespace）
Step 3: 名前空間付きのクラスの CakePHP 2.x  ← ここで止めてもよい
        ↓（コードだけをアップグレード）
Step 4: CakePHP 5.x 風の構成の CakePHP 5.x
```

- フォルダーの再編とコードの変更を分けられるので、1 回の変更が小さく扱いやすくなります。
- 次に進む前に、各ステップをテストで確認できます。
- 必要なら Step 2 や Step 3 に留まり続けられます。
- 変更が小さいので、チームでのレビューもしやすくなります。

## レガシー構成とモダン構成 {#legacy-and-modern-layouts}

::: code-group

```text [レガシー (app/)]
your-project/
├── app/
│   ├── Config/          (大文字、app/ の下)
│   ├── Controller/
│   ├── Model/
│   ├── View/            (テンプレートとヘルパーが混在)
│   ├── Test/
│   ├── Plugin/
│   ├── Vendor/
│   └── tmp/
│       └── logs/
├── vendors/
└── ...
```

```text [モダン (src/)]
your-project/
├── bin/
│   └── cake
├── config/              (小文字、プロジェクト直下)
├── src/                 (PHP コードすべて、App 名前空間)
│   ├── Controller/
│   ├── Model/
│   └── View/            (ヘルパークラスのみ)
├── templates/           (テンプレートすべて、.php ファイル)
│   ├── Error/
│   ├── element/
│   ├── email/
│   └── layout/
├── resources/
│   └── locales/
├── tests/               (小文字、プロジェクト直下)
├── plugins/             (Composer で管理)
├── vendor/              (Composer の標準)
├── logs/                (tmp/ から分離)
├── tmp/
└── webroot/
```

:::

グローバルなクラスと `.ctp` のテンプレートのまま (Step 2。[pieceofcake2/app-legacy](/ja/installation/application-skeleton-legacy)
と同じ形) では、テンプレートは CakePHP 2 のディレクトリ (`Layouts/`、`Elements/`、`Emails/`、`Errors/`) のままです。

## ファイルの移行対応表 {#file-migration-map}

| 移行元 (レガシー) | 移行先 (モダン) |
| --- | --- |
| `app/Config/*` | `config/*` |
| `app/Controller/*` | `src/Controller/*` |
| `app/Model/*` | `src/Model/*` |
| `app/View/**/*.ctp` | `templates/**/*.php` (または `templates/**/*.ctp`) |
| `app/View/Layouts/*` | `templates/layout/*` |
| `app/View/Layouts/Emails/*` | `templates/layout/email/*` |
| `app/View/Elements/*` | `templates/element/*` |
| `app/View/Emails/*` | `templates/email/*` |
| `app/View/Errors/*` | `templates/Error/*` |
| `app/View/Helper/*` | `src/View/Helper/*` |
| `app/Console/*` | `src/Console/*` |
| `app/Console/cake` | `bin/cake` |
| `app/Lib/<Type>/*` (例: `app/Lib/Error/`) | `src/<Type>/*` (`src/Error/`)。名前空間付きのクラスの場合 |
| `app/Lib/*` | `src/Lib/*` |
| `app/Locale/*` | `resources/locales/*` |
| `app/Test/*` | `tests/*` |
| `app/Plugin/*` | `plugins/*` (Composer を使う) |
| `app/Vendor/*` | `vendor/*` (Composer を使う) |
| `app/tmp/logs/*` | `logs/*` |
| `app/tmp/*` | `tmp/*` |
| `app/webroot/*` | `webroot/*` |

`.ctp` のテンプレートは、名前も CakePHP 2 のディレクトリ (`templates/Layouts/`、`templates/Elements/` など) も
そのままにできます。2.13 からは、CakePHP は `.php` のテンプレートと CakePHP 5 のディレクトリも読み込みます。
スキャフォールドのテンプレート (`Scaffolds/`) は `.ctp` のままにします。

モダン構成の `tests/` では、`Test/Case/` と `Test/Fixture/` の代わりに `tests/TestCase/` と
`tests/Fixture/` を使います。テストスイートとフィクスチャローダーはどちらも検出します。

## フレームワークが構成を判別する仕組み {#how-the-framework-finds-the-layout}

パスの定数は、アプリケーションの `config/define.php` とフレームワーク自身の `config/define.php`
で定義されます。アプリケーション側で定義していない定数は、フレームワークが構成を見て決めます。

- `CONFIG`: `ROOT/config/` があればそこ、無ければ `ROOT/APP_DIR/Config/`
- `TMP`: `ROOT/tmp/` があればそこ、無ければ `APP/tmp/`
- `LOGS`: `ROOT/tmp/` があれば `ROOT/logs/`、無ければ `TMP/logs/`
- `TESTS`: `ROOT/tests/` があればそこ、無ければ `APP/Test/`
- テンプレートは `ROOT/templates/` からも、ロケールは `ROOT/resources/locales/` からも探します

モダンなスケルトンの `config/define.php` では、主に次の定数を設定しています。

```php
define('ROOT', dirname(__DIR__));
define('APP_DIR', 'src'); // 'app' の代わり
define('WEBROOT_DIR', 'webroot');
define('WWW_ROOT', ROOT . DS . WEBROOT_DIR . DS);
define('VENDORS', ROOT . DS . 'vendor' . DS);
```

`webroot/index.php`、`webroot/test.php`、`bin/cake` がこのファイルを読み込みます。
既存のアプリケーションコード（コントローラー、モデル、ビュー）は、どちらの構成でも変更せずに動作し、
移行の間も従来のファイルの場所は引き続き使えます。

## pieceofcake2/upgrade で移行する {#migrating-with-pieceofcake2-upgrade}

[pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/ja/) は `git mv` でファイルを移動し、Rector で
コードを書き換えます。実行する前にアプリケーションをコミットし、コマンドごとに結果を確認してコミットしてください。
変更後のコードには pieceofcake2/cakephp 2.13 以降が必要です。

1. プラグインを開発用の依存パッケージとしてインストールし、`app/Config/bootstrap.php` で読み込みます
   (pieceofcake2/upgrade の [インストール](https://pieceofcake2.github.io/upgrade/ja/#installation) を参照)。

   ```bash
   composer require --dev pieceofcake2/upgrade
   ```

   ```php
   CakePlugin::load('Upgrade');
   ```

2. アプリケーションをモダン構成に移します。テンプレートは CakePHP 5 のディレクトリにある `.php` ファイルになります。

   ```bash
   app/Console/cake upgrade layout --dry-run
   app/Console/cake upgrade layout
   composer dump-autoload
   ```

3. クラスを `App` 名前空間に移します。このコマンドは `config/core.php` に `App.namespace` を追加し、
   PSR-4 で `src/` から `App\` を、`tests/` から `App\Test\` を読み込むようにもします。

   ```bash
   bin/cake upgrade namespace --dry-run
   bin/cake upgrade namespace
   composer dump-autoload
   ```

4. コマンドが手作業で行うように示したことを済ませ、テストを実行して、`git diff` で変更を確認します。
5. プラグインを削除します。`composer remove --dev pieceofcake2/upgrade` を実行し、`config/bootstrap.php` に
   移った `CakePlugin::load('Upgrade');` の行を削除します。

コマンドが何を変更するか、`app/` のないアプリケーションやプラグインの場合については、pieceofcake2/upgrade の
ドキュメントの [構成と名前空間](https://pieceofcake2.github.io/upgrade/ja/#layout-and-namespace) をご覧ください。

## 手作業で移行する {#migrating-by-hand}

手作業で移行するには、次の手順に従います。

### 1. composer.json を更新する {#1-update-composer-json}

```json
{
    "require": {
        "php": ">=8.0",
        "pieceofcake2/cakephp": "^2.13"
    },
    "autoload": {
        "classmap": ["src/"]
    },
    "config": {
        "vendor-dir": "vendor/",
        "sort-packages": true,
        "allow-plugins": {
            "composer/installers": true
        }
    },
    "extra": {
        "installer-paths": {
            "plugins/{$name}/": ["type:cakephp-plugin"]
        }
    }
}
```

これまでプラグインを `app/Plugin/` にインストールしていた場合、`installer-paths` の設定で
インストール先が `plugins/` に変わります。`app/Plugin/` のプラグインを削除してから
`composer install` を実行し、`plugins/` にインストールし直してください。

### 2. ディスパッチャーのファイルを更新する {#2-update-the-dispatcher-files}

[pieceofcake2/app](https://github.com/pieceofcake2/app) (グローバルなクラスなら
[pieceofcake2/app-legacy](https://github.com/pieceofcake2/app-legacy)) から `config/define.php`、`webroot/index.php`、
`webroot/test.php`、`bin/cake` をプロジェクトにコピーします。`config/define.php` はモダン構成のパス定数を設定し
([フレームワークが構成を判別する仕組み](#how-the-framework-finds-the-layout) を参照)、他のファイルはこれを読み込みます。

### 3. ファイルを移動する {#3-move-the-files}

[移行対応表](#file-migration-map) に従ってファイルを移動します。たとえば次の順に、少しずつ進められます。

1. 設定ファイル: `app/Config/*` → `config/*`
2. PHP コード: `app/Controller/*`、`app/Model/*` → `src/`
3. テンプレート: `app/View/**/*.ctp` → `templates/`。CakePHP 5 の名前にするなら、テンプレートを `.php` に変え、
   `Layouts/`、`Elements/`、`Emails/`、`Errors/` を `layout/`、`element/`、`email/`、`Error/` に名前を変え、
   メールのレイアウトを `Layouts/Emails/` から `layout/email/` に移します。スキャフォールドのテンプレート
   (`Scaffolds/`) は `.ctp` のままにします。
4. テスト: `app/Test/*` → `tests/`
5. 依存パッケージ: プラグインとベンダーは Composer で管理

### 4. アプリケーションが引き続き動作することを確認する {#4-verify-that-the-application-still-works}

テストを実行し、アプリケーションのページを確認します。

`"classmap": ["src/"]` を設定して `composer dump-autoload` を実行したら、コントローラー、モデル、ヘルパー、
シェルから `App::uses()` を削除します。`App::uses()` は 2.13.0 で非推奨になりました。

```php
// 変更前
App::uses('AppController', 'Controller');

class UsersController extends AppController
{
    // ...
}
```

```php
// 変更後（Composer のオートロードを使用）
class UsersController extends AppController
{
    // ...
}
```

::: warning
次の手順で PSR-4 を使うまでは、新しいクラスファイルを作るたびに `composer dump-autoload` を実行してください。
classmap を再生成しないと、新しいクラスは見つかりません。
プラグインは引き続き `CakePlugin::load()` で読み込む必要があります。Composer がオートロードしない
プラグインは、classmap にプラグインのフォルダを追加してください。
:::

### 5. クラスを名前空間に移す (任意) {#5-move-the-classes-into-a-namespace-optional}

各クラスにディレクトリに対応する名前空間を宣言し (`src/Controller/` なら `App\Controller`、
`tests/TestCase/Controller/` なら `App\Test\TestCase\Controller`)、CakePHP とアプリケーションのクラスを `use` 文で
インポートします。

```php
// src/Controller/UsersController.php
namespace App\Controller;

class UsersController extends AppController
{
    // ...
}
```

`config/core.php` の `App.encoding` の設定より前で、CakePHP に名前空間を設定します
(`Cache::config('_cake_core_')` より前である必要があります)。

```php
/**
 * The namespace of the classes of the application
 */
Configure::write('App.namespace', 'App');
```

それから、classmap の代わりに PSR-4 でクラスを読み込むようにして、`composer dump-autoload` を実行します。

```json
"autoload": {
    "psr-4": {
        "App\\": "src/"
    }
},
"autoload-dev": {
    "psr-4": {
        "App\\Test\\": "tests/"
    }
},
```

名前空間付きのコードでは `Lib/` を使いません。`src/Lib/<Type>/` は `src/<Type>/` に移します
(例: `App\Error\AppExceptionRenderer` は `src/Error/AppExceptionRenderer.php`)。`config/` とテンプレートは
名前空間を持たないままで、使うクラスを `use` 文でインポートします。古いクラス名が残っているので、
`tmp/cache/` のキャッシュを削除してください。[名前空間のあるクラス](/ja/development/namespaced-classes) もご覧ください。

## CakePHP 5 へのアップグレード (Step 4) {#upgrading-to-cakephp-5-step-4}

アプリケーションがモダン構成の CakePHP 2 で動作するようになれば、CakePHP 5 へのアップグレードはコードの変更だけになります。

- ディレクトリ構成はすでに正しいので、ファイルを整理し直す必要はありません。
- テンプレートはすでに CakePHP 5 のディレクトリにある `.php` ファイルなので、移動する必要はありません。
- クラスは CakePHP 5 と同じく、すでに `App` 名前空間にあります。
- Composer は `vendor/` と `plugins/` ですでにセットアップされています。

残っている作業は、`composer.json` で CakePHP 5.x を require し、API の変更に合わせてコードを直し、互換性の問題を
修正することです。[CakePHP 5.x 移行ガイド](https://book.cakephp.org/5/ja/appendices/5-0-migration-guide.html)
を参照してください。
