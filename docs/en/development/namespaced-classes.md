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

## The Classes of the Framework

::: pieceofcake2 Added in version 2.13.0
The classes of the framework are in the `Cake` namespace
([#92](https://github.com/pieceofcake2/cakephp/pull/92)).
:::

The classes of the framework are in the `Cake` namespace, following their
directories: `Cake\Controller\Controller`, `Cake\Network\CakeRequest`,
`Cake\Utility\Hash`. The global names of CakePHP 2 are aliases of the same
classes, so `Controller` and `Cake\Controller\Controller` can be used
interchangeably, and existing code does not change. The files are in `src/`
(`src/Controller/Controller.php`), loaded by Composer with PSR-4. In a namespaced class,
import the classes of the framework like any other class:

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

Where the configuration names a class of the framework, both the global name
and the namespaced name work, for example `'routeClass' => RedirectRoute::class`
or `Configure::write('Acl.classname', DbAcl::class)`.

> [!WARNING]
> `get_class()`, `::class` and `ReflectionClass::getName()` return the
> namespaced names, e.g. `Cake\Network\CakeRequest`, and the error log, the
> `Debugger` output and the debug error pages show them. A comparison such as
> `get_class($request) === 'CakeRequest'` is now false: compare with
> `instanceof`, or with `CakeRequest::class` in a file that imports the class.

> [!TIP]
> `bin/cake upgrade rector --rules namespaced-classes src tests` of
> [pieceofcake2/upgrade](https://github.com/pieceofcake2/upgrade) replaces the
> global names in your code by the namespaced names with `use` statements, and
> the global names compared with `get_class()` by `::class`.

## How CakePHP Looks Up a Class

CakePHP resolves a configured class in one of four ways:

- **Callable.** The value is passed to PHP as a callable, for example to
  `set_error_handler()`. Any callable works, including
  `[Acme\Error\Handler::class, 'handleError']`.
- **Class name as given.** CakePHP calls `class_exists()` and `new` with the
  name, and `class_exists()` lets Composer load it. `Foo::class` works.
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
- **Name used as an alias.** For components, helpers, behaviors, shell tasks
  and models, the name also becomes a property or a model alias. For a fully
  qualified class name, the alias is the class name without the namespace and
  the suffix: `GreetingComponent::class` gives `$this->Greeting`, and
  `Post::class` the alias `Post`.

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
| `$components`, `$helpers`, `$actsAs`, shell `$tasks` | `App::className()`, and the name is the alias | Yes, the alias is the short class name | `[GreetingComponent::class]` |
| `ClassRegistry::init()`, association keys, `'className'` and `'with'` | `App::className()`, and the name is the alias | Yes, the alias is the short class name | `ClassRegistry::init(Post::class)` |
| `Controller::$uses`, `loadModel()`, shell `$uses` | The name is the alias | Yes, the alias is the short class name | `public $uses = [Post::class];` |
| Controllers, shells | `App::className()` from the URL or the command line | Short names in `App.namespace` | `App\Controller\PostsController` for `/posts` |

## Callables and Class Names as Given

These settings accept `Foo::class`:

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

The name without the suffix, such as `'Acme\Cache\Engine\Memory'`, keeps working.
`CakeLog` (`LogEngineCollection::_getLogger()`) appends `Log` only when the
name does not end with it, so `MemoryLog::class` works as well as
`'Acme\Log\Engine\Memory'`.

`RequestHandlerComponent::viewClassMap()` takes the same names as
`Controller::$viewClass`, for example
`$this->RequestHandler->viewClassMap('csv', CsvView::class)`.

## Components, Helpers and Behaviors

`ComponentCollection::load()`, `HelperCollection::load()`,
`BehaviorCollection::load()` and `TaskCollection::load()` find the class with
`App::className()` and use the name as the alias: the property of the
controller, view or shell, and the name in `Behaviors->loaded()`. For a fully
qualified class name, the alias is the class name without the namespace and
the suffix:

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

`$this->Greeting` is then an `Acme\Controller\Component\GreetingComponent`,
`$this->Badge` in the view an `Acme\View\Helper\BadgeHelper`, and the
behavior is loaded as `Stamp`. Shell `$tasks` work the same way:
`ReportTask::class` gives `$this->Report`. The same applies to a class name
given to `load()`, such as `$this->Components->load(GreetingComponent::class)`.
`enable()`, `disable()`, `unload()`, `loaded()`, `enabled()` and `set()` also
take the fully qualified class name instead of the alias.

For another alias, give the alias as the key and the class in `className`, as
before: `'Hello' => ['className' => GreetingComponent::class]` gives
`$this->Hello`. With [`App.namespace`](#app-namespace), a short name finds
the class in your namespace: `public $components = ['Greeting'];`.

When the `$components`, `$helpers`, `$actsAs` or `$tasks` of a parent class
(`AppController`, `AppModel`, the `AppController` of a plugin, the parent
class of a shell) and of a subclass are merged, a fully qualified class name
as the key is replaced by its alias, with the class name in `className`. So
`'Greeting' => [...]` in `AppController` and `GreetingComponent::class => [...]`
in `PostsController` give one `Greeting` with the merged settings, as two short
names do.

When the class is not found, the error shows the fully qualified class name
with the suffix added only when the name does not end with it, for example
`Acme\Controller\Component\GreetingComponent`.

::: pieceofcake2 Changed in version 2.13.0
A fully qualified class name as the key gives an alias without the namespace
and the suffix ([#91](https://github.com/pieceofcake2/cakephp/pull/91)). In
CakePHP 2.12, the alias was the name with the backslashes. In code that must
also work there, give a short alias, as in
`'Greeting' => ['className' => GreetingComponent::class]`.
:::

## Models

`ClassRegistry::init()` and the `'className'` of an association find the class
with `App::className()`. A fully qualified class name without an alias gets the
class name without the namespace as its alias, as a short name does:

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

`Author::class` in `belongsTo` is the same as
`'Author' => ['className' => Author::class]`: an association with the alias
`Author`. For another alias, write `'Writer' => ['className' => Author::class]`
or `ClassRegistry::init(['class' => Post::class, 'alias' => 'Article'])`.
`'with' => PostsTag::class` of a hasAndBelongsToMany association is the same as
`'with' => 'PostsTag'`: the join model `PostsTag` is used to find, save,
validate and delete the data.

The name of the model is the class name without its namespace (`Post`), and
the table name is derived from it (`posts`).

::: pieceofcake2 Changed in version 2.13.0
In CakePHP 2.12, the name kept the namespace (`Acme\Model\Post`), which gave
the table `acme\model\posts`. Set `$name` in models that must also work there.
The same applies to `Controller::$name` and `Shell::$name`.
:::

`Controller::$uses`, `Controller::loadModel()` and shell `$uses` use the
configured name without the namespace as the property and the model alias:

```php
// src/Controller/PostsController.php
use Acme\Model\Post;

class PostsController extends AppController
{
    public $uses = [Post::class];
}
```

`$this->Post` is then an `Acme\Model\Post`, and `$this->modelClass` is
`Post`. `$this->loadModel(Post::class)` works the same way.

`ClassRegistry::getObject()`, `isKeySet()` and `removeObject()` also take a
fully qualified class name. They use the object stored under the class name
without the namespace when it is of that class:
`ClassRegistry::getObject(Post::class)` returns the model with the alias
`Post`. Otherwise they use the only object of
exactly that class, such as a `Post` created with the alias `Article`. When
there are several objects of the class, none is found.

::: pieceofcake2 Changed in version 2.13.0
A fully qualified class name in `$uses`, `loadModel()`, the keys of the
associations and `ClassRegistry::init()` without an alias gives an alias
without the namespace ([#91](https://github.com/pieceofcake2/cakephp/pull/91)).
In CakePHP 2.12, the alias was the name with the backslashes, as in
`$this->{'Acme\Model\Post'}`. In code that must also work there, give a
short alias, as in `['class' => Post::class, 'alias' => 'Post']` or
`'Author' => ['className' => Author::class]`.
:::

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
  with [`App::className()`](/core-utility-libraries/app).
