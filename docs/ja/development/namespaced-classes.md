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
`lib/Error/Renderer.php` から読み込みます。`App::uses()` は要りません。ただし、CakePHP の
設定にクラス名を書く場所で名前空間のあるクラスを使えるかどうかは、CakePHP がクラスをどう探すかで
決まります。このページでは、pieceofcake2/cakephp v2.12.x の設定項目と、それぞれで名前空間のある
クラスを使う方法をまとめます。

> [!NOTE]
> フレームワーク自体の名前空間対応 (`Cake\` 名前空間、`App.namespace`、`App::className()`) は、
> pieceofcake2/cakephp のプルリクエスト
> [#23](https://github.com/pieceofcake2/cakephp/pull/23)、
> [#28](https://github.com/pieceofcake2/cakephp/pull/28)、
> [#29](https://github.com/pieceofcake2/cakephp/pull/29) でマージされましたが、
> `feature/phpstan-level5` ブランチにあります。`2.x` ブランチにも v2.12.x のどのリリースにも
> 含まれておらず、このページの内容はそれには当てはまりません。

<a id="how-cakephp-looks-up-a-class"></a>

## CakePHP がクラスを探す方法

CakePHP は、設定されたクラスを次の 4 つのどれかの方法で解決します。

- **コーラブル。** 値をコーラブルとして PHP に渡します。たとえば `set_error_handler()` です。
  `[Acme\Error\Handler::class, 'handleError']` を含め、どのコーラブルも使えます。
- **書いたままのクラス名。** 名前に `pluginSplit()` と `App::uses()` を呼び、`class_exists()`
  と `new` を使います。`pluginSplit()` はドットでしか分割しないので、名前空間のある名前は
  そのまま残り、`class_exists()` で Composer が読み込みます。`Foo::class` が使えます。
- **名前とサフィックス。** 名前に `Component` や `Engine` のようなサフィックスを付けます。
  名前空間のあるクラスは、クラス名がサフィックスで終わり、設定ではサフィックスを省いた場合に
  だけ使えます。`'Acme\Cache\Memory'` は `Acme\Cache\MemoryEngine` を読み込みますが、
  `MemoryEngine::class` は `Acme\Cache\MemoryEngineEngine` を探して失敗します。
- **エイリアスとして使う名前。** 名前がプロパティ名やモデルのエイリアスにもなります。
  名前空間のある名前では、`$this->{'Acme\Model\Post'}` のようなプロパティと、
  バックスラッシュを含む SQL のエイリアスになります。

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
| `CakeLog` の `'engine'` | 名前が `Log` で終わらなければサフィックス `Log` | クラス名が `Log` で終われば使える | `MemoryLog::class` |
| `Cache` の `'engine'` | サフィックス `Engine` | サフィックスを省く | `'Acme\Cache\Memory'` |
| `CakeEmail` の `'transport'` | サフィックス `Transport` | サフィックスを省く | `'Acme\Mail\Memory'` |
| `AuthComponent` の `authenticate`、`authorize`、`passwordHasher` | サフィックス `Authenticate`、`Authorize`、`PasswordHasher` | サフィックスを省く | `['Acme\Auth\Token']` |
| `Controller::$viewClass`、`RequestHandlerComponent::viewClassMap()` | サフィックス `View` | サフィックスを省く | `'Acme\View\Csv'` |
| `$components`、`$helpers`、`$actsAs` | サフィックス。名前がエイリアスになる | サフィックスを省き、`className` で指定 | `'Greeting' => ['className' => 'Acme\Controller\Component\Greeting']` |
| `ClassRegistry::init()`、アソシエーションの `'className'` | 書いたまま | エイリアスと `$name` があれば使える | `['class' => Post::class, 'alias' => 'Post']` |
| `Controller::$uses`、`loadModel()` | 名前がエイリアスになる | 使えない | `ClassRegistry::init()` を使う |
| コントローラー、シェル、`App::uses()` | グローバルなクラス名 | 使えない | |

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
`BaseAuthenticate::passwordHasher()`、`Controller::_getViewObject()` は、設定した名前に
サフィックスを付けます。クラス名にはサフィックスを付け、設定ではサフィックスを省いた文字列を
書きます。

```php
// lib/Cache/MemoryEngine.php: class Acme\Cache\MemoryEngine
Cache::config('memory', ['engine' => 'Acme\Cache\Memory']);

// lib/Mail/SesTransport.php: class Acme\Mail\SesTransport
$email = new CakeEmail(['transport' => 'Acme\Mail\Ses']);

// lib/Auth/TokenAuthenticate.php: class Acme\Auth\TokenAuthenticate
public $components = [
    'Auth' => [
        'authenticate' => ['Acme\Auth\Token'],
        'authorize' => ['Acme\Auth\Admin'],
    ],
];
```

`CakeLog` (`LogEngineCollection::_getLogger()`) は、名前が `Log` で終わらないときだけ `Log` を
付けるので、`'Acme\Log\Memory'` だけでなく `MemoryLog::class` も使えます。

`RequestHandlerComponent::viewClassMap()` には、`Controller::$viewClass` と同じく `View` を
省いた名前を書きます。たとえば `Acme\View\CsvView` なら
`$this->RequestHandler->viewClassMap('csv', 'Acme\View\Csv')` です。

<a id="components-helpers-and-behaviors"></a>

## コンポーネント、ヘルパー、ビヘイビア

`ComponentCollection::load()`、`HelperCollection::load()`、`BehaviorCollection::load()` は、
`Component`、`Helper`、`Behavior` を付け、名前をエイリアス (コントローラーやビューの
プロパティ名、`Behaviors->loaded()` の名前) にします。キーに短いエイリアスを書き、
`className` にサフィックスを省いた名前空間のある名前を書きます。

```php
// src/Controller/PostsController.php
public $components = [
    'Greeting' => ['className' => 'Acme\Controller\Component\Greeting'],
];
public $helpers = [
    'Badge' => ['className' => 'Acme\View\Helper\Badge'],
];
```

```php
// src/Model/Post.php
public $actsAs = [
    'Stamp' => ['className' => 'Acme\Model\Behavior\Stamp'],
];
```

これで `$this->Greeting` は `Acme\Controller\Component\GreetingComponent` になります。
`className` に `GreetingComponent::class` を書くと、サフィックスがもう一度付くので使えません。

<a id="models"></a>

## モデル

`ClassRegistry::init()` とアソシエーションの `'className'` はクラス名をそのまま使うので、
名前空間のあるモデルも、普通のエイリアスを付ければ使えます。

```php
// lib/Model/Post.php
namespace Acme\Model;

use AppModel;

class Post extends AppModel
{
    public $name = 'Post';

    public $belongsTo = [
        'Author' => ['className' => Author::class],
    ];
}
```

```php
$Post = ClassRegistry::init(['class' => Post::class, 'alias' => 'Post']);
```

モデルには `$name` を書きます。書かないと、名前が名前空間を含むクラス名になり、そこから
既定のエイリアスとテーブル名 (`acme\model\posts`) が作られます。

`Controller::$uses` と `Controller::loadModel()` は、設定した名前をプロパティ名とモデルの
エイリアスに使います。名前空間のある名前を書くと、`$this->{'Acme\Model\Post'}` になり、
`Acme\Model\Post` というエイリアスでクエリーが作られます。名前空間のあるモデルは、
`beforeFilter()` などで `ClassRegistry::init()` を使って読み込みます。

<a id="what-does-not-work"></a>

## 使えないもの

- コントローラーは URL から `PostsController` として (`Dispatcher::_loadController()`)、
  シェルはコマンドラインから `PostsShell` として探すので、グローバルなクラスにします。
- `App::uses()` では名前空間のあるクラスを読み込めません。`App::load()` は、名前全体を
  ファイル名にしてパッケージの中を探します。名前空間のあるクラスは Composer が読み込むので、
  `App::uses()` は要りません。
