# Namespaced Classes

CakePHP 2 classes are global classes without a namespace, but your own code can
use namespaces. Register them with a PSR-4 entry in `composer.json`, next to
the classmap of the application skeleton, and run `composer dump-autoload`:

```json
"autoload": {
    "classmap": ["src/"],
    "psr-4": {
        "Acme\\": "lib/"
    }
}
```

Composer then loads a class such as `Acme\Error\Renderer` from
`lib/Error/Renderer.php` on first use, without `App::uses()`. Where CakePHP
takes a class name in its configuration, you can give it a namespaced class
as `Foo::class`. This page lists the configuration points of
pieceofcake2/cakephp and how CakePHP finds the class for each of them, and
how `App.namespace` lets CakePHP find controllers, models and other classes
in your namespace by their short names.

::: pieceofcake2 Added in version 2.13.0
`App::className()` and `App.namespace`
([#91](https://github.com/pieceofcake2/cakephp/pull/91)). In CakePHP 2.12, the
settings with a suffix take a namespaced name only without the suffix
(`'Acme\Cache\Engine\Memory'`, not `MemoryEngine::class`), and controllers, shells
and models in `$uses` must be global classes.
:::

> [!NOTE]
> The classes of the framework itself are still global; the `Cake\` namespace
> of pull requests [#23](https://github.com/pieceofcake2/cakephp/pull/23) and
> [#28](https://github.com/pieceofcake2/cakephp/pull/28) is only on the
> `feature/phpstan-level5` branch.

## How CakePHP Looks Up a Class

CakePHP resolves a configured class in one of four ways:

- **Callable.** The value is passed to PHP as a callable, for example to
  `set_error_handler()`. Any callable works, including
  `[Acme\Error\Handler::class, 'handleError']`.
- **Class name as given.** CakePHP calls `pluginSplit()` and `App::uses()` on
  the name, then `class_exists()` and `new`. `pluginSplit()` only splits at a
  dot, so a namespaced name passes through unchanged, and `class_exists()`
  lets Composer load it. `Foo::class` works.
- **`App::className()`.** For the settings that add a suffix such as
  `Component` or `Engine`, `App::className($name, $type, $suffix)` finds the
  class. A name with a backslash is a fully qualified class name, and the
  suffix is added only when the name does not end with it: both
  `MemoryEngine::class` and `'Acme\Cache\Engine\Memory'` give
  `Acme\Cache\Engine\MemoryEngine`. A short name (`'Memory'` or `'MyPlugin.Memory'`)
  is looked up in the namespace of the plugin or in
  [`App.namespace`](#app-namespace) as
  `<Namespace>\<Type>\<Name><Suffix>`, then as the global class
  `<Name><Suffix>`.
- **Name used as an alias.** For components, helpers, behaviors and models,
  the name also becomes a property or a model alias. A namespaced name then
  gives a property like `$this->{'Acme\Model\Post'}` and an SQL alias with
  backslashes, so give a short alias.

## Summary

| Configuration | Lookup | `Foo::class` | Example |
|---|---|---|---|
| `Error.handler`, `Exception.handler`, `Error.consoleHandler`, `Exception.consoleHandler` | Callable | Yes | `[Handler::class, 'handleError']` |
| `Exception.renderer` | `App::className()` | Yes | `Renderer::class` |
| `Exception.skipLog` | `instanceof` | Yes | `[NotFoundException::class, MyException::class]` |
| `Dispatcher.filters` | `App::className()`, or a callable | Yes | `[HeaderFilter::class => ['header' => 'X-Frame-Options']]` |
| Datasource `'datasource'` | `App::className()` | Yes | `'datasource' => MemorySource::class` |
| `Session.handler.engine` | `App::className()` | Yes | `MemoryHandler::class` |
| `'routeClass'`, `Router::defaultRouteClass()` | `App::className()` | Yes | `['routeClass' => SlugRoute::class]` |
| `Acl.classname` | `App::className()` | Yes | `NullAcl::class` |
| `'engine'` of `TextHelper`, `NumberHelper`, `TimeHelper` | `App::className()` | Yes | `['engine' => Text::class]` |
| `CakeEmail::$_configClass` (in a subclass) | As given | Yes | `EmailConfig::class` |
| `CakeLog` `'engine'` | `App::className()`, suffix `Log` unless the name ends with it | Yes | `MemoryLog::class` |
| `HtmlHelper::loadConfig()` reader | `App::className()`, suffix `Reader` | Yes | `['tags', TagReader::class]` |
| Localized `Validation` rules (`$country`) | `App::className()`, suffix `Validation` | Yes | `Validation::postal($check, null, NlValidation::class)` |
| `HttpSocket::$responseClass`, `configAuth()`, `configProxy()` | `App::className()`, suffix `Authentication` for the methods | Yes | `TokenAuthentication::class` |
| `Cache` `'engine'` | `App::className()`, suffix `Engine` | Yes | `MemoryEngine::class` |
| `CakeEmail` `'transport'` | `App::className()`, suffix `Transport` | Yes | `SesTransport::class` |
| `AuthComponent` `authenticate`, `authorize`; `passwordHasher` | `App::className()`, suffix `Authenticate`, `Authorize`, `PasswordHasher` | Yes | `[TokenAuthenticate::class]` |
| `Controller::$viewClass`, `RequestHandlerComponent::viewClassMap()` | `App::className()`, suffix `View` | Yes | `CsvView::class` |
| `$components`, `$helpers`, `$actsAs` | `App::className()`, and the name is the alias | Yes, with an alias and `className` | `'Greeting' => ['className' => GreetingComponent::class]` |
| `ClassRegistry::init()`, association `'className'` | `App::className()` | Yes, with an alias | `['class' => Post::class, 'alias' => 'Post']` |
| `Controller::$uses`, `loadModel()` | The name is the alias | Short names in `App.namespace` | `public $uses = ['Post'];` |
| Controllers, shells | `App::className()` from the URL or the command line | Short names in `App.namespace` | `App\Controller\PostsController` for `/posts` |

## Callables and Class Names as Given

These settings accept `Foo::class`:

```php
// config/core.php
use Acme\Error\Handler;
use Acme\Error\Renderer;
use Acme\Routing\Filter\HeaderFilter;
use Acme\Model\Datasource\Session\DynamoDbHandler;

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

`Exception.renderer` is loaded in `ErrorHandler::handleException()`,
`Session.handler.engine` in `CakeSession::_getHandler()`, the datasource in
`ConnectionManager::loadDataSource()`, the route class in
`Router::_validateRouteClass()` and `Acl.classname` in
`AclComponent::__construct()`. All of them find the class with
`App::className()`: a fully qualified name as given, or a short name in the
namespace of a plugin, in [`App.namespace`](#app-namespace) or as a global
class, as before.

`Dispatcher.filters` (`Dispatcher::_attachFilters()`) takes a class name, a
class name as the key of its settings, or an array with a `'callable'` key.
The `'callable'` key takes a class name or an array callable together with
`'on'`, such as
`['callable' => [Filters::class, 'before'], 'on' => 'before']`. A string
callable such as `'Acme\Filters::before'` does not work there, because a
string is treated as a class name.

## Class Names with a Suffix

`Cache::_buildEngine()`, `CakeEmail::transportClass()`,
`AuthComponent::constructAuthenticate()` and `constructAuthorize()`,
`BaseAuthenticate::passwordHasher()`, `Controller::_getViewObject()` and
`RequestHandlerComponent::renderAs()` find the class with `App::className()`.
Give the fully qualified class name, with or without the suffix:

```php
// config/bootstrap.php
use Acme\Cache\Engine\MemoryEngine;
use Acme\Network\Email\SesTransport;

Cache::config('memory', ['engine' => MemoryEngine::class]);

$email = new CakeEmail(['transport' => SesTransport::class]);
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

The name without the suffix, such as `'Acme\Cache\Engine\Memory'`, keeps working.
`CakeLog` (`LogEngineCollection::_getLogger()`) appends `Log` only when the
name does not end with it, so `MemoryLog::class` works as well as
`'Acme\Log\Engine\Memory'`.

`RequestHandlerComponent::viewClassMap()` takes the same names as
`Controller::$viewClass`, for example
`$this->RequestHandler->viewClassMap('csv', CsvView::class)`.

## Components, Helpers and Behaviors

`ComponentCollection::load()`, `HelperCollection::load()` and
`BehaviorCollection::load()` find the class with `App::className()` and use
the name as the alias: the property of the controller or view, and the name
in `Behaviors->loaded()`. Give a short alias as the key and the class in
`className`:

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

`$this->Greeting` is then an `Acme\Controller\Component\GreetingComponent`.
A fully qualified class name as the key, without `className`, becomes an alias
with backslashes. With [`App.namespace`](#app-namespace), a short name finds
the class in your namespace: `public $components = ['Greeting'];`.

## Models

`ClassRegistry::init()` and the `'className'` of an association find the class
with `App::className()`, so namespaced models work when they have a plain
alias:

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

The name of the model is the class name without its namespace (`Post`), and
the table name is derived from it (`posts`).

::: pieceofcake2 Changed in version 2.13.0
In CakePHP 2.12, the name kept the namespace (`Acme\Model\Post`), which gave
the table `acme\model\posts`. Set `$name` in models that must also work there.
The same applies to `Controller::$name` and `Shell::$name`.
:::

`Controller::$uses` and `Controller::loadModel()` use the configured name as
the property and the model alias, so a fully qualified name there gives
`$this->{'Acme\Model\Post'}` and queries with an `Acme\Model\Post` alias. Use
short names with [`App.namespace`](#app-namespace), or load the models with
`ClassRegistry::init()`, for example in `beforeFilter()`.

<a id="app-namespace"></a>

## App.namespace

::: pieceofcake2 Added in version 2.13.0
The `App.namespace` setting ([#91](https://github.com/pieceofcake2/cakephp/pull/91)).
:::

Set `App.namespace` to the namespace of your application, for example in
`config/bootstrap.php`:

```php
Configure::write('App.namespace', 'App');
```

CakePHP then looks for the classes in that namespace first, and for the global
classes after them:

| Name | Class in the namespace |
|---|---|
| `/posts` (URL), `PostsController` | `App\Controller\PostsController` |
| `bin/cake report` | `App\Console\Command\ReportShell` |
| `ClassRegistry::init('Post')`, `$uses = ['Post']` | `App\Model\Post` |
| `$components = ['Greeting']` | `App\Controller\Component\GreetingComponent` |
| `$helpers = ['Badge']` | `App\View\Helper\BadgeHelper` |
| `$actsAs = ['Stamp']` | `App\Model\Behavior\StampBehavior` |
| `Cache` `'engine' => 'Memory'` | `App\Cache\Engine\MemoryEngine` |

The namespace follows the package of each class (`<Namespace>\<Type>\<Name><Suffix>`).
Register it with PSR-4 in `composer.json`, for example `"App\\": "src/"`.
The global classes of the framework, such as `AuthComponent` or `FileEngine`,
are still found when the namespace has no class of that name.

> [!NOTE]
> `App.namespace` has no default value: when it is not set, CakePHP only looks
> for global classes, as before. CakePHP 5 also reads it from the configuration
> of the application (`'App' => ['namespace' => 'App']` in `config/app.php`).

> [!WARNING]
> A class in the namespace is used instead of a global class with the same
> short name, for example `App\Controller\Component\AuthComponent` instead of
> `AuthComponent`.

A name with a plugin prefix (`'MyPlugin.Greeting'`) is looked up in the
namespace of the plugin (`MyPlugin\Controller\Component\GreetingComponent`)
instead of `App.namespace`, then as the global class of the plugin.

### AppController and AppModel

A controller in the namespace merges `$components` and `$helpers` with its
parent class `AppController` (`App\Controller\AppController`), as a global
controller does with the global `AppController`. The parent class may be in any
namespace, also without `App.namespace`, as long as it is named
`AppController`. A plugin controller also merges them with the `AppController`
of the namespace of its plugin (`MyPlugin\Controller\MyPluginAppController`).
A model in the namespace merges `$actsAs` and `$findMethods` with
`App\Model\AppModel` in the same way.

Some classes of the framework, such as `CakeErrorController`, extend the global
`AppController`, `AppModel`, `AppHelper` or `AppShell`. When the application
has these classes only in `App.namespace`, `App::load()` gives the global name
to that class. A global class is used when there is one.

### Fixtures

Fixtures may be namespaced classes too. For the names in `$fixtures`, CakePHP
looks for these classes, then for the global classes as before:

| Name | Class in the namespace |
|---|---|
| `'app.article'` | `App\Test\Fixture\ArticleFixture` (in `App.namespace`) |
| `'plugin.my_plugin.article'` | `MyPlugin\Test\Fixture\ArticleFixture` |
| `'core.article'` | `Cake\Test\Fixture\ArticleFixture` |

The name and the table of a fixture come from the class name without the
namespace (`Article` and `articles` for `ArticleFixture`).

### Cached Views

`Controller::$controllerClass` and `View::$controllerClass` hold the class name
of the controller, such as `App\Controller\PostsController`. The views cached
by `CacheHelper` create a namespaced controller by that name.

## What Does Not Work

- `App::uses()` does not load namespaced classes: `App::load()` would look for
  a file named after the whole name in the package. Composer loads them, so
  `App::uses()` is not needed. `App::import()` also finds namespaced classes
  with [`App::className()`](/en/core-utility-libraries/app).

> [!WARNING]
> Fully qualified class names as the keys of `$uses`, `$components`,
> `$helpers` and `$actsAs` become aliases with backslashes. Use short aliases
> with `className`.
