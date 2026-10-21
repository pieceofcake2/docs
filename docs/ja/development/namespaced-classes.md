# 名前空間のあるクラス

CakePHP 2 のクラスは名前空間の無いグローバルなクラスですが、アプリケーションのコードでは
名前空間を使えます。アプリケーションスケルトンのクラスマップと並べて `composer.json` に
PSR-4 の設定を書き、`composer dump-autoload` を実行します。

```json
"autoload": {
    "classmap": ["src/"],
    "psr-4": {
        "Acme\\": "lib/"
    }
}
```

これで、`Acme\Error\Renderer` のようなクラスを、最初に使うときに Composer が
`lib/Error/Renderer.php` から読み込みます。`App::uses()` は要りません。

CakePHP の設定にクラス名を書く場所には、名前空間のあるクラスを `Foo::class` で書けます。
このページでは、pieceofcake2/cakephp の設定項目と、それぞれで CakePHP がクラスを探す方法を
まとめます。また、`App.namespace` を設定して、コントローラーやモデルなどのクラスを
アプリケーションの名前空間から短い名前で探す方法も説明します。

::: pieceofcake2 Added in version 2.13.0
`App::className()` と `App.namespace` ([#91](https://github.com/pieceofcake2/cakephp/pull/91))。
CakePHP 2.12 までは、サフィックスの付く設定には名前空間のある名前をサフィックスを省いてしか書けず
(`MemoryEngine::class` ではなく `'Acme\Cache\Engine\Memory'`)、コントローラー、シェル、`$uses` のモデルは
グローバルなクラスにする必要があります。
:::

<a id="the-classes-of-the-framework"></a>

## フレームワークのクラス

::: pieceofcake2 Added in version 2.13.0
フレームワークのクラスは `Cake` 名前空間にあります ([#92](https://github.com/pieceofcake2/cakephp/pull/92))。
:::

フレームワークのクラスは、ディレクトリに合わせた `Cake` 名前空間にあります。たとえば
`Cake\Controller\Controller`、`Cake\Network\CakeRequest`、`Cake\Utility\Hash` です。CakePHP 2 の
グローバルな名前は同じクラスの別名なので、`Controller` と `Cake\Controller\Controller` はどちらを使っても
同じで、これまでのコードは変わりません。ファイルは `src/` にあり (`src/Controller/Controller.php`)、
Composer が PSR-4 で読み込みます。名前空間のあるクラスでは、ほかのクラスと同じようにフレームワークの
クラスを `use` します。

```php
// lib/Controller/Component/GreetingComponent.php
namespace Acme\Controller\Component;

use Cake\Controller\Component;
use Cake\Controller\Controller;

class GreetingComponent extends Component
{
    public function startup(Controller $controller)
    {
        $controller->set('greeting', 'Hello');
    }
}
```

設定にフレームワークのクラスを書くところには、グローバルな名前も名前空間を含む名前も書けます。
たとえば `'routeClass' => RedirectRoute::class` や `Configure::write('Acl.classname', DbAcl::class)`
です。

> [!WARNING]
> `get_class()`、`::class`、`ReflectionClass::getName()` は、`Cake\Network\CakeRequest` のように名前空間を
> 含む名前を返し、エラーログ、`Debugger` の出力、デバッグ用のエラーページにもその名前が出ます。
> `get_class($request) === 'CakeRequest'` のような比較は false になるので、`instanceof` で比べるか、
> クラスを `use` したファイルで `CakeRequest::class` と比べてください。

> [!TIP]
> [pieceofcake2/upgrade](https://github.com/pieceofcake2/upgrade) の
> `bin/cake upgrade rector --rules namespaced-classes src tests` は、コードのグローバルな名前を `use` を付けた
> 名前空間の名前に置き換え、`get_class()` と比べているグローバルな名前を `::class` に置き換えます。

<a id="how-cakephp-looks-up-a-class"></a>

## CakePHP がクラスを探す方法

CakePHP は、設定されたクラスを次の 4 つのどれかの方法で解決します。

- **コーラブル。** 値をコーラブルとして PHP に渡します。たとえば `set_error_handler()` です。
  `[Acme\Error\Handler::class, 'handleError']` を含め、どのコーラブルも使えます。
- **書いたままのクラス名。** 名前に `pluginSplit()` と `App::uses()` を呼び、`class_exists()`
  と `new` を使います。`pluginSplit()` はドットでしか分割しないので、名前空間のある名前は
  そのまま残り、`class_exists()` で Composer が読み込みます。`Foo::class` が使えます。
- **`App::className()`。** `Component` や `Engine` のようなサフィックスを付ける設定では、
  `App::className($name, $type, $suffix)` がクラスを探します。バックスラッシュを含む名前は
  完全なクラス名として扱い、名前がサフィックスで終わらないときだけサフィックスを付けます。
  `MemoryEngine::class` と `'Acme\Cache\Engine\Memory'` のどちらも `Acme\Cache\Engine\MemoryEngine`
  になります。短い名前 (`'Memory'` や `'MyPlugin.Memory'`) は、プラグインの名前空間か
  [`App.namespace`](#app-namespace) で `<名前空間>\<種類>\<名前><サフィックス>` を探し、
  次にグローバルなクラス `<名前><サフィックス>` を探します。
- **エイリアスとして使う名前。** コンポーネント、ヘルパー、ビヘイビア、シェルのタスク、モデルでは、
  名前がプロパティ名やモデルのエイリアスにもなります。完全なクラス名では、名前空間とサフィックスを
  除いたクラス名がエイリアスになります。`GreetingComponent::class` なら `$this->Greeting`、
  `Post::class` ならエイリアス `Post` です。

<a id="summary"></a>

## 一覧

| 設定 | 探し方 | `Foo::class` | 例 |
|---|---|---|---|
| `Error.handler`、`Exception.handler`、`Error.consoleHandler`、`Exception.consoleHandler` | コーラブル | 使える | `[Handler::class, 'handleError']` |
| `Exception.renderer` | `App::className()` | 使える | `Renderer::class` |
| `Exception.skipLog` | `instanceof` | 使える | `[NotFoundException::class, MyException::class]` |
| `Dispatcher.filters` | `App::className()`、またはコーラブル | 使える | `[HeaderFilter::class => ['header' => 'X-Frame-Options']]` |
| データソースの `'datasource'` | `App::className()` | 使える | `'datasource' => MemorySource::class` |
| `Session.handler.engine` | `App::className()` | 使える | `MemoryHandler::class` |
| `'routeClass'`、`Router::defaultRouteClass()` | `App::className()` | 使える | `['routeClass' => SlugRoute::class]` |
| `Acl.classname` | `App::className()` | 使える | `NullAcl::class` |
| `TextHelper`、`NumberHelper`、`TimeHelper` の `'engine'` | `App::className()` | 使える | `['engine' => Text::class]` |
| `CakeEmail::$_configClass` (サブクラスで) | 書いたまま | 使える | `EmailConfig::class` |
| `CakeLog` の `'engine'` | `App::className()`。名前が `Log` で終わらなければサフィックス `Log` | 使える | `MemoryLog::class` |
| `HtmlHelper::loadConfig()` のリーダー | `App::className()`、サフィックス `Reader` | 使える | `['tags', TagReader::class]` |
| `Validation` の地域ごとのルール (`$country`) | `App::className()`、サフィックス `Validation` | 使える | `Validation::postal($check, null, NlValidation::class)` |
| `HttpSocket::$responseClass`、`configAuth()`、`configProxy()` | `App::className()`。認証の方式はサフィックス `Authentication` | 使える | `TokenAuthentication::class` |
| `Cache` の `'engine'` | `App::className()`、サフィックス `Engine` | 使える | `MemoryEngine::class` |
| `CakeEmail` の `'transport'` | `App::className()`、サフィックス `Transport` | 使える | `SesTransport::class` |
| `AuthComponent` の `authenticate`、`authorize`、`passwordHasher` | `App::className()`、サフィックス `Authenticate`、`Authorize`、`PasswordHasher` | 使える | `[TokenAuthenticate::class]` |
| `Controller::$viewClass`、`RequestHandlerComponent::viewClassMap()` | `App::className()`、サフィックス `View` | 使える | `CsvView::class` |
| `$components`、`$helpers`、`$actsAs`、シェルの `$tasks` | `App::className()`。名前がエイリアスになる | 使える。エイリアスは短いクラス名 | `[GreetingComponent::class]` |
| `ClassRegistry::init()`、アソシエーションのキーと `'className'`、`'with'` | `App::className()`。名前がエイリアスになる | 使える。エイリアスは短いクラス名 | `ClassRegistry::init(Post::class)` |
| `Controller::$uses`、`loadModel()`、シェルの `$uses` | 名前がエイリアスになる | 使える。エイリアスは短いクラス名 | `public $uses = [Post::class];` |
| コントローラー、シェル | URL やコマンドラインから `App::className()` | `App.namespace` の短い名前 | `/posts` なら `App\Controller\PostsController` |

<a id="callables-and-class-names-as-given"></a>

## コーラブルと書いたままのクラス名

次の設定には `Foo::class` を書けます。

```php
// config/core.php
use Acme\Error\Handler;
use Acme\Error\Renderer;
use Acme\Model\Datasource\Session\DynamoDbHandler;
use Acme\Routing\Filter\HeaderFilter;

Configure::write('Error', [
    'handler' => [Handler::class, 'handleError'],
    'level' => E_ALL & ~E_DEPRECATED,
]);
Configure::write('Exception', [
    'handler' => [Handler::class, 'handleException'],
    'renderer' => Renderer::class,
    'log' => true,
]);
Configure::write('Session', [
    'defaults' => 'php',
    'handler' => ['engine' => DynamoDbHandler::class],
]);
Configure::write('Dispatcher.filters', [
    'AssetDispatcher',
    HeaderFilter::class => ['header' => 'X-Frame-Options'],
]);
```

`Exception.renderer` は `ErrorHandler::handleException()`、`Session.handler.engine` は
`CakeSession::_getHandler()`、データソースは `ConnectionManager::loadDataSource()`、ルートクラスは
`Router::_validateRouteClass()`、`Acl.classname` は `AclComponent::__construct()` で読み込まれます。
どれも `App::className()` でクラスを探します。完全なクラス名はそのまま使い、短い名前はプラグインの名前空間か
[`App.namespace`](#app-namespace) で探し、これまでどおりグローバルなクラスも探します。

`Dispatcher.filters` (`Dispatcher::_attachFilters()`) には、クラス名、設定の配列のキーにした
クラス名、`'callable'` キーのある配列を書けます。`'callable'` キーには、クラス名か、`'on'` と
組み合わせた配列のコーラブルを書きます。たとえば
`['callable' => [Filters::class, 'before'], 'on' => 'before']` です。
`'Acme\Filters::before'` のような文字列のコーラブルは、文字列がクラス名として扱われるため
使えません。

<a id="class-names-with-a-suffix"></a>

## サフィックスの付くクラス名

`Cache::_buildEngine()`、`CakeEmail::transportClass()`、
`AuthComponent::constructAuthenticate()` と `constructAuthorize()`、
`BaseAuthenticate::passwordHasher()`、`Controller::_getViewObject()`、
`RequestHandlerComponent::renderAs()` は、`App::className()` でクラスを探します。
完全なクラス名を、サフィックスを付けても省いても書けます。

```php
// config/bootstrap.php
use Acme\Cache\Engine\MemoryEngine;
use Acme\Network\Email\SesTransport;

Cache::config('memory', ['engine' => MemoryEngine::class]);

$email = new CakeEmail(['transport' => SesTransport::class]);
$email->send('Hello');
```

```php
// src/Controller/AppController.php
use Acme\Controller\Component\Auth\AdminAuthorize;
use Acme\Controller\Component\Auth\TokenAuthenticate;

class AppController extends Controller
{
    public $components = [
        'Auth' => [
            'authenticate' => [TokenAuthenticate::class],
            'authorize' => [AdminAuthorize::class],
        ],
    ];
}
```

`'Acme\Cache\Engine\Memory'` のようにサフィックスを省いた名前も、これまでどおり使えます。
`CakeLog` (`LogEngineCollection::_getLogger()`) は、名前が `Log` で終わらないときだけ `Log` を
付けるので、`'Acme\Log\Engine\Memory'` だけでなく `MemoryLog::class` も使えます。

`RequestHandlerComponent::viewClassMap()` には、`Controller::$viewClass` と同じ名前を書きます。
たとえば `$this->RequestHandler->viewClassMap('csv', CsvView::class)` です。

<a id="components-helpers-and-behaviors"></a>

## コンポーネント、ヘルパー、ビヘイビア

`ComponentCollection::load()`、`HelperCollection::load()`、`BehaviorCollection::load()`、
`TaskCollection::load()` は、`App::className()` でクラスを探し、名前をエイリアス (コントローラー、
ビュー、シェルのプロパティ名や、`Behaviors->loaded()` の名前) にします。完全なクラス名を書くと、
名前空間とサフィックスを除いたクラス名がエイリアスになります。

```php
// src/Controller/PostsController.php
use Acme\Controller\Component\GreetingComponent;
use Acme\View\Helper\BadgeHelper;

class PostsController extends AppController
{
    public $components = [
        GreetingComponent::class => ['greeting' => 'Hello'],
    ];

    public $helpers = [BadgeHelper::class];
}
```

```php
// src/Model/Post.php
use Acme\Model\Behavior\StampBehavior;

class Post extends AppModel
{
    public $actsAs = [StampBehavior::class];
}
```

これで `$this->Greeting` は `Acme\Controller\Component\GreetingComponent` に、ビューの `$this->Badge` は
`Acme\View\Helper\BadgeHelper` になり、ビヘイビアは `Stamp` という名前で読み込まれます。シェルの
`$tasks` も同じで、`ReportTask::class` は `$this->Report` になります。
`$this->Components->load(GreetingComponent::class)` のように `load()` に渡しても同じです。
`enable()`、`disable()`、`unload()`、`loaded()`、`enabled()`、`set()` にも、エイリアスの代わりに
完全なクラス名を渡せます。

別のエイリアスを使うときは、これまでどおりキーにエイリアスを書き、`className` にクラスを書きます。
たとえば `'Hello' => ['className' => GreetingComponent::class]` なら `$this->Hello` です。
[`App.namespace`](#app-namespace) を設定すれば、`public $components = ['Greeting'];`
のように短い名前で名前空間のクラスを使えます。

親クラス (`AppController`、`AppModel`、プラグインの `AppController`、シェルの親クラス) とサブクラスの
`$components`、`$helpers`、`$actsAs`、`$tasks` をマージするときは、完全なクラス名のキーをエイリアスに
置き換え、クラス名を `className` に入れます。そのため、`AppController` の `'Greeting' => [...]` と
`PostsController` の `GreetingComponent::class => [...]` は、短い名前どうしと同じように、設定を
マージした 1 つの `Greeting` になります。

クラスが見つからないときのエラーには、完全なクラス名がサフィックスで終わらないときだけサフィックスを
付けた名前が出ます。たとえば `Acme\Controller\Component\GreetingComponent` です。

::: pieceofcake2 Changed in version 2.13.0
完全なクラス名のキーから、名前空間とサフィックスを除いたエイリアスを作ります
([#91](https://github.com/pieceofcake2/cakephp/pull/91))。CakePHP 2.12 までは、エイリアスが
バックスラッシュを含む名前になっていました。そのバージョンでも動かすコードでは、
`'Greeting' => ['className' => GreetingComponent::class]` のように短いエイリアスを付けます。
:::

<a id="models"></a>

## モデル

`ClassRegistry::init()` とアソシエーションの `'className'` は `App::className()` でクラスを
探します。エイリアスを付けずに完全なクラス名を書くと、短い名前と同じように、名前空間を除いた
クラス名がエイリアスになります。

```php
// lib/Model/Post.php
namespace Acme\Model;

use AppModel;

class Post extends AppModel
{
    public $belongsTo = [Author::class];

    public $hasAndBelongsToMany = [
        Tag::class => ['with' => PostsTag::class],
    ];
}
```

```php
$Post = ClassRegistry::init(Post::class);
$Post->alias; // 'Post'
```

`belongsTo` の `Author::class` は、`'Author' => ['className' => Author::class]` と同じで、
エイリアス `Author` のアソシエーションになります。別のエイリアスを使うときは、
`'Writer' => ['className' => Author::class]` や
`ClassRegistry::init(['class' => Post::class, 'alias' => 'Article'])` のように書きます。
hasAndBelongsToMany の `'with' => PostsTag::class` は `'with' => 'PostsTag'` と同じで、データの取得、
保存、バリデーション、削除に中間モデル `PostsTag` を使います。

モデルの名前は、名前空間を除いたクラス名 (`Post`) になり、そこからテーブル名 (`posts`) が
作られます。

::: pieceofcake2 Changed in version 2.13.0
CakePHP 2.12 までは名前が名前空間を含み (`Acme\Model\Post`)、テーブル名が `acme\model\posts` に
なっていました。そのバージョンでも動かすモデルには `$name` を書きます。`Controller::$name` と
`Shell::$name` も同じです。
:::

`Controller::$uses`、`Controller::loadModel()`、シェルの `$uses` は、設定した名前から
名前空間を除いたクラス名をプロパティ名とモデルのエイリアスに使います。

```php
// src/Controller/PostsController.php
use Acme\Model\Post;

class PostsController extends AppController
{
    public $uses = [Post::class];
}
```

これで `$this->Post` は `Acme\Model\Post` で、`$this->modelClass` は `Post` です。
`$this->loadModel(Post::class)` も同じです。

`ClassRegistry::getObject()`、`isKeySet()`、`removeObject()` には、完全なクラス名も渡せます。
クラス名から名前空間を除いた名前で登録されたオブジェクトがそのクラスであれば、それを使います。
たとえば `ClassRegistry::getObject(Post::class)` は、エイリアス `Post` のモデルを返します。
無ければ、ちょうどそのクラスのオブジェクトが 1 つだけのときに、そのオブジェクトを使います。
たとえばエイリアス `Article` で作った `Post` です。そのクラスのオブジェクトが複数あれば、
どれも見つかりません。

::: pieceofcake2 Changed in version 2.13.0
`$uses`、`loadModel()`、アソシエーションのキー、エイリアスの無い `ClassRegistry::init()` の
完全なクラス名から、名前空間を除いたエイリアスを作ります
([#91](https://github.com/pieceofcake2/cakephp/pull/91))。CakePHP 2.12 までは、
`$this->{'Acme\Model\Post'}` のように、エイリアスがバックスラッシュを含む名前になっていました。
そのバージョンでも動かすコードでは、`['class' => Post::class, 'alias' => 'Post']` や
`'Author' => ['className' => Author::class]` のように短いエイリアスを付けます。
:::

<a id="app-namespace"></a>

## App.namespace

::: pieceofcake2 Added in version 2.13.0
`App.namespace` の設定 ([#91](https://github.com/pieceofcake2/cakephp/pull/91))。
:::

`App.namespace` に、アプリケーションの名前空間を設定します。たとえば
`config/bootstrap.php` に書きます。

```php
Configure::write('App.namespace', 'App');
```

CakePHP は、まずその名前空間のクラスを探し、無ければグローバルなクラスを探します。

| 名前 | 名前空間のクラス |
|---|---|
| `/posts` (URL)、`PostsController` | `App\Controller\PostsController` |
| `bin/cake report` | `App\Console\Command\ReportShell` |
| `ClassRegistry::init('Post')`、`$uses = ['Post']` | `App\Model\Post` |
| `$components = ['Greeting']` | `App\Controller\Component\GreetingComponent` |
| `$helpers = ['Badge']` | `App\View\Helper\BadgeHelper` |
| `$actsAs = ['Stamp']` | `App\Model\Behavior\StampBehavior` |
| `Cache` の `'engine' => 'Memory'` | `App\Cache\Engine\MemoryEngine` |

名前空間は、それぞれのクラスのパッケージに合わせます (`<名前空間>\<種類>\<名前><サフィックス>`)。
`composer.json` の PSR-4 に、たとえば `"App\\": "src/"` と登録します。`AuthComponent` や
`FileEngine` のようなフレームワークのグローバルなクラスは、名前空間に同じ名前のクラスが無ければ、
これまでどおり使われます。

> [!NOTE]
> `App.namespace` に既定値はありません。設定しなければ、これまでどおりグローバルなクラスだけを探します。
> CakePHP 5 でも、アプリケーションの設定 (`config/app.php` の `'App' => ['namespace' => 'App']`) から
> 読みます。

> [!WARNING]
> 名前空間のクラスは、同じ短い名前のグローバルなクラスより先に使われます。たとえば `AuthComponent` ではなく
> `App\Controller\Component\AuthComponent` が使われます。

プラグインの接頭辞が付いた名前 (`'MyPlugin.Greeting'`) は、`App.namespace` ではなくプラグインの
名前空間 (`MyPlugin\Controller\Component\GreetingComponent`) で探し、次にプラグインの
グローバルなクラスを探します。

### AppController と AppModel

名前空間のコントローラーは、親クラスの `AppController` (`App\Controller\AppController`) と
`$components` と `$helpers` をマージします。グローバルなコントローラーがグローバルな
`AppController` とマージするのと同じです。親クラスは、名前が `AppController` であれば、
`App.namespace` を設定していなくても、どの名前空間のクラスでもかまいません。プラグインの
コントローラーは、プラグインの名前空間の `AppController`
(`MyPlugin\Controller\MyPluginAppController`) ともマージします。名前空間のモデルも同じように、
`App\Model\AppModel` と `$actsAs` と `$findMethods` をマージします。

`CakeErrorController` のように、フレームワークには、グローバルな `AppController`、`AppModel`、
`AppHelper`、`AppShell` を継承するクラスがあります。アプリケーションにこれらのクラスが
`App.namespace` にしかないときは、`App::load()` がそのクラスにグローバルな名前を付けます。
グローバルなクラスがあれば、そちらが使われます。

### フィクスチャー

フィクスチャーも名前空間のクラスにできます。`$fixtures` の名前から、次のクラスを探し、
無ければこれまでどおりグローバルなクラスを探します。

| 名前 | 名前空間のクラス |
|---|---|
| `'app.article'` | `App\Test\Fixture\ArticleFixture` (`App.namespace` の名前空間) |
| `'plugin.my_plugin.article'` | `MyPlugin\Test\Fixture\ArticleFixture` |
| `'core.article'` | `Cake\Test\Fixture\ArticleFixture` |

フィクスチャーの名前とテーブル名は、名前空間を除いたクラス名から作られます
(`ArticleFixture` なら `Article` と `articles`)。

### キャッシュしたビュー

`Controller::$controllerClass` と `View::$controllerClass` は、コントローラーのクラス名
(`App\Controller\PostsController` など) です。`CacheHelper` がキャッシュするビューは、この名前で
名前空間のコントローラーを作ります。

<a id="what-does-not-work"></a>

## 使えないもの

- `App::uses()` では名前空間のあるクラスを読み込めません。`App::load()` は、名前全体を
  ファイル名にしてパッケージの中を探します。名前空間のあるクラスは Composer が読み込むので、
  `App::uses()` は要りません。`App::import()` は、名前空間のあるクラスも
  [`App::className()`](/ja/core-utility-libraries/app) で探します。
