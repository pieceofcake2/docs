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
takes a class name in its configuration, however, whether you can give it a
namespaced class depends on how CakePHP looks the class up. This page lists the
configuration points of pieceofcake2/cakephp v2.12.x and how to use a
namespaced class with each of them.

> [!NOTE]
> Namespace support for the framework itself (the `Cake\` namespace,
> `App.namespace` and `App::className()`), merged in pull requests
> [#23](https://github.com/pieceofcake2/cakephp/pull/23),
> [#28](https://github.com/pieceofcake2/cakephp/pull/28) and
> [#29](https://github.com/pieceofcake2/cakephp/pull/29), is on the
> `feature/phpstan-level5` branch. It is not part of the `2.x` branch or of
> any v2.12.x release, and this page does not apply to it.

## How CakePHP Looks Up a Class

CakePHP resolves a configured class in one of four ways:

- **Callable.** The value is passed to PHP as a callable, for example to
  `set_error_handler()`. Any callable works, including
  `[Acme\Error\Handler::class, 'handleError']`.
- **Class name as given.** CakePHP calls `pluginSplit()` and `App::uses()` on
  the name, then `class_exists()` and `new`. `pluginSplit()` only splits at a
  dot, so a namespaced name passes through unchanged, and `class_exists()`
  lets Composer load it. `Foo::class` works.
- **Name plus a suffix.** CakePHP appends a suffix such as `Component` or
  `Engine` to the name. A namespaced class works only if its name ends with
  the suffix and you leave the suffix out: `'Acme\Cache\Memory'` loads
  `Acme\Cache\MemoryEngine`, while `MemoryEngine::class` looks for
  `Acme\Cache\MemoryEngineEngine` and fails.
- **Name used as an alias.** The name also becomes a property or a model
  alias. A namespaced name then gives a property like
  `$this->{'Acme\Model\Post'}` and an SQL alias with backslashes.

## Summary

| Configuration | Lookup | `Foo::class` | Example |
|---|---|---|---|
| `Error.handler`, `Exception.handler`, `Error.consoleHandler`, `Exception.consoleHandler` | Callable | Yes | `[Handler::class, 'handleError']` |
| `Exception.renderer` | As given | Yes | `Renderer::class` |
| `Exception.skipLog` | `instanceof` | Yes | `[NotFoundException::class, MyException::class]` |
| `Dispatcher.filters` | As given, or a callable | Yes | `[HeaderFilter::class => ['header' => 'X-Frame-Options']]` |
| Datasource `'datasource'` | As given | Yes | `'datasource' => MemorySource::class` |
| `Session.handler.engine` | As given | Yes | `MemoryHandler::class` |
| `'routeClass'`, `Router::defaultRouteClass()` | As given | Yes | `['routeClass' => SlugRoute::class]` |
| `Acl.classname` | As given | Yes | `NullAcl::class` |
| `'engine'` of `TextHelper`, `NumberHelper`, `TimeHelper` | As given | Yes | `['engine' => Text::class]` |
| `CakeEmail::$_configClass` (in a subclass) | As given | Yes | `EmailConfig::class` |
| `CakeLog` `'engine'` | Suffix `Log`, unless the name ends with it | Yes, if the class name ends with `Log` | `MemoryLog::class` |
| `Cache` `'engine'` | Suffix `Engine` | Without the suffix | `'Acme\Cache\Memory'` |
| `CakeEmail` `'transport'` | Suffix `Transport` | Without the suffix | `'Acme\Mail\Memory'` |
| `AuthComponent` `authenticate`, `authorize`; `passwordHasher` | Suffix `Authenticate`, `Authorize`, `PasswordHasher` | Without the suffix | `['Acme\Auth\Token']` |
| `Controller::$viewClass`, `RequestHandlerComponent::viewClassMap()` | Suffix `View` | Without the suffix | `'Acme\View\Csv'` |
| `$components`, `$helpers`, `$actsAs` | Suffix, and the name is the alias | Without the suffix, with `className` | `'Greeting' => ['className' => 'Acme\Controller\Component\Greeting']` |
| `ClassRegistry::init()`, association `'className'` | As given | Yes, with an alias and `$name` | `['class' => Post::class, 'alias' => 'Post']` |
| `Controller::$uses`, `loadModel()` | The name is the alias | No | Use `ClassRegistry::init()` |
| Controllers, shells, `App::uses()` | Global class names | No | |

## Callables and Class Names as Given

These settings accept `Foo::class`:

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

`Exception.renderer` is loaded in `ErrorHandler::handleException()`,
`Session.handler.engine` in `CakeSession::_getHandler()`, the datasource in
`ConnectionManager::loadDataSource()`, the route class in
`Router::_validateRouteClass()` and `Acl.classname` in
`AclComponent::__construct()`. All of them use the configured name unchanged.

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
`BaseAuthenticate::passwordHasher()` and `Controller::_getViewObject()` append
a suffix to the configured name. Name the class with the suffix, and configure
it without the suffix as a string:

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

`CakeLog` (`LogEngineCollection::_getLogger()`) appends `Log` only when the
name does not end with it, so `MemoryLog::class` works as well as
`'Acme\Log\Memory'`.

`RequestHandlerComponent::viewClassMap()` takes the same name without the
`View` suffix as `Controller::$viewClass`, for example
`$this->RequestHandler->viewClassMap('csv', 'Acme\View\Csv')` for
`Acme\View\CsvView`.

## Components, Helpers and Behaviors

`ComponentCollection::load()`, `HelperCollection::load()` and
`BehaviorCollection::load()` append `Component`, `Helper` or `Behavior`, and
use the name as the alias: the property of the controller or view, and the
name in `Behaviors->loaded()`. Give a short alias as the key and the
namespaced name without the suffix in `className`:

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

`$this->Greeting` is then an `Acme\Controller\Component\GreetingComponent`.
`GreetingComponent::class` in `className` does not work, because the suffix is
appended again.

## Models

`ClassRegistry::init()` and the `'className'` of an association use the class
name as given, so namespaced models work when they have a plain alias:

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

Set `$name` in the model. Without it, the name is the class name with its
namespace, and the default alias and table name are derived from it
(`acme\model\posts`).

`Controller::$uses` and `Controller::loadModel()` use the configured name as
the property and the model alias, so a namespaced name there gives
`$this->{'Acme\Model\Post'}` and queries with an `Acme\Model\Post` alias. Load
namespaced models with `ClassRegistry::init()` instead, for example in
`beforeFilter()`.

## What Does Not Work

- Controllers are found from the URL as `PostsController`
  (`Dispatcher::_loadController()`), and shells from the command line as
  `PostsShell`; they must be global classes.
- `App::uses()` does not load namespaced classes: `App::load()` would look for
  a file named after the whole name in the package. Composer loads them, so
  `App::uses()` is not needed.
