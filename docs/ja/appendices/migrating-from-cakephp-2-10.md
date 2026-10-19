# CakePHP 2.10 からの移行

このガイドでは、本家の CakePHP 2.10.24 から pieceofcake2/cakephp へアプリケーションを移行する手順と、
破壊的変更を説明します。内容は [UPGRADE.md](https://github.com/pieceofcake2/cakephp/blob/2.x/UPGRADE.md) にもとづいています。

## 前提条件

このフォークに移行する前に、次の条件を満たしていることを確認してください。

- アプリケーションが PHP 7.4 で動作している
- CakePHP 2.10.24 を使っている（それより前のバージョンはサポートしません）
- 依存関係を Composer で管理している

## 本家 CakePHP 2.x からの移行

1. **まず CakePHP 2.10.24 に上げる**: それより古いバージョンを使っている場合は、PHP 7.4 上で
   `cakephp/cakephp:2.10.24` に更新します
2. **PHP 7.4 での互換性を確保する**: PHP 8.x に移行する前に、アプリケーションが PHP 7.4 で
   完全に動作している必要があります
3. **Composer の設定を変更する**: `composer.json` の `cakephp/cakephp` を
   `pieceofcake2/cakephp` に置き換え、プラグインが `app/Plugin/`（モダン構成では `plugins/`）にインストールされるよう
   `composer/installers` の設定を追加します。この設定は必須です。
   [composer.json の設定](/ja/installation#configuring-composer-json) を参照してください。
4. **PHP をアップグレードする**: PHP 8.0 以降に更新します
5. **十分にテストする**: アプリケーションのテストスイートを実行し、互換性を確認します

## v2.13.0

v2.13.0 の変更のうち、すでにフォークで動いているアプリケーションに影響する可能性があるものです。

### 名前空間のあるクラス: `App::className()` と `App.namespace` ([PR #91](https://github.com/pieceofcake2/cakephp/pull/91)) {#app-classname}

- サフィックスの付く設定に、サフィックスの付いた完全なクラス名を書けます。キャッシュエンジンなら
  `MemoryEngine::class`、トランスポートなら `SesTransport::class` です。Auth のオブジェクト、
  パスワードハッシャー、ビュークラス、コンポーネント、ヘルパー、ビヘイビア、シェルのタスク、
  シェルヘルパーも同じです。サフィックスを省いた名前 (`'Acme\Cache\Memory'`) も、これまでどおり
  使えます。
- `App.namespace` は新しい設定で、既定では設定されていません。設定すると、コントローラー、シェル、
  モデルと上のクラスを、まずその名前空間から短い名前で探します。たとえば `/posts` なら
  `App\Controller\PostsController` です。
  [名前空間のあるクラス](/ja/development/namespaced-classes) を参照してください。

> [!WARNING]
> 名前空間のあるクラスの `Controller::$name`、`Model::$name`、`Shell::$name` は、名前空間を含まなくなります。
> たとえば `Acme\Model\Post` ではなく `Post` です。前の値に頼っていた場合は `$name` を書いてください。

## v2.12.11

### セキュリティ修正: 先頭のスラッシュが多いリクエスト URL ([GHSA-wh43-gc93-wh4c](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-wh43-gc93-wh4c), [PR #90](https://github.com/pieceofcake2/cakephp/pull/90)) {#request-url-leading-slashes}

- `CakeRequest` は、リクエスト URL の先頭のスラッシュとバックスラッシュを、最初の 1 つだけでなくすべて取り除きます。
  `//posts/view/1` へのリクエストはこれまでどおりルーティングされますが、`$this->request->url` は `/posts/view/1`
  ではなく `posts/view/1` に、`here` はプロトコル相対の `//posts/view/1` ではなく `/posts/view/1` になります。
  `here()` も、返す値の先頭のスラッシュを 1 つにまとめます。
  先頭のスラッシュが 1 つの URL は変わりません。[脆弱性への対応](/ja/security/fixes#request-here) を参照してください。

### HTML5 の script、link、style のタグ ([PR #86](https://github.com/pieceofcake2/cakephp/pull/86)) {#html5-tags}

- HtmlHelper は HTML5 のタグを出力します: `<script src="...">`、`<link rel="stylesheet" href="...">`、`<style>`、
  `type="text/javascript"` も CDATA も付けないスクリプトブロック。ブラウザはどちらも同じに扱うので、ページの見た目や
  動きは変わりません。
- ページやヘルパーの HTML を比べるテストは、新しいタグに合わせる必要があるかもしれません。
- XHTML で配信するページなど、元の出力のままにするには、ヘルパーの設定で `html5` を `false` にします。

  ```php
  public $helpers = ['Html' => ['html5' => false]];
  ```

  [HTML5 のタグ](/ja/core-libraries/helpers/html#html5) を参照してください。

## v2.12.10

v2.12.10 の変更のうち、すでにフォークで動いているアプリケーションに影響する可能性があるものです。

### 新しい関数 `raw()` ([PR #84](https://github.com/pieceofcake2/cakephp/pull/84)) {#raw}

- `raw()` は `h()` の逆で、値をそのまま返します。わざとエスケープせずに HTML を出力するときに使います。
  アプリケーションが組み立てた HTML や無害化した HTML をビューで出力する箇所では、そのままの `echo` の代わりに
  使ってください。

  ```php
  <?= raw($post['Post']['body_html']) ?>
  ```

- ほかのグローバル関数と同じく、すでにグローバルな `raw()` を定義しているアプリケーションやパッケージでは、その
  関数が使われます。同じ動きをするか確かめるか、名前を変えてください。
  [raw()](/ja/core-libraries/global-constants-and-functions#raw) を参照してください。

## v2.12.7

v2.12.7 の変更のうち、すでにフォークで動いているアプリケーションに影響する可能性があるものです。

### DSN による設定: `DATABASE_URL` と `MAILER_DSN` ([PR #69](https://github.com/pieceofcake2/cakephp/pull/69)) {#dsn-configuration}

- `DATABASE_CONFIG` のデータソースと `EmailConfig` のプロファイルは、新しい `'dsn'` キーに DSN を書いて
  接続全体を設定できます。値は Symfony と同じ環境変数から読み込みます。

  ```php
  // config/database.php
  public $default = [
      'datasource' => 'Database/Mysql',
      'encoding' => 'utf8mb4',
  ];

  public function __construct()
  {
      $this->default['dsn'] = env('DATABASE_URL');
  }
  ```

  ```php
  // config/email.php
  public $default = [
      'from' => 'app@example.com',
  ];

  public function __construct()
  {
      $this->default['dsn'] = env('MAILER_DSN');
  }
  ```

  `DATABASE_URL` は `mysql://user:secret@localhost:3306/app` のような DSN で、`MAILER_DSN` は
  `smtp://user:secret@smtp.example.com:587` や `null://null` のような Symfony Mailer の形式です。
- DSN から読み取ったキーは配列の同じキーを置き換え、ほかのキーはそのまま残ります。`'dsn'` が空なら
  無視されるので、このキーを使わない設定はこれまでどおりに動きます。
- プラグインは `ConnectionManager::setDsnClassMap()` と `CakeEmail::setDsnClassMap()` で DSN のスキームを
  追加したり、組み込みのスキームを置き換えたりできます。pieceofcake2/aws と pieceofcake2/async-aws は
  Amazon SES のスキーム (`ses+api://`、`ses+smtp://` など) を登録します。
- アプリケーションスケルトン (pieceofcake2/app) は `DATABASE_URL` と `MAILER_DSN` を読み込み、テストでは
  `config/.env.test` を読み込みます。app v2.5.0 からは、Symfony と同じように symfony/dotenv で `.env` ファイルを
  読み込みます ([.env ファイルの読み込み](/ja/development/configuration#loading-env-files) を参照)。

[データベースの DSN](/ja/development/configuration#database-url)、
[メーラー DSN](/ja/core-utility-libraries/email#mailer-dsn)、
[DSN スキームの登録](/ja/plugins/how-to-create-plugins#registering-dsn-schemes) をご覧ください。

## v2.12.6

v2.12.6 の変更のうち、すでにフォークで動いているアプリケーションに影響する可能性があるものです。

### セキュリティ修正: 'cipher' 暗号化を非推奨に ([PR #62](https://github.com/pieceofcake2/cakephp/pull/62)) {#cipher-deprecated}

`Security::cipher()` と、`CookieComponent` の `'cipher'` タイプ (デフォルトのタイプ) は、
`Security.cipherSeed` をどの値にしても安全ではありません
([GHSA-65v5-h65v-8pgg](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-65v5-h65v-8pgg))。
これらを非推奨にしました。値の暗号化は以前とまったく同じで、例外も投げませんが、次の点が変わります。

- プロセス (1 回のリクエストや 1 回のコンソール実行) の中で最初に `'cipher'` を使うと、デバッグモードが
  オフでも `warning` をログに記録します。デバッグモードがオンのときは、同じメッセージを
  `E_USER_DEPRECATED` としても発生させます。`'aes'` と `'rijndael'` のタイプや、暗号化しない Cookie では
  何も記録しません。
- `Security.cipherSeed` が 1 から 4294967295 までの数字だけの値でない場合は、最初に `Security::cipher()` を
  呼び出したときに `warning` をログに記録し (シードの値は記録しません)、デバッグモードがオンのときは
  `Debugger::checkSecurityKeys()` (`AuthComponent` から呼ばれます) が `E_USER_NOTICE` を発生させます。
  デフォルトのシードの変更を求める通知は、以前のデフォルト値 `'76859309657453542496749683645'` と
  新しいデフォルト値 `'2659758734'` の両方で表示されるようになりました。
- コアの代替の `config/core.php` (アプリケーションに `config/core.php` がない場合にだけ使われます) は、
  `Security.cipherSeed` を `'2659758734'` に、`Security.useOpenSsl` を `true` に設定します。

**移行方法:**

- 警告を消すためだけに `Security.cipherSeed` を変更しないでください。シードを変えると既存の
  `'cipher'` の Cookie が読めなくなり、`'cipher'` は安全でないままです。
  [`'aes'` への移行](/ja/development/configuration#migrating-to-aes) の説明に従って `'aes'` に
  移行してください。
- テストで非推奨の警告や通知を例外にしている場合、テスト実行中の最初の `'cipher'` の使用や、
  デバッグモードがオンで範囲外のシードのときの `AuthComponent` は失敗するようになります。
  アプリケーションを `'aes'` に移行したら `cookieEncrypted()` / `assertCookieEncrypted()` のモードに
  `'aes'` を渡すか、テストの設定で範囲内のシードを指定してください。
- `Security::cipher()` を直接呼び出している場合は、32 バイト以上のキーを使う `Security::encrypt()` /
  `Security::decrypt()` に置き換えてください。

[脆弱性への対応](/ja/security/fixes#cipher) と
[cipherSeed と Cookie の 'cipher' タイプ](/ja/development/configuration#cipher-seed) を参照してください。

### CookieComponent が `Cookie.type` と `Cookie.key` を読む ([PR #62](https://github.com/pieceofcake2/cakephp/pull/62)) {#cookie-type-key}

- `'key'` の設定がない場合、`CookieComponent` は Configure の `Cookie.key` が空でない文字列ならその値を、
  そうでなければ以前と同じく `Security.salt` を使います。
- `'type'` の設定がない場合は、Configure の `Cookie.type` が設定されていればその値を、そうでなければ
  以前と同じく `'cipher'` を使います。不正な値を設定すると、これまでと同じ `E_USER_WARNING` が発生し、
  `'cipher'` になります。
- 明示的な `'key'` / `'type'` の設定、`$this->Cookie->key = ...`、`$this->Cookie->type(...)` が常に
  優先されます。
- `IntegrationTestTrait` の `cookieEncrypted()` / `assertCookieEncrypted()` も同じデフォルトのキーを
  使います。

どちらも設定していないアプリケーションでは何も変わりません。アプリケーション全体を `'aes'` に
切り替えるには、`config/core.php` で次のように設定します。

```php
Configure::write('Security.useOpenSsl', true);
Configure::write('Cookie.type', 'aes');
Configure::write('Cookie.key', env('COOKIE_KEY') ?: null);
```

`'aes'` には 32 バイト以上のキーが必要です。`'cipher'` のまま `Cookie.key` を設定するとキーが変わるため、
既存の `'cipher'` の Cookie は読めなくなります。[Cookie](/ja/core-libraries/components/cookie) を
参照してください。

### CakePHP 5 風のデバッグ用エラーページ ([PR #60](https://github.com/pieceofcake2/cakephp/pull/60)) {#dev-error-pages}

- デバッグモードでは、コアのエラーページが CakePHP 5 から移植した新しい `dev_error` レイアウトを
  使います。すべてのスタックフレームのコードの抜粋と、エディターでファイルを開く `(edit)` リンクを
  表示します (デフォルトは PhpStorm です。`Debugger.editor` と、コンテナーでは
  `Debugger.editorBasePath` を設定してください)。
  [デバッグ用のエラーページ](/ja/development/debugging#debug-error-pages) を参照してください。
- 本番モード (`debug` = 0)、AJAX リクエスト、コンソールでは何も変わらず、既存の PHP の API も
  変わっていません。アプリケーション、プラグイン、テーマが用意したエラーテンプレートはこれまでどおり
  アプリケーションのレイアウトで表示され、`exception_stack_trace` エレメントも使えます。

**移行方法:**

- コアのエラーページを以前の見た目のままにするには、独自の `Layouts/dev_error.ctp` を
  (`templates/Layouts/`、`app/View/Layouts/`、またはテーマに) 用意します。このレイアウトには
  `title`、`subheading`、`file`、`templateName` のブロックと、ビュー変数 `$error` が渡されます。
  または、残したいエラーテンプレートを以前のリリースからアプリケーションの `Errors/` ディレクトリに
  コピーしてください。

  ```php
  <?php $this->extend('/Layouts/default'); ?>
  <h2><?php echo $this->fetch('title'); ?></h2>
  <p class="error"><?php echo $this->fetch('subheading'); ?></p>
  <?php echo $this->fetch('file'); ?>
  <?php echo $this->element('exception_stack_trace'); ?>
  ```

### `E_USER_ERROR` の代わりに例外を投げる ([PR #58](https://github.com/pieceofcake2/cakephp/pull/58)) {#user-error-exceptions}

PHP 8.4 で `E_USER_ERROR` の発生が非推奨になったため、次のエラーは同じメッセージの例外になりました。

- `Configure::bootstrap()` は、アプリケーションの `core.php` または `bootstrap.php` を読み込めない場合に
  `ConfigureException` を投げます。コンソールではメッセージを `Error: ...` として表示し、終了コード 1 で
  終了します。
- `TranslateBehavior` は、モデルのデータソースが接続されていない場合や、`bindTranslation()` に
  すでにモデルに関連付けられているアソシエーション名を渡した場合に `CakeException` を投げます。
  [Translate](/ja/core-libraries/behaviors/translate) を参照してください。
- `bin/cake` は、Composer のオートローダーが見つからない場合にメッセージを STDERR に出力し、
  終了コード 1 で終了します。

**移行方法:** これらの箇所では以前から処理が止まっていたため、ほとんどのアプリケーションでは変更は
不要です。独自のエラーハンドラーでこれらの `E_USER_ERROR` を捕捉して処理を続けていた場合は、
代わりに例外を捕捉してください。[例外](/ja/development/exceptions) を参照してください。

### ConsoleLog の色付けの判定 ([PR #58](https://github.com/pieceofcake2/cakephp/pull/58)) {#consolelog-colors}

- `ConsoleLog` は、STDIN ではなく出力先の `stream` が端末かどうかで、色付きで出力するかどうかを
  決めます。端末から実行してファイルやパイプにログを出力しても ANSI エスケープコードは書き込まれず、
  STDIN をリダイレクトしていても端末上の `php://stderr` への出力は色付きになります。
- `ConsoleOutput` は `stream_isatty()` を使うため、判定できないストリーム (`php://memory`、
  `php://temp`) では警告を出さずにプレーンな出力になります。

**移行方法:** 出力の形式を自分で選ぶには、ログの設定で `'outputAs' => ConsoleOutput::COLOR`
(または `PLAIN` / `RAW`) を指定してください。[ロギング](/ja/core-libraries/logging) を参照してください。

### 新しいバリデーションルール `Validation::utf8()` ([PR #64](https://github.com/pieceofcake2/cakephp/pull/64)) {#validation-utf8}

- CakePHP 5 のルールのバックポートです。値が正しい UTF-8 の文字列で、MySQL の `utf8` (`utf8mb3`) の
  カラムに保存できない基本多言語面より外の文字を含まないことを確認します。`['extended' => true]` を
  指定すると、そのような文字も許可します (`utf8mb4` 向け)。文字列でない値と不正なバイト列は、
  どちらのモードでも拒否します。

  ```php
  public $validate = [
      'title' => ['rule' => 'utf8'],
      'body' => ['rule' => ['utf8', ['extended' => true]]],
  ];
  ```

- 既存のルールは変わりません。モデルに `utf8()` という名前のメソッドがある場合は、そちらがコアのルールより
  優先されます。[データバリデーション](/ja/models/data-validation) を参照してください。

### Folder が認識するストリームラッパーのスキームを拡大 ([PR #63](https://github.com/pieceofcake2/cakephp/pull/63)) {#folder-stream-wrappers}

- `Folder::isAbsolute()` と `Folder::isRegisteredStreamWrapper()` は、`s3://` のように数字、`+`、`-`、`.` を
  含む登録済みのストリームラッパーのスキームを認識します。`Folder` と `File` は、このようなパスを
  相対パスとして扱わなくなりました。[Folder & File](/ja/core-utility-libraries/file-folder) を
  参照してください。

### アプリケーションの `core.php` がなくてもコンソールが動作 ([PR #59](https://github.com/pieceofcake2/cakephp/pull/59)) {#console-without-core-php}

- アプリケーションに `config/core.php` がない場合でも、コンソールが「Can't find application core file」
  という致命的エラーで止まらなくなりました。コアの代替の `core.php` を読み込むため、アプリケーションの
  設定を必要としないコマンドを実行できます。`config/core.php` があるアプリケーションには影響しません。
  [シェルとタスクとコンソール](/ja/console-and-shells) を参照してください。

### PHPStan レベル 5 に合わせた docblock ([PR #56](https://github.com/pieceofcake2/cakephp/pull/56)) {#phpstan-docblocks}

- コア (`TestSuite` を含む) は、`pieceofcake2/phpstan-cakephp2` ^0.2.4 を使った PHPStan のレベル 5 を
  ベースラインなしで通過します。修正したのは docblock だけです。ネイティブの引数・戻り値・プロパティの
  型は追加しておらず、public と protected のシグネチャも変わっていないため、アプリケーションや
  プラグインのサブクラスはそのまま動作します。
- 動的に作られていた一部のプロパティ (PHP 8.2 で非推奨) を宣言しました。たとえば `Scaffold::$redirect` や
  `ControllerTestDispatcher::$response` です。

**移行方法:** 不要です。アプリケーションで PHPStan を実行している場合は、修正した docblock によって
コードに新しいエラーが報告されたり、ベースラインの項目が不要になったりすることがあります。
[静的解析](/ja/development/static-analysis) を参照してください。

## v2.12.5

v2.12.5 の変更のうち、すでにフォークで動いているアプリケーションに影響する可能性があるものです。

<a id="security-fixes"></a>

### セキュリティ修正

このリリースでは 3 件の脆弱性を修正しました。ほとんどのアプリケーションは更新するだけで対応できます。
詳しくは [脆弱性への対応](/ja/security/fixes) を参照してください。

- **メールのヘッダー** ([GHSA-4f9q-3c4x-52cv](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-4f9q-3c4x-52cv)):
  `SmtpTransport` と `MailTransport` は、ヘッダーの名前と値から改行を取り除きます。
  ヘッダーの値に改行を入れてヘッダーを追加していた場合は、代わりに `CakeEmail::setHeaders()` / `addHeaders()` に
  ヘッダーを 1 つずつ渡してください。
- **ビュー・レイアウト・エレメントの名前** ([GHSA-w2mh-x92p-5p95](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-w2mh-x92p-5p95)):
  `..` を含む名前は、ビューのパス（`App::path('View')`、プラグインとテーマのビューのパス）の中のファイルを
  指していなければなりません。それ以外のファイルは存在しないものとして扱います。`element()` はエレメントが無い場合と
  同じ動作になり、`elementExists()` は `false` を返し、ビューとレイアウトは `MissingViewException` /
  `MissingLayoutException` を投げます。ほかの場所から読み込んでいるテンプレートは、ビューのパスに移動してください。
- **ログイン後のリダイレクト** ([GHSA-x4p9-p4cx-gxwx](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-x4p9-p4cx-gxwx)):
  `AuthComponent::redirectUrl()` は、セッションに保存されたプロトコル相対 URL をローカルのパスに変換します。
  コードの変更は不要です。

### 不正な ORDER BY のソート方向を非推奨に ([PR #54](https://github.com/pieceofcake2/cakephp/pull/54))

- `order`（find のオプション、`Model::$order`、アソシエーションの `'order'` 設定、`DboSource::order()`）に指定する
  ソート方向は、`ASC` または `DESC` で、その後に `NULLS FIRST` か `NULLS LAST` を続けることもできます。
  大文字と小文字は区別せず、前後や途中の空白が多くてもかまいません。`'asc'`、`'Desc'`、`' DESC '`、
  `'ASC NULLS FIRST'`、`'desc  nulls  last'` はいずれも有効です。`NULLS FIRST` / `NULLS LAST` はそのまま
  データベースに渡され、MySQL、MariaDB、SQL Server では使えません。[並び順](/ja/models/retrieving-your-data#sort-order) を参照してください。
- それ以外のソート方向を指定すると `E_USER_DEPRECATED` エラー（`Invalid ORDER BY direction "..."`）が発生します。
  対象は配列の値（`['Post.title' => $direction]`）、`DboSource::order()` の `$direction` 引数、文字列のキーで
  `ASC` / `DESC` の後に続くテキスト（`'Post.title DESC LIMIT 1'`。括弧の中やクォートしたリテラルの中の
  `ASC` / `DESC` は無視します）です。
- **このリリースでの変更はこれだけです**: 生成される SQL はこれまでとまったく同じです。
  将来のバージョンでは、不正なソート方向は `InvalidArgumentException` を投げるようになります。
- `PaginatorComponent` はリクエストパラメーターの `direction` をすでに `asc` / `desc` に制限しているため、
  ページネーションでは非推奨エラーは発生しません。
- 検証するのはソート方向だけです。order のキーのフィールドや式の部分は信頼できる SQL として扱い
  （可能な場合はクォートし、それ以外はそのまま使います）、ユーザーの入力を含めてはいけません。
  `DboSource::expression()` のオブジェクトはそのまま使い、検証しません。

[並び順](/ja/models/retrieving-your-data#sort-order) も参照してください。

**移行:**

- ソート方向の値に式を書いている場合: 式をキーに移動します。`['Page.name' => "='test' DESC"]` は
  `["Page.name = 'test'" => 'DESC']` に置き換えるか、生の SQL には `DboSource::expression()` を使ってください。
- 文字列のキーでソート方向の後に SQL を書いている場合（例: `'Post.title DESC LIMIT 1'`）:
  `order` にはフィールドとソート方向だけを指定し、残りは対応する find のオプション（`limit` など）か
  `DboSource::expression()` のオブジェクトで渡してください。
- ソート方向を付けずに `NULLS FIRST` / `NULLS LAST` を指定している場合: `ASC` か `DESC` を追加してください
  （`'ASC NULLS FIRST'`）。
- 文字列以外のソート方向（`true`、`1` など）を指定している場合: `'ASC'` か `'DESC'` を渡してください。
- リクエストデータからソート方向を受け取っている場合: `PaginatorComponent` と同じように、先に `asc` / `desc` に
  制限してください。例:
  `$direction = strtolower((string)$this->request->query('direction')) === 'desc' ? 'DESC' : 'ASC';`
- 呼び出し箇所を探すには、メッセージが `Invalid ORDER BY direction` で始まる `E_USER_DEPRECATED` エラーを
  ログに記録するか、そこで処理を止めてください。

## v2.12.3

v2.12.3 の変更のうち、すでにフォークで動いているアプリケーションに影響する可能性があるものです。

### `Xml::build()` の `readFile` オプションで https の URL も取得しない ([PR #33](https://github.com/pieceofcake2/cakephp/pull/33))

- **セキュリティ修正 (SSRF)**: `Xml::build($input, ['readFile' => false])` は、括弧が 1 組欠けていたため
  `https://` の URL を取得していました（本家の CakePHP 2.x から引き継いだ不具合です）。
  `'readFile' => false` を指定すると、`http://` と `https://` のどちらの URL も取得せず、`XmlException` を投げます。
  [脆弱性への対応](/ja/security/fixes#xml-build-ssrf) も参照してください。

**移行:**

- リモートからの読み込みを無効にするために `'readFile' => false` を指定している場合は、対応不要です。
  ドキュメントどおりに動作するようになりました。
- `'readFile' => false` を指定したまま `Xml::build()` に `https://` の URL を取得させていた場合は、
  オプションを削除する（既定値は `true`）か、`HttpSocket` などで自分で文書を取得して XML 文字列を渡してください。

### CookieComponent が `'type'` の設定を反映する ([PR #34](https://github.com/pieceofcake2/cakephp/pull/34))

- `CookieComponent` は `$components` で指定した `'type'`（例: `'Cookie' => ['type' => 'aes']`）を反映するようになりました。
  これまではこの設定が黙って無視され、Cookie は安全でないデフォルトの `'cipher'` で暗号化されていました。

**移行:**

- コントローラーの `$components` で `'type'` を設定している場合、Cookie はそのタイプで書き込まれるようになり、
  `'cipher'` で書き込まれた既存の Cookie は復号できなくなります。他の暗号化タイプの変更と同じように切り替えを計画してください。
- `'aes'` を使う場合、mcrypt 拡張がインストールされていなければ `Configure::write('Security.useOpenSsl', true);`
  を設定してください（[OpenSSL を有効にする](#enable-openssl) を参照）。
- 不正な `'type'` を設定すると、`CookieComponent::type()` と同じく `E_USER_WARNING` を発生させ、`'cipher'` を使います。

### テストケースの一覧が `tests/TestCase` 構成に対応 ([PR #35](https://github.com/pieceofcake2/cakephp/pull/35))

- `CakeTestLoader::generateTestList()` は、アプリケーションとプラグインの `tests/TestCase` からテストケースを探し、
  無ければレガシー構成の `Test/Case` を探すようになりました。これまではモダン構成で一覧が空になり、
  テストケース名を指定しない `bin/cake test app` / `bin/cake test <Plugin>` や Web のテストランナーに
  テストケースが表示されませんでした。

**移行:**

- 対応不要です。

### SQL Server ドライバーがデフォルトで `PDO::ATTR_ERRMODE` を設定 ([PR #36](https://github.com/pieceofcake2/cakephp/pull/36))

- `Sqlserver::connect()` は、他のドライバーと同じく `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION` を
  `'flags'` の設定にマージするようになりました。演算子の優先順位の不具合により、これまでこのデフォルトは適用されていませんでした。

**移行:**

- 対応不要です。PHP 8 の PDO はもともと `PDO::ERRMODE_EXCEPTION` がデフォルトで、
  `'flags'` で指定した `PDO::ATTR_ERRMODE` の値はこれまでどおり優先されます。

### ServerShell のデフォルトポートを 8765 に変更 ([PR #37](https://github.com/pieceofcake2/cakephp/pull/37))

- `bin/cake server` は、多くのシステムで root 権限が必要な `80` ではなく、デフォルトでポート `8765` で待ち受けるようになりました
  （CakePHP 3 以降と同じです）。
- `ServerShell::DEFAULT_PORT` は `8765` になりました。

**移行:**

- ポート 80 を使い続ける場合は、`bin/cake server -p 80` のように明示的に指定してください。

## 破壊的変更

### インストール後の作業

#### レガシー構成の場合（`app/` ディレクトリ）

インストール後、Bake プラグインをインストールし、ディスパッチャーファイルをコピーします。

```bash
# Bake プラグインをインストール
composer require --dev pieceofcake2/bake

# Web 用ディスパッチャーをコピー
cp app/Plugin/Bake/Console/Templates/skel/webroot/index.php app/webroot/index.php
cp app/Plugin/Bake/Console/Templates/skel/webroot/test.php app/webroot/test.php

# コンソール用ディスパッチャーをコピー
cp app/Plugin/Bake/Console/Templates/skel/Console/cake app/Console/cake
chmod +x app/Console/cake
```

#### モダン構成の場合（`src/` ディレクトリ）

次のファイルを更新する必要があります。

- `config/define.php`
- `bin/cake`
- `webroot/index.php`
- `webroot/test.php`

CakePHP 2.x と 5.x の両方に対応したモダンなアプリケーションスケルトンは
[`pieceofcake2/app`](https://pieceofcake2.github.io/app/ja/) を参照してください。
[インストール](/ja/installation) と
[ディレクトリ構成](/ja/appendices/migrating-to-the-modern-layout) も参照してください。

### ディレクトリ構成のモダン化（[PR #21](https://github.com/pieceofcake2/cakephp/pull/21)）

- **ディレクトリ構成を現在の標準的な形に再編しました**
- `lib/Cake/` → `src/Cake/`（フレームワークのソースコード）
- `lib/Cake/Test/Case/` → `tests/TestCase/`（フレームワークのテスト）
- `lib/Cake/Test/Fixture/` → `tests/Fixture/`（テストフィクスチャ）
- `lib/Cake/Console/cake` → `bin/cake`（コンソール実行ファイル）
- `lib/Cake/Config/` → `config/`（フレームワークの設定ファイルをルート直下へ）
- `lib/Cake/View/` と `lib/Cake/Templates/` → `templates/`（フレームワークのデフォルトテンプレートをルート直下へ）
- すべての設定ファイルとパスをそれに合わせて更新
- フレームワークのルートディレクトリを指す `CORE_ROOT` 定数を追加

**モダンなテストディレクトリのサポート:**

- アプリケーションとプラグインで、モダンな `tests/TestCase/` と `tests/Fixture/` の構成を使えるようになりました
- テストスイートは `Test/Case/`（従来）と `tests/TestCase/`（モダン）の両方を自動検出します
- フィクスチャローダーは `Test/Fixture/` と `tests/Fixture/` の両方に対応します
- アプリケーションのテストを段階的にモダン構成へ移行できます
- 破壊的変更はありません。両方の構成を同時に使えます

**移行:**

- フレームワークを依存パッケージとして使っているアプリケーションでは対応不要です
- `bin/cake`（Composer でインストールした場合は `vendor/bin/cake`）がプロジェクトの構成を自動で判別します
- モダンな `src/` 構成と従来の `app/` 構成のどちらでも自動的に動作します
- アプリケーションのテストは、必要に応じて `app/Test/Case/` から `app/tests/TestCase/` へ
  自分のペースで移行できます

**コンソール実行ファイルの改善:**

- `bin/cake` は Composer のオートローダーからプロジェクトのルートを自動検出します
- 手動の設定は不要で、どのプロジェクト構成でもそのまま動作します
- モダン（`src/`）と従来（`app/`）のアプリケーションディレクトリに対応します

### Composer オートロードへの移行（[PR #22](https://github.com/pieceofcake2/cakephp/pull/22)）

フレームワークは include_path によるクラス読み込みから Composer の classmap オートロードに
移行しました。クラスの読み込みが単純になり、現在の PHP ツールとの互換性も向上します。

**変更点:**

- CakePHP フレームワークのクラスから `App::uses()` 宣言を削除しました（798 箇所）
- CakePHP フレームワークのクラスは Composer の classmap オートローダーで自動的に読み込まれます
- アプリケーションのクラスも、Composer の classmap オートロードを設定すれば自動で読み込めます（下記の「移行」を参照）
- アプリケーションのクラスを継承するフレームワークのクラスでは、引き続き `App::uses()` を使います
  - `App::uses('AppController', 'Controller')` - CakeErrorController 用
  - `App::uses('AppModel', 'Model')` - I18nModel、AcoAction、Permission 用
  - `App::uses('AppShell', 'Console/Command')` - すべてのコアシェルコマンド用
  - `App::uses('AppHelper', 'View/Helper')` - すべてのコアヘルパー用

**移行:**

**新規プロジェクト、またはモダン構成（`src/`）の場合:**

- 推奨: `composer.json` に classmap オートロードの設定を追加します
  （[pieceofcake2/app](https://pieceofcake2.github.io/app/ja/) と同様）。

  ```json
  {
    "autoload": {
      "classmap": ["src/"]
    }
  }
  ```

**レガシーなプロジェクト（`app/` ディレクトリ）の場合:**

- 対応不要です。既存のアプリケーションは変更なしで動作し続けます
- アプリケーションのクラスに対するフレームワーク側の `App::uses()` 呼び出しで後方互換性を保っています

**プラグインのクラス:**

- Composer オートロードに対応していないプラグインでは、これまでどおり `App::uses()` で
  クラスを読み込む必要があります。

  ```php
  App::uses('MyPluginHelper', 'MyPlugin.View/Helper');
  ```

- プラグインのドキュメントにオートロード対応と書かれていれば、`App::uses()` は不要な場合があります

**独自オートローダーによる上書き（任意）:**

特定のクラスについてオートロードの挙動を上書きしたい場合は、独自のオートローダーを prepend
フラグ付きで登録し、Composer のオートローダーより優先させます。

```php
// 例: classmap ベースの独自オートローダー
$customClassMap = [
    'MyCustomClass' => '/path/to/MyCustomClass.php',
    'AnotherClass' => '/path/to/AnotherClass.php',
];

spl_autoload_register(function ($class) use ($customClassMap) {
    if (isset($customClassMap[$class])) {
        require $customClassMap[$class];

        return true;
    }

    return false;
}, true, true); // throw=true, prepend=true（Composer より優先）
```

**引数の意味:**

- 1 つ目の `true`: オートローダーの登録に失敗したら例外を投げる
- 2 つ目の `true`: オートローダーのキューの先頭に追加する（Composer のオートローダーより優先される）

**クラス名重複の警告:**

classmap に同じ名前のクラスが複数あると、`composer install` や `composer dump-autoload` の際に
Composer が警告を出します。

```text
Warning: Ambiguous class resolution, "AppController" was found in both "/path/to/vendor/pieceofcake2/app/src/Controller/AppController.php" and "/path/to/your-app/src/Controller/AppController.php", the first will be used.
```

この警告を防ぎ、正しいクラスが読み込まれるようにするには、`composer.json` で
`exclude-from-classmap` を使います。

```json
{
  "autoload": {
    "classmap": ["src/"],
    "exclude-from-classmap": [
      "vendor/pieceofcake2/app/src/Controller/AppController.php",
      "vendor/pieceofcake2/app/src/Model/AppModel.php"
    ]
  },
  "autoload-dev": {
    ...
  }
}
```

これにより Composer は classmap の生成時に指定したファイルを無視し、アプリケーションの
基底クラスとの衝突を防げます。

### Bake プラグインの分離（[PR #17](https://github.com/pieceofcake2/cakephp/pull/17)）

- **Bake の機能を別プラグインに分離しました**
  （[pieceofcake2/bake](https://pieceofcake2.github.io/bake/ja/)）
- `BakeShell` とすべての Bake タスクをコアから削除しました（`BakeTask`、`ModelTask`、
  `ControllerTask`、`ViewTask`、`FixtureTask`、`TestTask`、`TemplateTask`、`ProjectTask`、
  `PluginTask`、`DbConfigTask`、`CommandTask`）
- アプリケーションスケルトンのテンプレートを `lib/Cake/Console/Templates/skel/` から Bake プラグインへ移動しました
- ディスパッチャーファイル（`index.php`、`test.php`、`cake`）は Bake プラグインに含まれるようになりました
- Bake プラグインをインストールしないと、コンソールの bake コマンドは使えません

**移行:**

1. Bake プラグインを個別にインストールします: `composer require --dev pieceofcake2/bake`
2. `app/Config/bootstrap.php` でプラグインを読み込みます。

   ```php
   CakePlugin::load('Bake');
   ```

3. 必要に応じて Bake プラグインからディスパッチャーファイルをコピーします（新規プロジェクトの場合）

**変更の理由:**

- Bake の機能を独立して開発・バージョン管理できる
- コアフレームワークのサイズを減らせる
- ほとんどのアプリケーションは本番環境で Bake を必要としない

### 旧 Upgrade プラグインの分離（[PR #19](https://github.com/pieceofcake2/cakephp/pull/19)）

- **旧来のアップグレード機能を別プラグインに分離しました**
  （[pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/ja/)）
- `UpgradeShell` をコアから削除しました
- Upgrade プラグインをインストールしないと、旧来のアップグレードコマンド（1.3 → 2.0）は使えません

**移行:**

::: info
Upgrade プラグインが必要になるのは CakePHP 1.3 のアプリケーションを 2.0 に上げる場合だけで、
2025 年の時点ではきわめてまれです。ほとんどの利用者には不要です。
:::

CakePHP 1.3 のアプリケーションを 2.0 に上げる必要がある場合は、Upgrade プラグインを個別に
インストールします。

```bash
composer require --dev pieceofcake2/upgrade
```

そのうえで `app/Config/bootstrap.php` でプラグインを読み込みます。

```php
CakePlugin::load('Upgrade');
```

**変更の理由:**

- コアフレームワークのサイズを減らせる
- Upgrade プラグインは CakePHP 1.3 から 2.0 へのアップグレードにしか使わない
- 2025 年の時点でこの機能を必要とする人はほとんどいない

### Composer によるインストールが必須に（[PR #14](https://github.com/pieceofcake2/cakephp/pull/14)）

- **Composer を使わないインストールはサポートしなくなりました**
- ディスパッチャーファイル（`app/webroot/index.php`、`app/webroot/test.php`、`app/Console/cake`）は
  Composer のオートロードを必要とします
- ディスパッチャーファイルから `CAKE_CORE_INCLUDE_PATH` を手動で検出する処理を削除しました
- `include_path` を操作する処理を削除しました
- 旧来の `app/Vendor/cakephp/cakephp` パスの検出を削除しました
- `app/Console/cake.bat` と `app/Console/cake.php` を削除しました（Windows 用バッチファイルと
  PHP ラッパーは不要になりました）

**移行:**

1. 依存関係を Composer で管理していることを確認します
2. Bake プラグインをインストールします: `composer require --dev pieceofcake2/bake`
3. 更新されたディスパッチャーファイルを `app/Plugin/Bake/Console/Templates/skel/` から
   アプリケーションにコピーします。

   ```bash
   cp app/Plugin/Bake/Console/Templates/skel/webroot/index.php app/webroot/index.php
   cp app/Plugin/Bake/Console/Templates/skel/webroot/test.php app/webroot/test.php
   cp app/Plugin/Bake/Console/Templates/skel/Console/cake app/Console/cake
   ```

4. 古いディスパッチャーファイルがあれば削除します。

   ```bash
   rm -f app/Console/cake.bat app/Console/cake.php
   ```

5. `composer install` を実行し、すべての依存パッケージが正しく読み込まれるようにします

### キャッシュエンジンの削除（[PR #4](https://github.com/pieceofcake2/cakephp/pull/4)）

- **Xcache** のサポートを削除しました（PHP 7.0 以降に非対応）
- **Wincache** のサポートを削除しました（PHP 8.x 向けに保守されていない）

**移行:**

- これらのキャッシュエンジンを使っている場合は、Redis、Memcached、APCu のいずれかに移行してください

### データベースドライバーへのメソッド追加（[PR #3](https://github.com/pieceofcake2/cakephp/pull/3)）

- データベースドライバーに新しいメソッドを追加しました（独自のドライバー実装がある場合は問題になる可能性があります）

**MySQL ドライバー（`Mysql.php`）:**

- `getVersion(): string` - MySQL / MariaDB / Aurora MySQL のバージョンを返す
- `getServerType(): string` - 'MySQL'、'MariaDB'、'Aurora MySQL' のいずれかを返す
- `utf8mb4Supported(): bool` - utf8mb4 文字セットに対応しているかを調べる
- `integerDisplayWidthDeprecated(): bool` - 整数型の表示幅が非推奨か（MySQL 8.0.17 以降）を調べる

**PostgreSQL ドライバー（`Postgres.php`）:**

- `getVersion(): string` - PostgreSQL のバージョンを返す

**移行:**

- これらのクラスを継承した独自のデータベースドライバーがある場合は、これらのメソッドを実装してください

### データベースの文字セット設定の変更（[PR #11](https://github.com/pieceofcake2/cakephp/pull/11)）

- 文字セットの設定を `SET NAMES` から DSN の接続オプションに移しました
- **MySQL**: 文字セットを DSN に含めます（例: `mysql:...;charset=utf8`）
- **PostgreSQL**: クライアントエンコーディングを DSN の options で指定します
  （例: `pgsql:...;options='--client_encoding=UTF8'`）
- **PostgreSQL**: DSN の `sslmode` パラメーターは省略可能になりました

**移行:**

- 対応不要です。変更は後方互換です
- 実行時に変更する場合は、引き続き `setEncoding()` メソッドを使えます
- 文字セットを DSN で指定するため、接続処理が効率的になります

### SQL Server ドライバーの更新（[PR #9](https://github.com/pieceofcake2/cakephp/pull/9)）

#### 設定の形式

- **スキーマ**: `schema` の設定で接続に使う SQL Server のスキーマを指定します。既定値は `dbo` です。
  1 つのデータベースの複数のスキーマにあるテーブルを使う場合は、スキーマごとに接続を定義します。

  ```php
  public $default = [
      'datasource' => 'Database/Sqlserver',
      'host' => 'localhost',
      'port' => 1433,
      'login' => 'user',
      'password' => 'password',
      'database' => 'my_app',
      'schema' => 'dbo',
  ];
  ```

- **接続オプション**: SSL/TLS のオプションは `options` 配列で指定します

  ```php
  'options' => [
      'TrustServerCertificate' => 'yes',
      'Encrypt' => 'no',
  ]
  ```

- **ポートの設定**: ポートは個別に指定します（サーバー名に自動的に付加されます）

#### メソッドシグネチャの変更

- `describe($model): array` - 戻り値の型を明示しました
- `insertMulti()` - `void` ではなく `bool` を返すようになりました

**移行:**

- テーブルが `dbo` 以外のスキーマにある場合は `schema` を設定します
- DSN に直接 SSL/TLS オプションを書いている場合は `options` 配列に移します
- Sqlserver クラスを継承している場合は、メソッドシグネチャを合わせてください

### メール関数の更新（[PR #10](https://github.com/pieceofcake2/cakephp/pull/10)）

- `MailTransport::_mail()` のメソッドシグネチャが厳密な型付きに変わりました
- 変更前: `protected function _mail($to, $subject, $message, $headers, $params = null)`
- 変更後: `protected function _mail(string $to, string $subject, string $message, array|string $headers = [], string $params = ''): void`

**移行:**

- `MailTransport` クラスを継承していなければ対応不要です
- 継承している場合は、厳密な型に合わせてメソッドシグネチャを更新してください

### CSRF トークンのセキュリティ強化（[PR #5](https://github.com/pieceofcake2/cakephp/pull/5)）

- 新しい CSRF トークンは HMAC-SHA1 の署名を使います（CVE-2020-15400 への対策）
- トークンの形式は base64 エンコード（16 バイトの値 + 20 バイトの HMAC）に変わりました

**移行:**

- **対応不要です**。自動的に適用され、後方互換です
- 既存のトークンも引き続き有効です
- 新しいトークンは強化された方式で生成されます

詳しくは [セキュリティ](/ja/security/fixes) を参照してください。

### strftime() の置き換え

- `strftime()` は PHP 8.1 で非推奨になり、PHP 8.2 で削除されました
- 代わりに Symfony の ICU Polyfill を介して `IntlDateFormatter` を使います
- 互換性のために `PHP81_BC\strftime` にフォールバックします

**移行:**

- ほとんどの日付フォーマットは同じ結果になります
- 境界的なケースでは出力がわずかに異なる場合があります
- アプリケーションの日付フォーマットをテストしてください

### 開発ツールの更新

#### PHP CodeSniffer（[PR #8](https://github.com/pieceofcake2/cakephp/pull/8)）

- 1.0.0 から 5.3 に更新しました
- 自動フォーマットによる修正を適用しました

**移行:**

- 開発時のみの変更です
- 独自のコーディング規約を使っている場合は `phpcs.xml` を更新してください

[コーディング規約](/ja/contributing/cakephp-coding-conventions) を参照してください。

#### PHPUnit の互換性

- フレームワークのテストを PHPUnit 9.6 に移行しました
- 非推奨の PHPUnit 機能はすべて修正しました

**移行:**

- 非推奨の PHPUnit 機能を使っている場合は、テストを更新してください

### PHP 8 構文へのモダン化（[PR #7](https://github.com/pieceofcake2/cakephp/pull/7)）

- コードベースを PHP 8 の構文に書き換えました

**変更点:**

- `array()` → `[]`
- `get_class()` → `::class`
- 配列の分割代入で `list()` → `[]`
- `dirname(__FILE__)` → `__DIR__`
- null 合体演算子を追加
- ネイティブの `str_contains()`、`str_starts_with()`、`str_ends_with()` を使用

**移行:**

- **対応不要です**。構文の変更のみで、機能の変更はありません

### CookieComponent の 'cipher' タイプは安全ではないため置き換えが必要

- `CookieComponent` のデフォルトの暗号化タイプ `'cipher'` はきわめて安全性が低い方式です

::: warning
CakePHP のソースコードのコメントにも次のように書かれています。

> "Cipher is horribly insecure and only the default because of backwards
> compatibility. In new applications you should always change this to 'aes' or
> 'rijndael'."

（cipher はきわめて安全性が低く、後方互換性のためだけにデフォルトになっている。新しいアプリケーションでは必ず 'aes' か 'rijndael' に変更すること。）
:::

**'cipher' が安全でない理由:**

- XOR による暗号化を行う `Security::cipher()` を使っている（暗号学的に弱い）
- `Security.cipherSeed` を float から int へ変換する際の挙動が未定義
- シード値 `mt_srand((int)(float)Configure::read('Security.cipherSeed'))` の一貫性が保証されない
- 機密データの保護には適さない

**移行:**

```php
// 旧（安全でない。使わないこと）
public $components = [
    'Cookie' => [
        'type' => 'cipher'  // デフォルト。きわめて安全性が低い
    ]
];

// 新（推奨）
public $components = [
    'Cookie' => [
        'type' => 'aes',
    ],
];
```

`beforeFilter()` で `$this->Cookie->type('aes');` を呼び出して設定することもできます。

::: pieceofcake2
`$components` の `'type'` は v2.12.3 から反映されます。v2.12.2 以前では無視され、
Cookie は `'cipher'` で暗号化されていました。

v2.12.6 以降では `'cipher'` は非推奨で警告をログに記録し、`config/core.php` の `Cookie.type` と
`Cookie.key` でデフォルトのタイプとキーを設定できます。[v2.12.6](#cipher-deprecated) を参照してください。
:::

PHP 8 では `'aes'` を使う場合も `Security.useOpenSsl` の設定が必要です。下記の
[OpenSSL を有効にする](#enable-openssl) を参照してください。

::: danger
暗号化タイプを変更すると、既存のクッキーは無効になります。移行期間中は両方のタイプに対応するなど、
移行の方針を計画してください。
:::

## PHP 8 移行時の補足

以下は `UPGRADE.md` には書かれていませんが、PHP 8 へ移行するアプリケーションでよく問題になる点です。

### `Security::encrypt()` と AES クッキーで OpenSSL を有効にする {#enable-openssl}

`mcrypt` 拡張は PHP 7.2 以降同梱されておらず、通常はインストールされていません（[PECL](https://pecl.php.net/package/mcrypt) から入れることはできます）。
`Security::encrypt()` と `Security::decrypt()` は `Security.useOpenSsl` が有効な場合にだけ
OpenSSL を使い、そうでなければ `mcrypt_*` 関数を呼び出して失敗します。これらのメソッドを使う
`CookieComponent` の `'aes'` タイプも同様です。フレームワークのデフォルトではこの設定は有効になっていません。
[pieceofcake2/app](https://pieceofcake2.github.io/app/ja/) のスケルトン (v2.3.1 以降) から作成したアプリケーションでは
`config/core.php` で有効になっています。既存のアプリケーションでは、`app/Config/core.php`（レガシー構成）または
`config/core.php`（モダン構成）に追加してください。

```php
Configure::write('Security.useOpenSsl', true);
```

::: warning
クッキーの `'rijndael'` タイプと `Security::rijndael()` は常に mcrypt を必要とします。
mcrypt の無い PHP 8 環境では、`Security.useOpenSsl` を有効にしたうえで `'aes'` を使ってください。
:::

### `String` クラスの削除

非推奨だった `String` ユーティリティクラスは削除されました（`string` は PHP 7 から予約語です）。
代わりに `CakeText` を使ってください。

```php
// 変更前
App::uses('String', 'Utility');
$uuid = String::uuid();

// 変更後
$uuid = CakeText::uuid();
```
