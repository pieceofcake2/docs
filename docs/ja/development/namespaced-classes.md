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
`lib/Error/Renderer.php` から読み込みます。`App::uses()` は要りません。CakePHP の設定に
クラス名を書く場所には、名前空間のあるクラスを `Foo::class` で書けます。このページでは、
pieceofcake2/cakephp の設定項目と、それぞれで CakePHP がクラスを探す方法をまとめます。
また、`App.namespace` を設定して、コントローラーやモデルなどのクラスをアプリケーションの
名前空間から短い名前で探す方法も説明します。

> [!NOTE]
> `App::className()` と `App.namespace` は CakePHP 2.13 から使えます
> ([#91](https://github.com/pieceofcake2/cakephp/pull/91))。CakePHP 2.12 までは、
> サフィックスの付く設定には名前空間のある名前をサフィックスを省いてしか書けず
> (`MemoryEngine::class` ではなく `'Acme\Cache\Memory'`)、コントローラー、シェル、`$uses` の
> モデルはグローバルなクラスにする必要があります。

<a id="the-classes-of-the-framework"></a>

## フレームワークのクラス

CakePHP 2.13 から ([#92](https://github.com/pieceofcake2/cakephp/pull/92))、
フレームワークのクラスはディレクトリに合わせた `Cake` 名前空間にあります。たとえば
`Cake\Controller\Controller`、`Cake\Network\CakeRequest`、`Cake\Utility\Hash` です。
CakePHP 2 のグローバルな名前は同じクラスの別名なので、`Controller` と
`Cake\Controller\Controller` はどちらを使っても同じで、これまでのコードは変わりません。
名前空間のあるクラスでは、ほかのクラスと同じようにフレームワークのクラスを `use` します。

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

`get_class()` と `::class` は名前空間を含む名前を返すので、クラスはグローバルな名前の文字列と
比べずに、`instanceof` で比べます。

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
  `MemoryEngine::class` と `'Acme\Cache\Memory'` のどちらも `Acme\Cache\MemoryEngine`
  になります。短い名前 (`'Memory'` や `'MyPlugin.Memory'`) は、プラグインの名前空間か
  [`App.namespace`](#app-namespace) で `<名前空間>\<種類>\<名前><サフィックス>` を探し、
  次にグローバルなクラス `<名前><サフィックス>` を探します。
- **エイリアスとして使う名前。** コンポーネント、ヘルパー、ビヘイビア、モデルでは、名前が
  プロパティ名やモデルのエイリアスにもなります。名前空間のある名前では、
  `$this->{'Acme\Model\Post'}` のようなプロパティと、バックスラッシュを含む SQL の
  エイリアスになるので、短いエイリアスを付けます。

<a id="summary"></a>

## 一覧

| 設定 | 探し方 | `Foo::class` | 例 |
|---|---|---|---|
| `Error.handler`、`Exception.handler`、`Error.consoleHandler`、`Exception.consoleHandler` | コーラブル | 使える | `[Handler::class, 'handleError']` |
| `Exception.renderer` | 書いたまま | 使える | `Renderer::class` |
| `Exception.skipLog` | `instanceof` | 使える | `[NotFoundException::class, MyException::class]` |
| `Dispatcher.filters` | 書いたまま、またはコーラブル | 使える | `[HeaderFilter::class => ['header' => 'X-Frame-Options']]` |
| データソースの `'datasource'` | 書いたまま | 使える | `'datasource' => MemorySource::class` |
| `Session.handler.engine` | 書いたまま | 使える | `MemoryHandler::class` |
| `'routeClass'`、`Router::defaultRouteClass()` | 書いたまま | 使える | `['routeClass' => SlugRoute::class]` |
| `Acl.classname` | 書いたまま | 使える | `NullAcl::class` |
| `TextHelper`、`NumberHelper`、`TimeHelper` の `'engine'` | 書いたまま | 使える | `['engine' => Text::class]` |
| `CakeEmail::$_configClass` (サブクラスで) | 書いたまま | 使える | `EmailConfig::class` |
| `CakeLog` の `'engine'` | 名前が `Log` で終わらなければサフィックス `Log` | 使える | `MemoryLog::class` |
| `Cache` の `'engine'` | `App::className()`、サフィックス `Engine` | 使える | `MemoryEngine::class` |
| `CakeEmail` の `'transport'` | `App::className()`、サフィックス `Transport` | 使える | `SesTransport::class` |
| `AuthComponent` の `authenticate`、`authorize`、`passwordHasher` | `App::className()`、サフィックス `Authenticate`、`Authorize`、`PasswordHasher` | 使える | `[TokenAuthenticate::class]` |
| `Controller::$viewClass`、`RequestHandlerComponent::viewClassMap()` | `App::className()`、サフィックス `View` | 使える | `CsvView::class` |
| `$components`、`$helpers`、`$actsAs` | `App::className()`。名前がエイリアスになる | エイリアスを付けて `className` で指定 | `'Greeting' => ['className' => GreetingComponent::class]` |
| `ClassRegistry::init()`、アソシエーションの `'className'` | `App::className()` | エイリアスがあれば使える | `['class' => Post::class, 'alias' => 'Post']` |
| `Controller::$uses`、`loadModel()` | 名前がエイリアスになる | `App.namespace` の短い名前 | `public $uses = ['Post'];` |
| コントローラー、シェル | URL やコマンドラインから `App::className()` | `App.namespace` の短い名前 | `/posts` なら `App\Controller\PostsController` |

<a id="callables-and-class-names-as-given"></a>

## コーラブルと書いたままのクラス名

次の設定には `Foo::class` を書けます。

```php
// config/core.php
use Acme\Error\Handler;
use Acme\Error\Renderer;
use Acme\Routing\Filter\HeaderFilter;
use Acme\Session\DynamoDbHandler;

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
どれも設定した名前をそのまま使います。

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
use Acme\Cache\MemoryEngine;
use Acme\Mail\SesTransport;

Cache::config('memory', ['engine' => MemoryEngine::class]);

$email = new CakeEmail(['transport' => SesTransport::class]);
```

```php
// src/Controller/AppController.php
use Acme\Auth\AdminAuthorize;
use Acme\Auth\TokenAuthenticate;

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

`'Acme\Cache\Memory'` のようにサフィックスを省いた名前も、これまでどおり使えます。
`CakeLog` (`LogEngineCollection::_getLogger()`) は、名前が `Log` で終わらないときだけ `Log` を
付けるので、`'Acme\Log\Memory'` だけでなく `MemoryLog::class` も使えます。

`RequestHandlerComponent::viewClassMap()` には、`Controller::$viewClass` と同じ名前を書きます。
たとえば `$this->RequestHandler->viewClassMap('csv', CsvView::class)` です。

<a id="components-helpers-and-behaviors"></a>

## コンポーネント、ヘルパー、ビヘイビア

`ComponentCollection::load()`、`HelperCollection::load()`、`BehaviorCollection::load()` は、
`App::className()` でクラスを探し、名前をエイリアス (コントローラーやビューのプロパティ名、
`Behaviors->loaded()` の名前) にします。キーに短いエイリアスを書き、`className` に
クラスを書きます。

```php
// src/Controller/PostsController.php
use Acme\Controller\Component\GreetingComponent;
use Acme\View\Helper\BadgeHelper;

class PostsController extends AppController
{
    public $components = [
        'Greeting' => ['className' => GreetingComponent::class],
    ];

    public $helpers = [
        'Badge' => ['className' => BadgeHelper::class],
    ];
}
```

```php
// src/Model/Post.php
use Acme\Model\Behavior\StampBehavior;

class Post extends AppModel
{
    public $actsAs = [
        'Stamp' => ['className' => StampBehavior::class],
    ];
}
```

これで `$this->Greeting` は `Acme\Controller\Component\GreetingComponent` になります。
`className` を使わずに完全なクラス名をキーに書くと、エイリアスがバックスラッシュを含む名前に
なります。[`App.namespace`](#app-namespace) を設定すれば、`public $components = ['Greeting'];`
のように短い名前で名前空間のクラスを使えます。

<a id="models"></a>

## モデル

`ClassRegistry::init()` とアソシエーションの `'className'` は `App::className()` でクラスを
探すので、名前空間のあるモデルも、普通のエイリアスを付ければ使えます。

```php
// lib/Model/Post.php
namespace Acme\Model;

use AppModel;

class Post extends AppModel
{
    public $belongsTo = [
        'Author' => ['className' => Author::class],
    ];
}
```

```php
$Post = ClassRegistry::init(['class' => Post::class, 'alias' => 'Post']);
```

モデルの名前は、名前空間を除いたクラス名 (`Post`) になり、そこからテーブル名 (`posts`) が
作られます。CakePHP 2.12 までは名前が名前空間を含んでいたので、そのバージョンでも動かす
モデルには `$name` を書きます。

`Controller::$uses` と `Controller::loadModel()` は、設定した名前をプロパティ名とモデルの
エイリアスに使います。完全なクラス名を書くと、`$this->{'Acme\Model\Post'}` になり、
`Acme\Model\Post` というエイリアスでクエリーが作られます。[`App.namespace`](#app-namespace)
を設定して短い名前を書くか、`beforeFilter()` などで `ClassRegistry::init()` を使って
読み込みます。

<a id="app-namespace"></a>

## App.namespace

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
`composer.json` の PSR-4 に、たとえば `"App\\": "src/"` と登録します。名前空間のコントローラーは、
グローバルなコントローラーがグローバルな `AppController` とするのと同じように、名前空間の
`AppController` (`App\Controller\AppController`) と `$components` と `$helpers` を
マージします。`AuthComponent` や `FileEngine` のようなフレームワークのグローバルなクラスは、
名前空間に同じ名前のクラスが無ければ、これまでどおり使われます。

プラグインの接頭辞が付いた名前 (`'MyPlugin.Greeting'`) は、`App.namespace` ではなくプラグインの
名前空間 (`MyPlugin\Controller\Component\GreetingComponent`) で探し、次にプラグインの
グローバルなクラスを探します。

<a id="what-does-not-work"></a>

## 使えないもの

- `App::uses()` では名前空間のあるクラスを読み込めません。`App::load()` は、名前全体を
  ファイル名にしてパッケージの中を探します。名前空間のあるクラスは Composer が読み込むので、
  `App::uses()` は要りません。
- `$uses`、`$components`、`$helpers`、`$actsAs` のキーに完全なクラス名を書くと、エイリアスが
  バックスラッシュを含む名前になります。短いエイリアスを付けます。
