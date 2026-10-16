# End-to-End テスト {#e2e-テスト}

End-to-End テストは、アプリケーションを本物のブラウザーで開き、ユーザーと同じように
操作します。リンクをクリックし、フォームに入力し、ボタンを押して、画面に表示される内容を確認します。
[pieceofcake2/panther](https://pieceofcake2.github.io/panther/) パッケージは、
[Symfony Panther](https://symfony.com/doc/current/testing/end_to_end.html) を Symfony の
アプリケーションと同じ使い方で CakePHP 2 アプリケーションに導入します。
`static::createPantherClient()` が PHP のビルトイン Web サーバーでアプリケーションを起動して
Chrome を操作し、テストケースは `CakeTestCase` なので、`$fixtures` はほかのテストと同じように
使えます。

このページはセットアップの簡単なガイドです。設定、クライアント、アサーションの詳細は
パッケージのドキュメント ([Web サイト](https://pieceofcake2.github.io/panther/)、
[Markdown](https://github.com/pieceofcake2/panther/blob/1.x/docs/README.md)、英語) を
参照してください。[Symfony のドキュメント](https://symfony.com/doc/current/testing/end_to_end.html)
もそのまま当てはまります。

<a id="when-to-use-e2e-tests"></a>

## End-to-End テストを使う場面 {#e2e-テストを使う場面}

CakePHP アプリケーションのテストは、通常いくつかのレベルで行います
([テストの種類](/ja/development/testing#types-of-tests) を参照してください)。

| | ユニットテスト | コントローラーのテスト (`testAction()`) | [統合テスト](/ja/development/integration-testing) | End-to-End テスト |
| --- | --- | --- | --- | --- |
| 実行場所 | PHPUnit のプロセス内 | PHPUnit のプロセス内 (HTTP なし) | PHPUnit のプロセス内 (HTTP なし) | 本物の HTTP サーバーと本物の Chrome |
| 速度 | 数ミリ秒 | 1 リクエストあたり数ミリ秒 | 1 リクエストあたり数ミリ秒 | 1 テストあたり数秒 |
| JavaScript、CSS、レイアウト | なし | なし | なし | あり |
| Cookie、セッション、リダイレクト | - | シミュレーション | 本物のディスパッチャーを通したシミュレーション | 本物 |
| `SecurityComponent` のトークン | - | 通常は無効化またはモック | 送信するデータから生成 | 実際のフォームの本物のトークン |
| ビュー変数、戻り値 | 確認できる | 確認できる | 確認できる | 確認できない (ユーザーに見えるものだけ) |
| 準備 | 不要 | 不要 | 不要 | Chrome、chromedriver、Web サーバー用のテストデータベース |

End-to-End テストが適しているのは次のような場合です。

- 会員登録、購入手続き、複数ステップのフォームなど、複数のページにまたがる操作の流れ
- JavaScript、レイアウト、レスポンシブデザイン、ブラウザーに依存する機能
- `SecurityComponent` で保護されたフォームや、テンプレートで使うヘルパー。コントローラーのテストでは
  通常トークンの検証を回避し、統合テストでは実際のフォームではなく送信するデータからトークンを
  計算するため、フォームの誤りはブラウザーでしか見つかりません。

モデル、コンポーネント、コントローラーのロジックや、ステータスコード、境界条件は、ユニットテストと
統合テストで確認してください。End-to-End テストはどれもアプリケーション全体をブラウザーで
動かすので、数を絞り、主要な操作の流れに集中させましょう。

<a id="requirements"></a>

## 動作要件

- PHP 8.0 以降 (PHP 8.0 では Panther 2.1、PHP 8.1 以降では Panther 2.4)
- pieceofcake2/cakephp 2.12 以降と PHPUnit 9.6
- Google Chrome または Chromium と、同じメジャーバージョンの chromedriver

<a id="installation"></a>

## インストール

パッケージと [bdi](https://github.com/dbrekelmans/bdi) を開発用の依存としてインストールし、
bdi で Chrome に合った chromedriver をダウンロードします。

```bash
composer require --dev pieceofcake2/panther dbrekelmans/bdi
vendor/bin/bdi detect drivers
```

bdi はドライバーを `drivers/` に保存し、Panther はそこからドライバーを探します。`/drivers/` を
`.gitignore` に追加してください。Chrome が自動で更新されたら `vendor/bin/bdi detect drivers` を
もう一度実行します。パッケージマネージャーでインストールした chromedriver も使えます。

このパッケージは CakePHP のプラグインではなくライブラリ (名前空間 `PieceOfCake2\Panther\`) なので、
`CakePlugin::load()` で読み込むものはありません。

<a id="setup"></a>

## セットアップ

<a id="phpunit-configuration"></a>

### PHPUnit の設定

Symfony のアプリケーションと同じように、End-to-End テストはほかのテストと並べて
`tests/TestCase/EndToEnd/` (従来の構成では `app/Test/Case/EndToEnd/`) に置き、同じ
`phpunit.xml.dist` から実行し、Panther の `PANTHER_*` 変数を `<server>` 要素で設定します。
[pieceofcake2/app](https://pieceofcake2.github.io/app/) スケルトンにはこの設定が含まれているので、
拡張のコメントを外してください。

```xml
<phpunit bootstrap="./tests/bootstrap.php"
         backupGlobals="true"
         defaultTestSuite="App"
         stderr="true"
         colors="true">
    <php>
        <!-- 失敗した End-to-End テストのスクリーンショット -->
        <server name="PANTHER_ERROR_SCREENSHOT_DIR" value="./tmp/error-screenshots"/>
    </php>
    <!-- 実行中ずっと Web サーバーとブラウザーを動かしておく -->
    <extensions>
        <extension class="PieceOfCake2\Panther\ServerExtension"/>
    </extensions>
    <listeners>
        <listener class="CakeFixtureInjector" file="./vendor/pieceofcake2/cakephp/src/Cake/TestSuite/Fixture/CakeFixtureInjector.php"/>
    </listeners>
    <testsuites>
        <testsuite name="App">
            <directory>./tests/TestCase/</directory>
            <exclude>./tests/TestCase/EndToEnd/</exclude>
        </testsuite>
        <testsuite name="EndToEnd">
            <directory>./tests/TestCase/EndToEnd/</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

- `defaultTestSuite="App"` により、`vendor/bin/phpunit` は `App` スイートだけを実行するので、
  速いままで Chrome も不要です。この属性を削除すると、Symfony のアプリケーションと同じように
  `vendor/bin/phpunit` がすべてを実行します。
- テストが `createPantherClient()` を呼ぶまで何も起動しないので、`App` スイートの実行では
  ブラウザーは起動しません。拡張は、最初の End-to-End テストから最後のテストまで Web サーバーと
  ブラウザーを動かしておきます。
- `<server>` 要素は同じ名前のシェルの変数を置き換えます。`PANTHER_NO_HEADLESS=1` のような
  マシンごとの値は、シェルか、`phpunit.xml.dist` をコピーした `phpunit.xml` に書いてください。

`PANTHER_WEB_SERVER_PORT` や `PANTHER_WEB_SERVER_DIR` などのほかの変数と、`createPantherClient()`
のオプションは、パッケージのドキュメントの
[Configuration](https://pieceofcake2.github.io/panther/configuration)
([Markdown](https://github.com/pieceofcake2/panther/blob/1.x/docs/configuration.md)) に
まとめられています。

<a id="the-test-database"></a>

### テスト用データベース

Web サーバーは別の PHP プロセスで、ブラウザーに見えるのは Web サーバーが読み込むデータだけです。
そのため Web サーバーの `default` 接続は、`CakeTestCase` がフィクスチャを読み込むデータベース
(`test` データソースのデータベース) でなければなりません。Symfony と同じように、これは `test` 環境で
決まります。pieceofcake2/app のテストはデフォルトで `APP_ENV=test` で動きます (`APP_ENV` が設定されて
いなければ、`tests/bootstrap.php` が `putenv()`、`$_ENV`、`$_SERVER` で設定します)。そのため、テストはコミットされている `config/.env.test` を読み込みます。Panther が
起動する Web サーバーはテストのプロセスの環境を引き継ぐので、`APP_ENV=test` で動き、同じファイルを
読み込みます。このファイルでは `DATABASE_URL` がテスト用データベースです。

```dotenv
# config/.env.test
DATABASE_URL="mysql://user:password@localhost:3306/test_database_name?encoding=utf8mb4"
```

`config/database.php` は `DATABASE_URL` を両方の接続に渡すので、ページが使う `default` 接続は
フィクスチャのデータベースになります。

```php
public function __construct()
{
    $this->default['dsn'] = env('DATABASE_URL');
    $this->test['dsn'] = env('DATABASE_URL');
}
```

自分のマシンのテスト用データベースは `config/.env.test.local` か、実際の環境変数 (CI など) で設定して
ください。実際の環境変数は Web サーバーにも引き継がれ、両方のプロセスで `config/.env.test` より優先され
ます。Panther の `PANTHER_APP_ENV` が必要になるのは、Web サーバーにテストとは異なる `APP_ENV` を渡す
ときだけです。`config/database.php` に、`test` 接続を `default` として使う `APP_ENV` の分岐が残って
いる場合は削除してください。

- テストに開発用のデータベースを使わないでください。
- 表示するページが使うテーブルは、すべて `$fixtures` に指定してください。
- テストと Web サーバーは同じデータベースを使います。テストに必要なレコードはページを開く前に
  モデルで作成し、フォームの結果はページだけでなくデータベースでも確認してください。
- SQLite の `:memory:` はプロセス間で共有できません。データベースファイルかデータベースサーバーを
  使ってください。

詳しくはパッケージのドキュメントの
[Test Database](https://pieceofcake2.github.io/panther/test-database) を参照してください。

<a id="writing-tests"></a>

## テストを書く

`PieceOfCake2\Panther\TestSuite\PantherTestCase` を継承し、`static::createPantherClient()` で
ブラウザーを取得します。

```php
<?php
declare(strict_types=1);

namespace App\Test\TestCase\EndToEnd;

use ClassRegistry;
use PieceOfCake2\Panther\TestSuite\PantherTestCase;

class ArticlesTest extends PantherTestCase
{
    /**
     * @var array<string>
     */
    public $fixtures = ['app.article'];

    /**
     * @return void
     */
    public function testIndex(): void
    {
        $client = static::createPantherClient();
        $client->request('GET', '/articles');

        $this->assertPageTitleContains('Articles');
        $this->assertSelectorTextContains('table', 'First article');
    }

    /**
     * @return void
     */
    public function testAdd(): void
    {
        $client = static::createPantherClient();
        $client->request('GET', '/articles/add');
        $client->submitForm('Submit', [
            'data[Article][title]' => 'Written in the browser',
            'data[Article][body]' => 'Hello',
        ]);

        $this->assertSelectorTextContains('.message', 'The article has been saved.');
        $this->assertSame(1, ClassRegistry::init('Article')->find('count', [
            'conditions' => ['Article.title' => 'Written in the browser'],
        ]));
    }
}
```

クライアントは Panther のものです。`request()`、`clickLink()`、`submitForm()`、`waitFor()`、
`executeScript()`、`takeScreenshot()` などは
[Symfony のドキュメント](https://symfony.com/doc/current/testing/end_to_end.html) のとおりに
動きます。アサーションは Symfony と Panther の `WebTestAssertionsTrait` のもので、
`assertSelectorTextContains()`、`assertSelectorIsVisible()`、`assertSelectorWillContain()` などが
あります。CakePHP では次の点に注意してください。

- `submitForm()` には、`data[User][email]` のような `FormHelper` の完全なフィールド名を
  指定します。ブラウザーが本物のフォームを送信するので、`SecurityComponent` で保護された
  フォームも動きます。
- ブラウザーは Cookie を含めて次のテストに引き継がれます。ログアウトした状態で始める必要がある
  テストでは、`$client->getCookieJar()->clear()` で Cookie を削除してください。
- JavaScript の結果は `waitFor()` か `assertSelectorWill*()` のアサーションで待ちます。
  `sleep()` は使わないでください。

フォーム、待機、複数のブラウザー、スマートフォンの表示領域、アサーションの一覧は、パッケージの
ドキュメントの [Writing Tests](https://pieceofcake2.github.io/panther/writing-tests) を
参照してください。

<a id="failure-screenshots-and-html"></a>

## 失敗時のスクリーンショット

`PANTHER_ERROR_SCREENSHOT_DIR` と拡張を設定すると、ブラウザーを作成した後に失敗またはエラーになった
テストは、ブラウザーごとのスクリーンショットを `tmp/error-screenshots/` に保存します。
`/tmp/error-screenshots/` を `.gitignore` に追加してください (pieceofcake2/app には追加済みです)。

![お問い合わせフォームのテストが失敗したときに保存されたスクリーンショット](/e2e-failure-screenshot.png)

<a id="running-the-tests"></a>

## テストの実行

```bash
# App スイートだけ (defaultTestSuite)。Web サーバーもブラウザーも起動しない
vendor/bin/phpunit

# End-to-End テスト
vendor/bin/phpunit --testsuite EndToEnd

# すべてのテスト
vendor/bin/phpunit --testsuite App,EndToEnd

# 1 つのテストケース
vendor/bin/phpunit --testsuite EndToEnd --filter ArticlesTest

# ブラウザーの動きを見て、失敗したら Enter を押すまで止める
PANTHER_NO_HEADLESS=1 vendor/bin/phpunit --testsuite EndToEnd --debug
```

`--filter` が探すのは選択したスイートの中だけです。`--testsuite EndToEnd` を付けないと `App` スイートを
探すので、End-to-End テストは見つかりません。パスを引数に指定すると、`defaultTestSuite` に関係なく
そのファイルやディレクトリのテストを実行します。

<a id="continuous-integration"></a>

## 継続的インテグレーション

GitHub Actions では、End-to-End テストを別のジョブで実行します。ユニットテストのジョブは引き続き
`vendor/bin/phpunit` (`App` スイート) をブラウザーなしで実行します。`ubuntu-latest` のランナーには
Chrome があり、`vendor/bin/bdi detect drivers` が対応する chromedriver をインストールします。
主なステップは次のとおりです。

```yaml
  end-to-end:
    name: End-to-End
    runs-on: ubuntu-latest

    steps:
      # ... checkout, setup-php, composer install, the test database

      - name: Install chromedriver for the installed Chrome
        run: vendor/bin/bdi detect drivers

      - name: Run End-to-End tests
        env:
          PANTHER_NO_SANDBOX: 1
        run: vendor/bin/phpunit --testsuite EndToEnd

      - name: Upload the error screenshots
        if: ${{ failure() }}
        uses: actions/upload-artifact@v7
        with:
          name: error-screenshots
          path: tmp/error-screenshots/
          retention-days: 7
          if-no-files-found: ignore
```

GitHub のランナーでは `PANTHER_NO_SANDBOX=1` が必要です。ビルドで CSS や JavaScript のファイルを
作成する場合は、End-to-End テストのステップの前にビルドしてください。`phpunit.xml.dist` から
`defaultTestSuite` を削除した場合は、ユニットテストのジョブを `--testsuite App` で実行してください。

ジョブ全体は [継続的インテグレーション](/ja/development/continuous-integration#e2e-tests) に
あります。パッケージには
[ジョブサマリー付きのワークフロー](https://github.com/pieceofcake2/panther/blob/1.x/docs/examples/end-to-end.yml)
もあります ([CI with GitHub Actions](https://pieceofcake2.github.io/panther/ci))。

<a id="troubleshooting"></a>

## トラブルシューティング

**"This version of ChromeDriver only supports Chrome version N"**: Chrome が更新されました。
`vendor/bin/bdi detect drivers` をもう一度実行してください。

**Docker や CI で Chrome が起動時にクラッシュする**: `PANTHER_NO_SANDBOX=1` を設定してください。

**`--filter` でテストが見つからない**: `--testsuite EndToEnd` を付けてください。
`defaultTestSuite="App"` の場合、`--filter` は `App` スイートの中だけを探します。

**ブラウザーに "Missing Database Table" が表示される**: Web サーバーがテスト用データベースを
使っていないか、`$fixtures` にテーブルが足りません。テストが `APP_ENV=test` で動いていること (環境変数に
ほかの `APP_ENV` が無いこと) と、`config/.env.test` (または環境変数) の `DATABASE_URL` がテスト用データベースで
あることを確認してください。

そのほかは、パッケージのドキュメントの
[Troubleshooting](https://pieceofcake2.github.io/panther/troubleshooting) を参照してください。
