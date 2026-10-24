# Migrating from CakePHP 2.10

This guide describes how to move an application from the original CakePHP
2.10.24 to pieceofcake2/cakephp, and lists the breaking changes. It is based on
[UPGRADE.md](https://github.com/pieceofcake2/cakephp/blob/2.x/UPGRADE.md).

## Prerequisites

Before migrating to this fork, ensure:

- Your application is running on PHP 7.4
- You're using CakePHP 2.10.24 (earlier versions are not supported)
- Your application uses Composer for dependency management

## From Original CakePHP 2.x

1. **Update to CakePHP 2.10.24 first**: If you're using an earlier version,
   update to `cakephp/cakephp:2.10.24` on PHP 7.4 first
2. **Ensure PHP 7.4 Compatibility**: Your application must be fully working on
   PHP 7.4 before migrating to PHP 8.x
3. **Update Composer**: Replace `cakephp/cakephp` with `pieceofcake2/cakephp` in
   your `composer.json`, and add the `composer/installers` settings so that
   plugins are installed into `app/Plugin/` (`plugins/` in the modern layout).
   These settings are required; see
   [Configuring composer.json](/installation#configuring-composer-json)
4. **Upgrade PHP**: Update your PHP version to 8.0 or newer
5. **Test Thoroughly**: Run your application's test suite to ensure compatibility

## Unreleased

Changes in the next release that may affect applications already running on the fork.

### Hints of the debug error pages ([PR #116](https://github.com/pieceofcake2/cakephp/pull/116)) {#error-page-hints}

- In an application with `templates/`, the Missing View and Missing Layout
  pages and the customization hint suggest the file that the View finds first,
  e.g. `templates/Pages/home.php`, `templates/layout/default.php` and
  `templates/Error/error400.php`, instead of `.ctp` files under `src/View/` and
  `templates/Errors/`.
- With `App.namespace`, the pages for a missing controller, action, component,
  helper or behavior suggest the class in the namespace of the application or
  the plugin, with the `use` of its parent class, and a plugin that is not
  loaded in `plugins/<Plugin>/src/`.
- Applications in the legacy layout keep their hints.

### Core templates in `.php` and the CakePHP 5 directories ([PR #117](https://github.com/pieceofcake2/cakephp/pull/117)) {#core-templates-php}

- The templates of the framework are `.php` files in the CakePHP 5
  directories: `templates/Error/`, `templates/layout/` (with
  `layout/email/`), `templates/element/` and `templates/email/` instead of
  `.ctp` files in `Errors/`, `Layouts/` (`Layouts/Emails/`), `Elements/` and
  `Emails/`, e.g. `templates/layout/dev_error.php`. `Pages/`, `Scaffolds/` and
  the subdirectories (`element/Flash/`, `layout/js/`, ...) keep their names.
- The templates of your application, plugins and themes still override them,
  in either directory and with either extension.
- The View looks up templates, layouts and elements path by path also when a
  controller or view sets another `$ext` than `.ctp`: in each view path it
  tries `$ext` and then `.ctp`. Before, it tried `$ext` in all the view paths
  first. With `$ext = '.php'`, a `.ctp` template of the application still
  overrides a core template such as `Error/error400.php`, and a `.ctp`
  template of a theme or plugin now comes before a `.php` template of the
  application. Nothing changes with the default `.ctp`.
- The messages for a missing template, layout or element name the file that
  the View looks for first, e.g. `Pages/home.php`, `layout/default.php` or
  `element/foo.php` in `templates/`, and `.ctp` names in `View/`.

**Migration:**

- Code that builds the path of a core template, e.g.
  `CORE_ROOT . DS . 'templates' . DS . 'Layouts' . DS . 'default.ctp'`, must use
  the new path.

### Layouts of the core error pages ([PR #117](https://github.com/pieceofcake2/cakephp/pull/117)) {#error-page-layouts}

- The core error templates switch layouts as the error templates of CakePHP 5
  do. `error400` and `error500` set the `error` layout first and use the
  `dev_error` layout in debug mode. The other core error pages, which are only
  rendered in debug mode, always use `dev_error`.
- In debug mode, AJAX requests also get the debug error page, instead of the
  error inside the layout of the request.
- In production mode, the core `error400` and `error500` templates, which are
  used when the application has none, are rendered in the `error` layout
  instead of the default layout. Without an `error` layout in the application,
  this is the `error` layout of the framework.

**Migration:**

- The error templates of applications created from pieceofcake2/app v2.5.3 or
  earlier, or from pieceofcake2/app-legacy, use the `dev_error` layout only
  when the framework has `templates/Layouts/dev_error.ctp`:

  ```php
  $devErrorLayout = is_file(dirname(CAKE_CORE_INCLUDE_PATH) . DS . 'templates' . DS . 'Layouts' . DS . 'dev_error.ctp');
  ```

  This file is now `templates/layout/dev_error.php`, so in debug mode these
  templates show errors in the default layout of the application. Remove the
  check from `error400.ctp` and `error500.ctp` (in `templates/Errors/`, or
  `app/View/Errors/` in the legacy layout), as the framework has the layout
  since v2.12.6, and switch the layouts as the core templates do:

  ```php
  $this->layout = 'error';

  if (Configure::read('debug')) :
      $this->layout = 'dev_error';

      $this->assign('title', $message);
      $this->assign('templateName', 'error400.php');
      // ...
  endif;
  ```

  The core templates assign `templateName` with `.php`, as in CakePHP 5.
  `error400.ctp` also works: the customization hint names the file in the
  layout of your application either way (`templates/Error/error400.php`, or
  `app/View/Errors/error400.ctp` in the legacy layout).

- To keep the default layout for the error pages of the framework in
  production mode, add an `error` layout to the application (e.g. a copy of the
  default layout), or your own `error400` and `error500` templates.

### `App::uses()` in namespaced applications and plugins ([PR #119](https://github.com/pieceofcake2/cakephp/pull/119)) {#app-uses-namespaces}

- In an application with `App.namespace` and in a namespaced plugin,
  `App::load()` loads a class declared with `App::uses()` from the namespace
  with Composer, when the CakePHP 2 directories have no global class of that
  name: `App::uses('Comment', 'Blog.Model')` loads `Blog\Model\Comment`. This
  also works in the packages that `App::path()` does not list, such as
  `Blog.Utility` or `Blog.Lib/Custom`, and when two plugins have a class of the
  same name. Before, `App::load()` returned `false` for those packages, and for
  the second plugin it included the file of the first plugin instead.
- No global name is declared for these classes, as before. `AppController`,
  `AppModel`, `AppHelper` and `AppShell` keep their global names. Refer to the
  classes by their names in the configuration (`'Post'`, `'Blog.Comment'`),
  which [`App::className()`](/core-utility-libraries/app) resolves.
- Applications without `App.namespace` and their global plugins load their
  classes as before.

### Assets of the browser test runner ([PR #122](https://github.com/pieceofcake2/cakephp/pull/122)) {#test-suite-assets}

- The browser test runner (`webroot/test.php`) loads `cake.generic.css` and
  the `test-*-icon.png` icons from `webroot/test_suite/` when they are there,
  as in the application skeletons, and from `webroot/css/` and `webroot/img/`
  otherwise. Applications that keep the files in `webroot/css/` and
  `webroot/img/` do not need to change anything.

## v2.13.0

Changes in v2.13.0 that may affect applications already running on the fork.

CakePHP 2.13 adds features; follow these steps to update from 2.12:

1. Update the constraint in `composer.json` to `"pieceofcake2/cakephp": "^2.13"`
   and run `composer update pieceofcake2/cakephp`.
2. If you use pieceofcake2/phpstan-cakephp2, update it to `^1.1`, which knows
   the classes of the `Cake` namespace:
   `composer require --dev pieceofcake2/phpstan-cakephp2:^1.1`. With version
   1.0, PHPStan reports errors in code that worked before.
3. Read the items below, especially the class names in logs and output and the
   comparisons of class names as strings, and run your tests.

### Variadic arguments ([PR #89](https://github.com/pieceofcake2/cakephp/pull/89)) {#variadic-arguments}

- The methods that read a variable number of arguments with `func_get_args()`
  now declare them as a variadic parameter. The calls do not change:
  `$this->Auth->allow('index', 'view')` and `$this->Post->query($sql, $params)`
  work as before.
- A class that **overrides** one of these methods must declare the same
  variadic parameter. PHP refuses an override with fewer parameters and stops
  when the class is loaded:
  `Declaration of AppModel::query($sql) must be compatible with Model::query($sql, ...$params)`.

  | Method | New signature |
  | --- | --- |
  | `Shell::dispatchShell()` | `dispatchShell(...$args)` |
  | `AuthComponent::allow()`, `deny()` | `allow(...$args)`, `deny(...$args)` |
  | `SecurityComponent::requirePost()`, `requireGet()`, `requirePut()`, `requireDelete()`, `requireSecure()`, `requireAuth()` | `requirePost(...$args)`, ... |
  | `Controller::setAction()` | `setAction($action, ...$args)` |
  | `Controller::validate()`, `validateErrors()` | `validate(...$args)`, `validateErrors(...$objects)` |
  | `ContainableBehavior::contain()` | `contain(Model $Model, ...$args)` |
  | `DboSource::query()` | `query(...$args)` |
  | `Model::isUnique()` | `isUnique($fields, $or = true, ...$args)` |
  | `Model::query()` | `query($sql, ...$params)` |
  | `CakeRequest::data()`, `param()` | `data($name, ...$args)`, `param($name, ...$args)` |
  | `CakeRequest::input()` | `input($callback = null, ...$args)` |
  | `CakeRequest::allowMethod()`, `onlyAllow()` | `allowMethod($methods, ...$args)`, `onlyAllow($methods, ...$args)` |
  | `Router::parseExtensions()` | `parseExtensions(...$extensions)` |
  | `CakeTestCase::loadFixtures()` | `loadFixtures(...$args)` |
  | `Hash::merge()` | `merge(array $data, $merge, ...$args)` |
  | `Sanitize::stripTags()` | `stripTags($str, ...$tags)` |
  | `Set::merge()` | `merge($data, $merge = null, ...$args)` |
  | `HtmlHelper::css()` | `css($path, $options = [], ...$args)` |
  | `HtmlHelper::useTag()` | `useTag($tag, ...$args)` |

  ```php
  // Before
  public function query($sql)
  {
      return parent::query(...func_get_args());
  }

  // After
  public function query($sql, ...$params)
  {
      return parent::query($sql, ...$params);
  }
  ```

- The `rector` command of pieceofcake2/upgrade adds the parameters for you.
  The changed code also runs on the versions before, so run it before you
  update:

  ```bash
  bin/cake upgrade rector --rules variadic-arguments src tests
  ```

  See [Rector rules](/console-and-shells/upgrade-shell#rector-rules).
- `CakeRequest::input(null)` now returns the raw input, as `input()` does.
  It called `null` before.

### Plugins in the CakePHP 5 layout ([PR #88](https://github.com/pieceofcake2/cakephp/pull/88)) {#plugin-modern-layout}

- A plugin can put namespaced classes in `src/` (loaded by Composer with the
  plugin's `psr-4` entry), its templates in `templates/`, its translations in
  `resources/locales/`, its vendor files in `vendor/` and its bootstrap, routes,
  configuration files and schemas in `config/`, as the application does.
- CakePHP searches these directories first, and `Controller/`, `View/`,
  `Locale/`, `Vendor/`, `Config/` and the other CakePHP 2 directories after
  them. Nothing changes for a plugin that does not have them. See
  [How To Create Plugins](/plugins/how-to-create-plugins).

> [!WARNING]
> In a plugin that already has one of these directories for another purpose,
> its files are now found first. For example, `App::import('Vendor', ...)`
> looks first in the Composer `vendor/` directory of a plugin checked out for
> development.

### Templates in the CakePHP 5 style ([PR #94](https://github.com/pieceofcake2/cakephp/pull/94)) {#php-templates}

- In `templates/`, templates, layouts and elements may use the `.php`
  extension and the CakePHP 5 directories `layout/`, `element/`, `email/` and
  `Error/`, e.g. `templates/email/html/welcome.php`.
- Nothing changes for `.ctp` templates and for the `View/` directories of the
  legacy layout. See [Templates in the CakePHP 5 style](/views#templates-in-the-cakephp-5-style).

> [!WARNING]
> A `.php` file next to a `.ctp` file of the same name in `templates/` is now
> used instead of it. Check that `templates/` holds no other `.php` files.

### PHP 8.1 to 8.5 deprecations ([PR #96](https://github.com/pieceofcake2/cakephp/pull/96)) {#php-deprecations}

The core no longer triggers the deprecations of PHP 8.1 to 8.5 in its own
code. Most applications only need to update. These changes may affect yours:

- `Xml::build()` no longer has the `loadEntities` option. It only decided
  whether `libxml_disable_entity_loader()` was called, which PHP 8 deprecates
  and which no longer changes anything with libxml 2.9 and later: external
  entities are not loaded either way. Passing the option is ignored, so you
  can remove it from your calls. See [Xml](/core-utility-libraries/xml).
- `CakeRequest::input()` with a decoding callback returns `null` without
  calling the callback when the request has no body. `RequestHandlerComponent`
  sets the same empty data as before. If your own callback handled an empty
  body, handle `null` instead. See
  [Request and Response objects](/controllers/request-response).
- The core requires `symfony/polyfill-php84` for the `Pdo\Mysql` constants on
  PHP 8.0 to 8.3. Composer installs it.

### Classes in a namespace: `App::className()` and `App.namespace` ([PR #91](https://github.com/pieceofcake2/cakephp/pull/91)) {#app-classname}

- Settings with a suffix take fully qualified class names with the suffix:
  `MemoryEngine::class` for a cache engine, `SesTransport::class` for a
  transport, and the same for Auth objects, password hashers, view classes,
  components, helpers, behaviors, shell tasks and shell helpers. The names
  without the suffix (`'Acme\Cache\Engine\Memory'`) keep working.
- `App.namespace` is new and not set by default. When it is set, controllers,
  shells, models and the classes above are first looked up in that namespace
  by their short name, e.g. `App\Controller\PostsController` for `/posts`.
  See [Namespaced Classes](/development/namespaced-classes).
- Fully qualified class names as the keys of `$components`, `$helpers`,
  `$actsAs`, shell `$tasks`, `$uses` and the model associations, or given to
  `load()`, `loadModel()` and `ClassRegistry::init()` without an alias, get the
  class name without the namespace and the suffix as their alias:
  `GreetingComponent::class` gives `$this->Greeting`, and `Post::class` gives
  `$this->Post`. Before, the alias was the name with the backslashes; code that
  used such an alias must use the short alias. A parent class and a subclass
  that name the same object, one by its class name and one by its short name,
  now give one entry with the merged settings. `'with' => PostsTag::class` of a
  hasAndBelongsToMany association finds the join model `PostsTag`, and
  `ClassRegistry::getObject()`, `isKeySet()` and `removeObject()` also find
  objects by their fully qualified class names. See
  [Namespaced Classes](/development/namespaced-classes#components-helpers-and-behaviors).
- `Controller` and `View` have the new public property `$controllerClass`,
  which is also passed from the controller to the view. Rename a property of
  that name in a subclass.

> [!WARNING]
> `Controller::$name`, `Model::$name` and `Shell::$name` of a class in a
> namespace no longer contain the namespace, e.g. `Post` instead of
> `Acme\Model\Post`. Set `$name` if you relied on the old value.

> [!WARNING]
> A namespaced controller now merges `$components` and `$helpers` with its
> parent class `AppController` in any namespace, also without `App.namespace`.

### The framework classes in the `Cake` namespace ([PR #92](https://github.com/pieceofcake2/cakephp/pull/92)) {#cake-namespace}

- The classes of the framework moved into the `Cake` namespace, e.g.
  `Cake\Controller\Controller`. The global names of CakePHP 2 are aliases of the
  same classes: code that extends, type-hints, catches or loads `Controller`,
  `CakeRequest` or `NotFoundException` keeps working.
- The files of the framework moved from `src/Cake/` to `src/`, loaded by
  Composer with PSR-4 (`Cake\` in `src/`), and `CAKE` points to `src/`.
  `src/Cake/bootstrap.php`, which `webroot/index.php` requires, and
  `src/Cake/TestSuite/Fixture/CakeFixtureInjector.php`, the listener of
  `phpunit.xml.dist`, stay at their paths and keep working. If you include other
  files of `vendor/pieceofcake2/cakephp/src/Cake/` by path, change them to `src/`.
- Objects serialized before the update still unserialize. Objects serialized
  after it name the namespaced classes and cannot be read by an older version.
- A global class registered with `App::uses()` no longer overrides a core
  class, because the framework uses the namespaced names. Copy the core file to
  `src/Cake/` instead. See
  [Overriding classes in CakePHP](/core-utility-libraries/app#overriding-classes-in-cakephp).
- PHPStan needs pieceofcake2/phpstan-cakephp2 `^1.1`: run
  `composer require --dev pieceofcake2/phpstan-cakephp2:^1.1`. With version 1.0,
  PHPStan no longer finds the type of components, models, tasks and behaviors.
  See [Static Analysis](/development/static-analysis#installation).

> [!WARNING]
> `get_class()`, `::class` and `ReflectionClass::getName()` return the
> namespaced names, so a comparison like `get_class($x) === 'CakeRequest'` is
> now false. The error log, `Debugger` output and the debug error pages show
> the namespaced names too, e.g. `[Cake\Error\NotFoundException]`: update the
> log filters and alerts that match the global names.

> [!TIP]
> `bin/cake upgrade rector --rules namespaced-classes src tests` of
> [pieceofcake2/upgrade](https://github.com/pieceofcake2/upgrade) rewrites the
> comparisons with `get_class()` and imports the namespaced classes.

### `App::uses()` is deprecated ([PR #98](https://github.com/pieceofcake2/cakephp/pull/98)) {#app-uses-deprecated}

- Composer loads the classes of the core, of the application (pieceofcake2/app
  loads `src/` with a classmap) and of the plugins installed with Composer.
  The classes named in the configuration (`$components`, `$helpers`, `$uses`,
  the associations, the engines and the other settings) are found without
  `App::uses()`, as before.
- The first call of `App::uses()` in a process logs a warning with the file
  and line of the call. With `debug` above 0, it also triggers
  `E_USER_DEPRECATED`. The call still works.
- The core no longer calls `App::uses()`. `AppController`, `AppModel`,
  `AppHelper` and `AppShell` of the application are found in their packages
  without it.
- To remove the calls, let Composer load your classes, then delete the
  `App::uses()` lines: in an application, add `"classmap": ["src/"]` to the
  `autoload` of its `composer.json` and run `composer dump-autoload`; in a
  plugin, add a classmap or PSR-4 to the `composer.json` of the plugin. See
  [Removing App::uses() calls](/core-utility-libraries/app#removing-app-uses).

## v2.12.11

### Security fix: request URLs with extra leading slashes ([GHSA-wh43-gc93-wh4c](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-wh43-gc93-wh4c), [PR #90](https://github.com/pieceofcake2/cakephp/pull/90)) {#request-url-leading-slashes}

- `CakeRequest` removes all the leading slashes and backslashes of the request
  URL, not only the first one. A request to `//posts/view/1` is routed as
  before, but `$this->request->url` is now `posts/view/1` instead of
  `/posts/view/1`, and `here` is `/posts/view/1` instead of the
  protocol-relative `//posts/view/1`. `here()` also collapses the leading
  slashes of its result. URLs with a single leading slash are not affected. See [Security Fixes](/security/fixes#request-here).

### HTML5 script, link and style tags ([PR #86](https://github.com/pieceofcake2/cakephp/pull/86)) {#html5-tags}

- HtmlHelper outputs the tags of HTML5: `<script src="...">`,
  `<link rel="stylesheet" href="...">`, `<style>`, and script blocks without
  `type="text/javascript"` and without CDATA. Browsers treat both forms the
  same, so the pages do not change.
- Tests that compare the HTML of a page or of a helper may need the new tags.
- To keep the former output, e.g. for pages served as XHTML, set `html5` to
  `false` in the settings of the helper:

  ```php
  public $helpers = ['Html' => ['html5' => false]];
  ```

  See [HTML5 Tags](/core-libraries/helpers/html#html5).

## v2.12.10

Changes in v2.12.10 that may affect applications already running on the fork.

### New function `raw()` ([PR #84](https://github.com/pieceofcake2/cakephp/pull/84)) {#raw}

- `raw()`, the opposite of `h()`, returns a value as it is, to print HTML
  without escaping on purpose. Use it where a view prints HTML that the
  application built or sanitized, instead of a bare `echo`:

  ```php
  <?= raw($post['Post']['body_html']) ?>
  ```

- An application or a package that already defines a global `raw()` keeps its
  own function, as for the other global functions. Check that yours does the
  same, or rename it. See [raw()](/core-libraries/global-constants-and-functions#raw).

## v2.12.7

Changes in v2.12.7 that may affect applications already running on the fork.

### DSN configuration: `DATABASE_URL` and `MAILER_DSN` ([PR #69](https://github.com/pieceofcake2/cakephp/pull/69)) {#dsn-configuration}

- A datasource in `DATABASE_CONFIG` and a profile in `EmailConfig` can set the
  whole connection with a DSN in a new `'dsn'` key, read from the environment
  variables Symfony uses:

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

  `DATABASE_URL` is a DSN such as `mysql://user:secret@localhost:3306/app`, and
  `MAILER_DSN` uses the Symfony Mailer format, such as
  `smtp://user:secret@smtp.example.com:587` or `null://null`.
- The keys parsed from the DSN replace the same keys of the array, and the
  other keys stay. An empty `'dsn'` is ignored, so configurations without it
  work exactly as before.
- Plugins can add DSN schemes, or replace built-in ones, with
  `ConnectionManager::setDsnClassMap()` and `CakeEmail::setDsnClassMap()`.
  pieceofcake2/aws and pieceofcake2/async-aws register the Amazon SES schemes
  (`ses+api://`, `ses+smtp://`, ...).
- The application skeleton (pieceofcake2/app) reads `DATABASE_URL` and
  `MAILER_DSN`, and loads `config/.env.test` in the tests. Since app v2.5.0 it
  loads the `.env` files with symfony/dotenv as Symfony does (see
  [Loading .env Files](/development/configuration#loading-env-files)).

See [Database DSNs](/development/configuration#database-url),
[Mailer DSN](/core-utility-libraries/email#mailer-dsn) and
[Registering DSN Schemes](/plugins/how-to-create-plugins#registering-dsn-schemes).

## v2.12.6

Changes in v2.12.6 that may affect applications already running on the fork.

### Security fix: 'cipher' encryption is deprecated ([PR #62](https://github.com/pieceofcake2/cakephp/pull/62)) {#cipher-deprecated}

`Security::cipher()` and the `'cipher'` type of `CookieComponent` (its default
type) are not secure with any `Security.cipherSeed`
([GHSA-65v5-h65v-8pgg](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-65v5-h65v-8pgg)).
They are now deprecated. Values are encrypted exactly as before, and nothing
throws, but:

- The first `'cipher'` use in a process (one request or one console run)
  writes a `warning` to the log, also with debug off. With debug on, the same
  message is also triggered as `E_USER_DEPRECATED`. The `'aes'` and
  `'rijndael'` types and unencrypted cookies log nothing.
- A `Security.cipherSeed` that is not a digits-only number from 1 to
  4294967295 is reported: the first `Security::cipher()` call logs a
  `warning` (the seed is never logged), and with debug on
  `Debugger::checkSecurityKeys()` (called by `AuthComponent`) triggers an
  `E_USER_NOTICE`. The notice asking you to change the default seed now fires
  for both the former default `'76859309657453542496749683645'` and the new
  default `'2659758734'`.
- The fallback `config/core.php` of the core (used only when the application
  has no `config/core.php`) sets `Security.cipherSeed` to `'2659758734'` and
  `Security.useOpenSsl` to `true`.

**Migration:**

- Do not change `Security.cipherSeed` just to silence the warning. A new seed
  makes existing `'cipher'` cookies unreadable, and `'cipher'` stays insecure.
  Switch to `'aes'` as described in
  [Migrating to 'aes'](/development/configuration#migrating-to-aes).
- If your test suite turns deprecations or notices into exceptions, the first
  `'cipher'` use in a test run, or `AuthComponent` with debug on and an
  out-of-range seed, now fails. Pass `'aes'` as the mode of
  `cookieEncrypted()` / `assertCookieEncrypted()` once the application uses
  `'aes'`, or set an in-range seed in the test configuration.
- Replace your own calls to `Security::cipher()` with `Security::encrypt()` /
  `Security::decrypt()` and a key of at least 32 bytes.

See [Security Fixes](/security/fixes#cipher) and
[cipherSeed and the 'cipher' Cookie Type](/development/configuration#cipher-seed).

### CookieComponent reads `Cookie.type` and `Cookie.key` ([PR #62](https://github.com/pieceofcake2/cakephp/pull/62)) {#cookie-type-key}

- Without a `'key'` setting, `CookieComponent` uses Configure `Cookie.key`
  when it is a non-empty string, otherwise `Security.salt` as before.
- Without a `'type'` setting, it uses Configure `Cookie.type` when it is set,
  otherwise `'cipher'` as before. An invalid value triggers the usual
  `E_USER_WARNING` and falls back to `'cipher'`.
- An explicit `'key'` / `'type'` setting, `$this->Cookie->key = ...` or
  `$this->Cookie->type(...)` always wins.
- `cookieEncrypted()` / `assertCookieEncrypted()` of `IntegrationTestTrait`
  use the same default key.

Nothing changes for applications that set neither value. To switch the whole
application to `'aes'`, set these in `config/core.php`:

```php
Configure::write('Security.useOpenSsl', true);
Configure::write('Cookie.type', 'aes');
Configure::write('Cookie.key', env('COOKIE_KEY') ?: null);
```

`'aes'` needs a key of at least 32 bytes. Setting `Cookie.key` while still on
`'cipher'` changes the key, so existing `'cipher'` cookies become unreadable.
See [Cookie](/core-libraries/components/cookie).

### Debug error pages in the CakePHP 5 style ([PR #60](https://github.com/pieceofcake2/cakephp/pull/60)) {#dev-error-pages}

- In debug mode, the core error pages use a new `dev_error` layout ported
  from CakePHP 5, with code excerpts for every stack frame and `(edit)` links
  that open the file in your editor (PhpStorm by default; set
  `Debugger.editor` and, in a container, `Debugger.editorBasePath`). See
  [Debug Error Pages](/development/debugging#debug-error-pages).
- Production mode (`debug` = 0), AJAX requests and the console are unchanged,
  and no existing PHP API changed. Error templates provided by your
  application, plugin or theme keep rendering in your layout, and the
  `exception_stack_trace` element still works.

**Migration:**

- To keep the old look for the core error pages, provide your own
  `Layouts/dev_error.ctp` (in `templates/Layouts/`, `app/View/Layouts/` or a
  theme), which receives the blocks `title`, `subheading`, `file` and
  `templateName` and the `$error` view variable, or copy the error templates
  you want to keep from the previous release into your `Errors/` directory:

  ```php
  <?php $this->extend('/Layouts/default'); ?>
  <h2><?php echo $this->fetch('title'); ?></h2>
  <p class="error"><?php echo $this->fetch('subheading'); ?></p>
  <?php echo $this->fetch('file'); ?>
  <?php echo $this->element('exception_stack_trace'); ?>
  ```

### Exceptions instead of `E_USER_ERROR` ([PR #58](https://github.com/pieceofcake2/cakephp/pull/58)) {#user-error-exceptions}

Triggering `E_USER_ERROR` is deprecated as of PHP 8.4, so these errors are now
exceptions with the same message:

- `Configure::bootstrap()` throws `ConfigureException` when the application
  `core.php` or `bootstrap.php` cannot be included. The console prints the
  message as `Error: ...` and exits with code 1.
- `TranslateBehavior` throws `CakeException` when the model's datasource is
  not connected, or when `bindTranslation()` is given an association name that
  is already bound to the model. See [Translate](/core-libraries/behaviors/translate).
- `bin/cake` prints a missing Composer autoloader to STDERR and exits with
  code 1.

**Migration:** execution already stopped at these points, so most applications
need no change. If a custom error handler caught these `E_USER_ERROR` errors
and continued, catch the exceptions instead. See [Exceptions](/development/exceptions).

### ConsoleLog color detection ([PR #58](https://github.com/pieceofcake2/cakephp/pull/58)) {#consolelog-colors}

- `ConsoleLog` decides between colored and plain output by checking whether
  its `stream` is a terminal, not STDIN. Logging to a file or pipe from a
  terminal no longer writes ANSI escape codes, and logging to `php://stderr`
  on a terminal is colored even when STDIN is redirected.
- `ConsoleOutput` uses `stream_isatty()`, so streams that cannot be checked
  (`php://memory`, `php://temp`) get plain output without a warning.

**Migration:** set `'outputAs' => ConsoleOutput::COLOR` (or `PLAIN` / `RAW`)
in the log configuration to choose the format yourself. See
[Logging](/core-libraries/logging).

### New validation rule `Validation::utf8()` ([PR #64](https://github.com/pieceofcake2/cakephp/pull/64)) {#validation-utf8}

- A backport of the CakePHP 5 rule. It checks that a value is a valid UTF-8
  string without characters above the Basic Multilingual Plane, which MySQL's
  `utf8` (`utf8mb3`) columns cannot store. With `['extended' => true]`, such
  characters are allowed (for `utf8mb4`). Non-string values and invalid byte
  sequences are rejected in both modes:

  ```php
  public $validate = [
      'title' => ['rule' => 'utf8'],
      'body' => ['rule' => ['utf8', ['extended' => true]]],
  ];
  ```

- No existing rule changes. A model method named `utf8()` keeps precedence
  over the core rule. See [Data Validation](/models/data-validation).

### Folder recognizes more stream wrapper schemes ([PR #63](https://github.com/pieceofcake2/cakephp/pull/63)) {#folder-stream-wrappers}

- `Folder::isAbsolute()` and `Folder::isRegisteredStreamWrapper()` recognize
  registered stream wrapper schemes that contain digits, `+`, `-` or `.`, such
  as `s3://`. `Folder` and `File` no longer treat such paths as relative. See
  [Folder & File](/core-utility-libraries/file-folder).

### Console without an application `core.php` ([PR #59](https://github.com/pieceofcake2/cakephp/pull/59)) {#console-without-core-php}

- When the application has no `config/core.php`, the console no longer stops
  with the fatal error "Can't find application core file". It loads the
  fallback `core.php` of the core, so commands that do not need the
  application settings can run. Applications with a `config/core.php` are not
  affected. See [Shells, Tasks & Console Tools](/console-and-shells).

### PHPStan level 5 docblocks ([PR #56](https://github.com/pieceofcake2/cakephp/pull/56)) {#phpstan-docblocks}

- The core (including `TestSuite`) passes PHPStan level 5 with
  `pieceofcake2/phpstan-cakephp2` ^0.2.4, without a baseline. Only docblocks
  were corrected; no native parameter, return or property types were added,
  and no public or protected signature changed, so subclasses in applications
  and plugins keep working.
- Some properties that were created dynamically (deprecated as of PHP 8.2) are
  now declared, for example `Scaffold::$redirect` and
  `ControllerTestDispatcher::$response`.

**Migration:** none needed. If you run PHPStan on your application, the
corrected docblocks may report new errors in your code, or make baseline
entries obsolete. See [Static Analysis](/development/static-analysis).

## v2.12.5

Changes in v2.12.5 that may affect applications already running on the fork.

### Security fixes

This release fixes three vulnerabilities. Most applications only need to
update. See [Security Fixes](/security/fixes) for details.

- **Email headers** ([GHSA-4f9q-3c4x-52cv](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-4f9q-3c4x-52cv)):
  `SmtpTransport` and `MailTransport` remove line breaks from header names and
  values. If you relied on putting raw line breaks in a header value to add
  headers, use `CakeEmail::setHeaders()` / `addHeaders()` with one entry per
  header instead.
- **View, layout and element names** ([GHSA-w2mh-x92p-5p95](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-w2mh-x92p-5p95)):
  a name containing `..` must resolve to a file inside the view paths
  (`App::path('View')`, plugin and theme view paths). Files outside them are
  treated as missing: `element()` behaves as for a missing element,
  `elementExists()` returns `false`, and views and layouts throw
  `MissingViewException` / `MissingLayoutException`. Move templates you load
  from elsewhere into a view path.
- **Login redirect** ([GHSA-x4p9-p4cx-gxwx](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-x4p9-p4cx-gxwx)):
  `AuthComponent::redirectUrl()` turns a protocol-relative URL stored in the
  session into a local path. No change is needed.

### Invalid ORDER BY directions are deprecated ([PR #54](https://github.com/pieceofcake2/cakephp/pull/54))

- The sort direction given to `order` (find options, `Model::$order`,
  association `'order'` settings, `DboSource::order()`) should be `ASC` or
  `DESC`, optionally followed by `NULLS FIRST` or `NULLS LAST`. It is matched
  case-insensitively and surrounding / repeated whitespace is allowed, so
  `'asc'`, `'Desc'`, `' DESC '`, `'ASC NULLS FIRST'` and `'desc  nulls  last'`
  are all valid. `NULLS FIRST` / `NULLS LAST` are passed to the database as
  written and are not supported by MySQL, MariaDB and SQL Server; see
  [Sort Order](/models/retrieving-your-data#sort-order).
- Any other direction now triggers an `E_USER_DEPRECATED` error
  (`Invalid ORDER BY direction "..."`). This covers array values
  (`['Post.title' => $direction]`), the `$direction` argument of
  `DboSource::order()`, and text after an `ASC` / `DESC` in a string key
  (`'Post.title DESC LIMIT 1'`; `ASC` / `DESC` inside parentheses or quoted
  literals is ignored).
- **Nothing else changes in this release**: the generated SQL is exactly the
  same as before. Invalid directions will throw an `InvalidArgumentException`
  in a future version.
- `PaginatorComponent` already restricts the `direction` request parameter to
  `asc` / `desc`, so pagination does not trigger the deprecation.
- Only the direction is checked. The field / expression part of an order key
  is trusted SQL (quoted where possible, otherwise used as is) and must never
  contain user input. `DboSource::expression()` objects are used verbatim and
  are not checked.

See also [Sort Order](/models/retrieving-your-data#sort-order).

**Migration:**

- An expression in the direction value: move the expression into the key.
  Replace `['Page.name' => "='test' DESC"]` with
  `["Page.name = 'test'" => 'DESC']`, or use `DboSource::expression()` for raw
  SQL.
- SQL after the direction in a string key (for example
  `'Post.title DESC LIMIT 1'`): put only the field and the direction in
  `order`, and pass the rest with the matching find option (`limit`, ...) or
  as a `DboSource::expression()` object.
- `NULLS FIRST` / `NULLS LAST` without a direction: add `ASC` or `DESC`
  (`'ASC NULLS FIRST'`).
- A direction that is not a string (`true`, `1`, ...): pass `'ASC'` or
  `'DESC'`.
- A direction taken from request data: restrict it to `asc` / `desc` first,
  as `PaginatorComponent` does, e.g.
  `$direction = strtolower((string)$this->request->query('direction')) === 'desc' ? 'DESC' : 'ASC';`.
- To find the call sites, log or break on `E_USER_DEPRECATED` errors whose
  message starts with `Invalid ORDER BY direction`.

## v2.12.3

Changes in v2.12.3 that may affect applications already running on the fork.

### `Xml::build()` `readFile` option also blocks https URLs ([PR #33](https://github.com/pieceofcake2/cakephp/pull/33))

- **Security fix (SSRF)**: `Xml::build($input, ['readFile' => false])` fetched
  `https://` URLs because of a missing pair of parentheses (inherited from the
  original CakePHP 2.x). With `'readFile' => false`, neither `http://` nor
  `https://` URLs are fetched now; an `XmlException` is thrown instead. See
  [Security Fixes](/security/fixes#xml-build-ssrf).

**Migration:**

- No action required if you pass `'readFile' => false` to disable remote
  loading; this now works as documented.
- If you relied on `Xml::build()` fetching an `https://` URL with
  `'readFile' => false`, either remove the option (default is `true`) or fetch
  the document yourself (e.g. with `HttpSocket`) and pass the XML string.

### CookieComponent honors the `'type'` setting ([PR #34](https://github.com/pieceofcake2/cakephp/pull/34))

- `CookieComponent` now applies the `'type'` setting given in `$components`
  (e.g. `'Cookie' => ['type' => 'aes']`). Previously the setting was silently
  ignored and cookies were encrypted with the insecure default `'cipher'` type.

**Migration:**

- If your controllers set `'type'` in `$components`, cookies will now be
  written with that type, and cookies already written with `'cipher'` can no
  longer be decrypted. Plan the switch like any other encryption type change.
- When using `'aes'`, set `Configure::write('Security.useOpenSsl', true);`
  unless the mcrypt extension is installed (see [Enable OpenSSL](#enable-openssl)).
- An invalid `'type'` setting now triggers an `E_USER_WARNING` and falls back
  to `'cipher'`, as `CookieComponent::type()` does.

### Test case listing supports the `tests/TestCase` layout ([PR #35](https://github.com/pieceofcake2/cakephp/pull/35))

- `CakeTestLoader::generateTestList()` now finds test cases in `tests/TestCase`
  for the app and plugins, falling back to the legacy `Test/Case` layout.
  Previously the list was empty for the modern layout, so `bin/cake test app` /
  `bin/cake test <Plugin>` without a test case name and the web test runner
  showed no test cases.

**Migration:**

- No action required.

### SQL Server driver sets `PDO::ATTR_ERRMODE` by default ([PR #36](https://github.com/pieceofcake2/cakephp/pull/36))

- `Sqlserver::connect()` now merges `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`
  into the `'flags'` setting as the other drivers do. Because of an operator
  precedence bug, the default was never applied.

**Migration:**

- No action required. PDO already uses `PDO::ERRMODE_EXCEPTION` by default on
  PHP 8, and a `PDO::ATTR_ERRMODE` value given in `'flags'` still takes
  precedence.

### ServerShell default port changed to 8765 ([PR #37](https://github.com/pieceofcake2/cakephp/pull/37))

- `bin/cake server` now listens on port `8765` by default (as in CakePHP 3 and
  later) instead of `80`, which requires root privileges on most systems.
- `ServerShell::DEFAULT_PORT` is now `8765`.

**Migration:**

- To keep using port 80, pass it explicitly: `bin/cake server -p 80`.

## Breaking Changes

### Post-Installation Steps

#### For Legacy Directory Structure (`app/` directory)

After installation, install the Bake plugin and copy dispatcher files:

```bash
# Install Bake plugin
composer require --dev pieceofcake2/bake

# Copy web dispatcher files
cp app/Plugin/Bake/Console/Templates/skel/webroot/index.php app/webroot/index.php
cp app/Plugin/Bake/Console/Templates/skel/webroot/test.php app/webroot/test.php

# Copy console dispatcher
cp app/Plugin/Bake/Console/Templates/skel/Console/cake app/Console/cake
chmod +x app/Console/cake
```

#### For Modern Directory Structure (`src/` directory)

The following files need to be updated:

- `config/define.php`
- `bin/cake`
- `webroot/index.php`
- `webroot/test.php`

See [`pieceofcake2/app`](/installation/application-skeleton) for the modern
application skeleton compatible with both CakePHP 2.x and 5.x.
See also [Installation](/installation) and
[Directory Structure](/appendices/migrating-to-the-modern-layout).

### Directory Structure Modernization ([PR #21](https://github.com/pieceofcake2/cakephp/pull/21))

- **Directory layout has been restructured to modern standards**
- `lib/Cake/` → `src/Cake/` (framework source code)
- `lib/Cake/Test/Case/` → `tests/TestCase/` (framework tests)
- `lib/Cake/Test/Fixture/` → `tests/Fixture/` (test fixtures)
- `lib/Cake/Console/cake` → `bin/cake` (console executable)
- `lib/Cake/Config/` → `config/` (framework configuration files at root level)
- `lib/Cake/View/` and `lib/Cake/Templates/` → `templates/` (framework default
  templates at root level)
- All configuration files and paths updated accordingly
- Added `CORE_ROOT` constant to reference framework root directory

**Modern Test Directory Support:**

- Applications and plugins can now use modern `tests/TestCase/` and
  `tests/Fixture/` directory structure
- Test suite automatically detects both `Test/Case/` (traditional) and
  `tests/TestCase/` (modern) directories
- Fixture loader supports both `Test/Fixture/` and `tests/Fixture/` locations
- Enables gradual migration to modern directory structure for application tests
- No breaking changes - both directory structures work simultaneously

**Migration:**

- No action required for applications using this framework as a dependency
- `bin/cake` (or `vendor/bin/cake` when installed via Composer) now
  intelligently detects your project structure
- Automatically works with both modern `src/` and traditional `app/` directory
  layouts
- Optionally migrate your application tests from `app/Test/Case/` to
  `app/tests/TestCase/` at your own pace

**Enhanced Console Executable:**

- `bin/cake` now automatically detects project root from Composer's autoloader
- No manual configuration needed - works out of the box with any project
  structure
- Supports modern (`src/`) and traditional (`app/`) application directories

### Composer Autoloading Migration ([PR #22](https://github.com/pieceofcake2/cakephp/pull/22))

The framework has migrated from include-path to Composer classmap autoloading.
This change simplifies class loading and improves compatibility with modern PHP
tooling.

**What changed:**

- Removed `App::uses()` declarations from CakePHP framework classes (798
  occurrences removed)
- CakePHP framework classes are now automatically loaded via Composer's
  classmap autoloader
- Application classes can be automatically loaded by configuring Composer
  classmap autoloading (see Migration section below)
- CakePHP framework classes that extend application classes use `App::uses()`:
  - `App::uses('AppController', 'Controller')` - for CakeErrorController
  - `App::uses('AppModel', 'Model')` - for I18nModel, AcoAction, Permission
  - `App::uses('AppShell', 'Console/Command')` - for all core shell commands
  - `App::uses('AppHelper', 'View/Helper')` - for all core helpers

**Migration:**

**For new projects or modern directory structure (`src/`):**

- Recommended: Update your `composer.json` to include classmap autoloading
  (similar to [pieceofcake2/app-legacy](/installation/application-skeleton-legacy)):

  ```json
  {
    "autoload": {
      "classmap": ["src/"]
    }
  }
  ```

**For legacy projects (`app/` directory):**

- No action required - existing applications continue to work without changes
- The framework's `App::uses()` calls for application classes ensure backward
  compatibility

**Plugin classes:**

- Plugins that don't support Composer autoloading require `App::uses()` to load
  their classes (same as before):

  ```php
  App::uses('MyPluginHelper', 'MyPlugin.View/Helper');
  ```

- If a plugin's documentation indicates autoloading support, `App::uses()` may
  not be needed

**Custom Autoloader Override (optional):**

If you need to override autoloading behavior for specific classes, register
your custom autoloader with the prepend flag to give it priority over
Composer's autoloader:

```php
// Example: Custom classmap-based autoloader
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
}, true, true); // throw=true, prepend=true (priority over Composer)
```

**Parameters explained:**

- First `true`: Throw exception if autoloader registration fails
- Second `true`: Prepend to autoloader queue (gives priority over Composer's
  autoloader)

**Duplicate Class Name Warnings:**

When Composer encounters duplicate class names in the classmap, it will issue a
warning during `composer install` or `composer dump-autoload`:

```text
Warning: Ambiguous class resolution, "AppController" was found in both "/path/to/vendor/pieceofcake2/app/src/Controller/AppController.php" and "/path/to/your-app/src/Controller/AppController.php", the first will be used.
```

To prevent these warnings and ensure the correct class is loaded, use
`exclude-from-classmap` in your `composer.json`:

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

This tells Composer to ignore specific files when building the classmap,
preventing conflicts with your application's base classes.

### Bake Plugin Extraction ([PR #17](https://github.com/pieceofcake2/cakephp/pull/17))

- **Bake functionality has been extracted to a separate plugin**
  ([pieceofcake2/bake](https://pieceofcake2.github.io/bake/))
- `BakeShell` and all Bake tasks removed from core (`BakeTask`, `ModelTask`,
  `ControllerTask`, `ViewTask`, `FixtureTask`, `TestTask`, `TemplateTask`,
  `ProjectTask`, `PluginTask`, `DbConfigTask`, `CommandTask`)
- Application skeleton templates moved from `lib/Cake/Console/Templates/skel/`
  to Bake plugin
- Dispatcher files (`index.php`, `test.php`, `cake`) now located in Bake plugin
- Console bake commands no longer available without installing the Bake plugin

**Migration:**

1. Install the Bake plugin separately: `composer require --dev pieceofcake2/bake`
2. Load the plugin in your `app/Config/bootstrap.php`:

   ```php
   CakePlugin::load('Bake');
   ```

3. Copy dispatcher files from Bake plugin if needed (for new projects)

**Why this change:**

- Allows independent development and versioning of Bake functionality
- Reduces core framework size
- Most production applications don't need Bake in production

### Legacy Upgrade Plugin Extraction ([PR #19](https://github.com/pieceofcake2/cakephp/pull/19))

- **Legacy Upgrade functionality has been extracted to a separate plugin**
  ([pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/))
- `UpgradeShell` removed from core
- Legacy upgrade commands (1.3 → 2.0) no longer available without installing
  the Upgrade plugin

**Migration:**

::: info
The Upgrade plugin is only relevant for upgrading legacy CakePHP 1.3
applications to 2.0, which is extremely rare in 2025. Most users will never
need this plugin.
:::

If you need to upgrade a CakePHP 1.3 application to 2.0, install the Upgrade
plugin separately:

```bash
composer require --dev pieceofcake2/upgrade
```

Then load the plugin in your `app/Config/bootstrap.php`:

```php
CakePlugin::load('Upgrade');
```

**Why this change:**

- Reduces core framework size
- The Upgrade plugin is only useful for upgrading from CakePHP 1.3 to 2.0
- Almost no one needs this functionality in 2025

### Composer-Only Installation Required ([PR #14](https://github.com/pieceofcake2/cakephp/pull/14))

- **Non-Composer installation is no longer supported**
- Dispatcher files (`app/webroot/index.php`, `app/webroot/test.php`,
  `app/Console/cake`) require Composer autoload
- Removed manual `CAKE_CORE_INCLUDE_PATH` detection from dispatcher files
- Removed `include_path` manipulation logic
- Removed legacy `app/Vendor/cakephp/cakephp` path detection
- Removed `app/Console/cake.bat` and `app/Console/cake.php` (Windows batch file
  and PHP wrapper no longer needed)

**Migration:**

1. Ensure you're using Composer for dependency management
2. Install the Bake plugin: `composer require --dev pieceofcake2/bake`
3. Copy updated dispatcher files from `app/Plugin/Bake/Console/Templates/skel/` to
   your application:

   ```bash
   cp app/Plugin/Bake/Console/Templates/skel/webroot/index.php app/webroot/index.php
   cp app/Plugin/Bake/Console/Templates/skel/webroot/test.php app/webroot/test.php
   cp app/Plugin/Bake/Console/Templates/skel/Console/cake app/Console/cake
   ```

4. Remove old dispatcher files if present:

   ```bash
   rm -f app/Console/cake.bat app/Console/cake.php
   ```

5. Run `composer install` to ensure all dependencies are properly loaded

### Cache Engines Removed ([PR #4](https://github.com/pieceofcake2/cakephp/pull/4))

- **Xcache** support has been removed (not compatible with PHP 7.0+)
- **Wincache** support has been removed (not actively maintained for PHP 8.x)

**Migration:**

- If using these cache engines, migrate to Redis, Memcached, or APCu

### Database Driver Methods Added ([PR #3](https://github.com/pieceofcake2/cakephp/pull/3))

- New methods added to database drivers (may cause issues if you have custom
  driver implementations)

**MySQL Driver (`Mysql.php`):**

- `getVersion(): string` - Returns MySQL/MariaDB/Aurora MySQL version
- `getServerType(): string` - Returns 'MySQL', 'MariaDB', or 'Aurora MySQL'
- `utf8mb4Supported(): bool` - Checks utf8mb4 character set support
- `integerDisplayWidthDeprecated(): bool` - Checks if integer display width is
  deprecated (MySQL 8.0.17+)

**PostgreSQL Driver (`Postgres.php`):**

- `getVersion(): string` - Returns PostgreSQL version

**Migration:**

- If you have custom database drivers extending these classes, implement these
  methods

### Database Charset Configuration Changes ([PR #11](https://github.com/pieceofcake2/cakephp/pull/11))

- Character set configuration moved from `SET NAMES` to DSN connection options
- **MySQL**: Charset now in DSN (e.g., `mysql:...;charset=utf8`)
- **PostgreSQL**: Client encoding in DSN options (e.g.,
  `pgsql:...;options='--client_encoding=UTF8'`)
- **PostgreSQL**: `sslmode` parameter is now optional in DSN

**Migration:**

- No action required - changes are backward compatible
- `setEncoding()` methods still work for runtime changes
- More efficient connection setup with charset in DSN

### SQL Server Driver Updates ([PR #9](https://github.com/pieceofcake2/cakephp/pull/9))

#### Configuration Format

- **Schema**: the `schema` setting selects the SQL Server schema of a
  connection and defaults to `dbo`. To use tables in several schemas of one
  database, define one connection per schema:

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

- **Connection options**: SSL/TLS options now in `options` array

  ```php
  'options' => [
      'TrustServerCertificate' => 'yes',
      'Encrypt' => 'no',
  ]
  ```

- **Port configuration**: Specify port separately (automatically appended to
  server)

#### Method Signature Changes

- `describe($model): array` - Now has explicit return type
- `insertMulti()` - Now returns `bool` instead of `void`

**Migration:**

- Set `schema` if your tables are not in the `dbo` schema
- Move SSL/TLS options to `options` array if using inline DSN
- If extending Sqlserver class, update method signatures to match

### Mail Function Updates ([PR #10](https://github.com/pieceofcake2/cakephp/pull/10))

- `MailTransport::_mail()` method signature changed with strict types
- Old: `protected function _mail($to, $subject, $message, $headers, $params = null)`
- New: `protected function _mail(string $to, string $subject, string $message, array|string $headers = [], string $params = ''): void`

**Migration:**

- No action required unless you've extended `MailTransport` class
- If extending, update method signature to match strict types

### CSRF Token Security Enhancement ([PR #5](https://github.com/pieceofcake2/cakephp/pull/5))

- New CSRF tokens use HMAC-SHA1 signatures (prevents CVE-2020-15400)
- Token format changed to base64-encoded (16-byte value + 20-byte HMAC)

**Migration:**

- **No action required** - automatic and backward compatible
- Existing tokens continue to work
- New tokens generated with enhanced security

See [Security](/security/fixes) for details.

### strftime() Replacement

- `strftime()` deprecated in PHP 8.1, removed in PHP 8.2
- Now uses `IntlDateFormatter` via Symfony's ICU Polyfill
- Fallback to `PHP81_BC\strftime` for compatibility

**Migration:**

- Most date formatting works identically
- Edge cases may produce slightly different output
- Test date formatting in your application

### Development Tools Updates

#### PHP CodeSniffer ([PR #8](https://github.com/pieceofcake2/cakephp/pull/8))

- Updated from 1.0.0 to 5.3
- Applied automatic formatting fixes

**Migration:**

- Development-time change only
- Update `phpcs.xml` if you have custom coding standards

See [Coding Standards](/contributing/cakephp-coding-conventions).

#### PHPUnit Compatibility

- Framework tests migrated to PHPUnit 9.6
- All deprecated PHPUnit features fixed

**Migration:**

- Update your tests if using deprecated PHPUnit features

### PHP 8 Syntax Modernization ([PR #7](https://github.com/pieceofcake2/cakephp/pull/7))

- Codebase modernized to PHP 8 syntax

**Changes:**

- `array()` → `[]`
- `get_class()` → `::class`
- `list()` → `[]` for array destructuring
- `dirname(__FILE__)` → `__DIR__`
- Added null coalescing operators
- Native `str_contains()`, `str_starts_with()`, `str_ends_with()`

**Migration:**

- **No action required** - syntax changes only, no functionality changes

### CookieComponent 'cipher' Type - Insecure and Should Be Replaced

- The default `CookieComponent` encryption type `'cipher'` is horribly insecure

::: warning
As stated in the CakePHP source code comments:

> "Cipher is horribly insecure and only the default because of backwards
> compatibility. In new applications you should always change this to 'aes' or
> 'rijndael'."
:::

**Why 'cipher' is insecure:**

- Uses `Security::cipher()` with XOR encryption (cryptographically weak)
- Uses `Security.cipherSeed` with undefined float-to-int casting behavior
- The seed value: `mt_srand((int)(float)Configure::read('Security.cipherSeed'))`
  has no guaranteed consistency
- Not suitable for protecting sensitive data

**Migration:**

```php
// OLD (insecure - DO NOT USE)
public $components = [
    'Cookie' => [
        'type' => 'cipher'  // Default, horribly insecure
    ]
];

// NEW (recommended)
public $components = [
    'Cookie' => [
        'type' => 'aes',
    ],
];
```

You can also call `$this->Cookie->type('aes');` in `beforeFilter()`.

::: pieceofcake2
The `'type'` setting in `$components` is honored since v2.12.3. In v2.12.2 and
earlier it was ignored and cookies were encrypted with `'cipher'`.

Since v2.12.6, `'cipher'` is deprecated and logs a warning, and `Cookie.type`
and `Cookie.key` in `config/core.php` set the default type and key; see
[v2.12.6](#cipher-deprecated).
:::

On PHP 8, `'aes'` also needs `Security.useOpenSsl`; see
[Enable OpenSSL](#enable-openssl) below.

::: danger
Changing encryption type will invalidate existing cookies. Plan your migration
strategy accordingly (e.g., support both types during transition period).
:::

## Additional notes for PHP 8

These points are not listed in `UPGRADE.md` but commonly affect applications
moving to PHP 8.

### Enable OpenSSL for `Security::encrypt()` and AES cookies {#enable-openssl}

The `mcrypt` extension is not bundled with PHP 7.2 and later, and is usually
not installed (it is available from [PECL](https://pecl.php.net/package/mcrypt)).
`Security::encrypt()` and `Security::decrypt()` use OpenSSL only when
`Security.useOpenSsl` is enabled; otherwise they call `mcrypt_*` functions and
fail. The same applies to `CookieComponent` with the `'aes'` type, which uses
these methods. The framework does not enable the setting by default.
Applications created from the [pieceofcake2/app](/installation/application-skeleton)
skeleton (v2.3.1 and later) already have it in `config/core.php`. In existing
applications, add it to `app/Config/core.php` (legacy) or `config/core.php`
(modern):

```php
Configure::write('Security.useOpenSsl', true);
```

::: warning
The `'rijndael'` cookie type and `Security::rijndael()` always require mcrypt.
On PHP 8 without mcrypt, use `'aes'` with `Security.useOpenSsl` enabled.
:::

### `String` class removed

The deprecated `String` utility class has been removed (`string` is a reserved
word since PHP 7). Use `CakeText` instead:

```php
// Before
App::uses('String', 'Utility');
$uuid = String::uuid();

// After
$uuid = CakeText::uuid();
```
