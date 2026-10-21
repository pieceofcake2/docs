# App Class

`class` **App**

The app class is responsible for path management, class location and class loading.
Make sure you follow the [File And Classname Conventions](../getting-started/cakephp-conventions#file-and-classname-conventions).

## Packages

CakePHP is organized around the idea of packages, each class belongs to a
package or folder where other classes reside. You can configure each package
location in your application using `App::build('APackage/SubPackage', $paths)`
to inform the framework where should each class be loaded. Almost every class in
the CakePHP framework can be swapped with your own compatible implementation. If
you wish to use your own class instead of the classes the framework provides,
copy the class to `src/Cake/`, at the path of its namespace. For instance if
you'd like to use your own HttpSocket class, put it under:

    src/Cake/Network/Http/HttpSocket.php

Composer then loads your file instead of the file inside CakePHP, as described
in [Overriding classes in CakePHP](#overriding-classes-in-cakephp).

## Loading classes {#loading-classes}

Classes are lazily loaded in CakePHP. Composer loads the classes of CakePHP,
of your application and of the plugins installed with Composer, so you don't
need to declare them. The application skeleton (pieceofcake2/app) loads `src/`
with a classmap in its `composer.json`:

```json
"autoload": {
    "classmap": ["src/"]
}
```

With a classmap, run `composer dump-autoload` after adding a new class file.

CakePHP also finds the classes named in the configuration, such as
`$components`, `$helpers`, `$uses`, the associations and the cache engines, in
their packages, and the base classes `AppController`, `AppModel`, `AppHelper`
and `AppShell`.

`static` App::**uses**(string $class, string $package)

Return type: `void`

::: pieceofcake2 Deprecated in version 2.13.0
Let Composer load your classes instead
([#98](https://github.com/pieceofcake2/cakephp/pull/98)). The first call of
`App::uses()` in a process logs a warning with the file and line of the call,
and with `debug` above 0 also triggers `E_USER_DEPRECATED`. The call still
works. The framework does not call it.
:::

`App::uses()` tells App which package a class can be found in, so that App
can locate the file and load it the first time the class is used.

### Removing `App::uses()` calls {#removing-app-uses}

1. Let Composer load the classes:
   - In an application, add the classmap of `src/` shown above to its
     `composer.json`.
   - In a plugin, add a classmap or PSR-4 to the `composer.json` of the
     plugin. For a plugin without one, you can add its folder to the classmap
     of your application, e.g. `"classmap": ["src/", "plugins/MyPlugin/"]`.
2. Run `composer dump-autoload`.
3. Delete the `App::uses()` lines.

Some examples for common types of classes are:

Console Commands  
`App::uses('AppShell', 'Console/Command');`

Console Tasks  
`App::uses('ImportTask', 'Console/Command/Task');`

Controllers  
`App::uses('PostsController', 'Controller');`

Components  
`App::uses('AuthComponent', 'Controller/Component');`

Models  
`App::uses('MyModel', 'Model');`

Behaviors  
`App::uses('TreeBehavior', 'Model/Behavior');`

Views  
`App::uses('ThemeView', 'View');`

Helpers  
`App::uses('HtmlHelper', 'View/Helper');`

Libs  
`App::uses('PaymentProcessor', 'Lib');`

Vendors  
`App::uses('Textile', 'Vendor');`

Utilities  
`App::uses('CakeText', 'Utility');`

So basically the second param should simply match the folder path of the class file in core or app.

> [!NOTE]
> Third-party libraries are normally installed and autoloaded with Composer,
> so you don't need `App::uses()` or `App::import()` for them. For the few
> files that are not available through Composer and do not follow
> conventions, use `App::import()`.

### Finding a class name

`static` App::**className**(string $class, string $type = '', string $suffix = '')

Return type: `string|null`

::: pieceofcake2 Added in version 2.13.0
Backported from CakePHP 3 and later.
:::

Returns the name of the class that CakePHP uses for a name in its
configuration, or `null` when the class does not exist. CakePHP uses it for
components, helpers, behaviors, cache engines, email transports, view classes,
controllers, shells and models:

```php
App::className('Auth', 'Controller/Component', 'Component');
// 'AuthComponent'

App::className(MemoryEngine::class, 'Cache/Engine', 'Engine');
App::className('Acme\Cache\Engine\Memory', 'Cache/Engine', 'Engine');
// both 'Acme\Cache\Engine\MemoryEngine'

Configure::write('App.namespace', 'App');
App::className('Post', 'Model');
// 'App\Model\Post' if that class exists, else the global 'Post', else null
```

A name with a backslash is a fully qualified class name; the suffix is added
only when the name does not end with it. Other names are looked up in the
namespace of the plugin or of `App.namespace` as
`<Namespace>\<Type>\<Name><Suffix>`, then as the global class
`<Name><Suffix>`, loaded by Composer or from the `<Type>` package. See
[Namespaced Classes](/development/namespaced-classes).

### Loading files from plugins

Composer loads the classes of a plugin when the plugin's `composer.json`
has a classmap or PSR-4. Otherwise, `App::uses()` loads them the same way as
app classes, except you must specify the plugin you are loading from:

```php
// Load the class Comment in plugins/PluginName/Model/Comment.php
App::uses('Comment', 'PluginName.Model');

// Load the class CommentComponent in
// plugins/PluginName/Controller/Component/CommentComponent.php
App::uses('CommentComponent', 'PluginName.Controller/Component');
```

## Finding paths to packages using App::path()

`static` App::**path**(string $package, string $plugin = null)

Return type: `array`

Used to read information stored path:

```php
// return the model paths in your application
App::path('Model');
```

This can be done for all packages that are apart of your application. You
can also fetch paths for a plugin:

```php
// return the component paths in DebugKit
App::path('Component', 'DebugKit');
```

`static` App::**paths**( )

Return type: `array`

Get all the currently loaded paths from App. Useful for inspecting or
storing all paths App knows about. For a paths to a specific package
use `App::path()`

`static` App::**core**(string $package)

Return type: `array`

Used for finding the path to a package inside CakePHP:

```php
// Get the path to Cache engines.
App::core('Cache/Engine');
```

`static` App::**location**(string $className)

Return type: `string`

Returns the package name where a class was defined to be located at.

## Adding paths for App to find packages in

`static` App::**build**(array $paths = [], mixed $mode = App::PREPEND)

Return type: `void`

Sets up each package location on the file system. You can configure multiple
search paths for each package, those will be used to look for files one
folder at a time in the specified order. All paths must be terminated
with a directory separator.

Adding additional controller paths for example would alter where CakePHP
looks for controllers. This allows you to split your application up across
the filesystem.

Usage:

```php
//will setup a new search path for the Model package
App::build(['Model' => ['/a/full/path/to/models/']]);

//will setup the path as the only valid path for searching models
App::build(['Model' => ['/path/to/models/']], App::RESET);

//will setup multiple search paths for helpers
App::build([
    'View/Helper' => ['/path/to/helpers/', '/another/path/'],
]);
```

If reset is set to true, all loaded plugins will be forgotten and they will
be needed to be loaded again. Package names are the folder names, such as
`Controller` or `View/Helper`. `App::build()` does not merge app paths with
core paths.

<a id="app-build-register"></a>

### Add new packages to an application

`App::build()` can be used to add new package locations. This is useful
when you want to add new top level packages or, sub-packages to your
application:

```php
App::build([
    'Service' => ['%s' . 'Service' . DS],
], App::REGISTER);
```

The `%s` in newly registered packages will be replaced with the
`APP` path. You must include a trailing `/` in registered
packages. Once packages are registered, you can use `App::build()` to
append/prepend/reset paths like any other package.

::: info Changed in version 2.1
Registering packages was added in 2.1
:::

## Finding which objects CakePHP knows about

`static` App::**objects**(string $type, mixed $path = null, boolean $cache = true)

Return type: `mixed`. Returns an array of objects of the given type or false if incorrect.

You can find out which objects App knows about using
`App::objects('Controller')` for example to find which application controllers
App knows about.

Example usage:

```php
//returns ['DebugKit', 'Blog', 'User'];
App::objects('plugin');

//returns ['PagesController', 'BlogController'];
App::objects('Controller');
```

You can also search only within a plugin's objects by using the plugin dot syntax.

```php
// returns ['MyPluginPost', 'MyPluginComment'];
App::objects('MyPlugin.Model');
```

`App::objects()` returns the complete class names. It returns `[]` for empty
results or invalid types, and it does not return core objects
(`App::objects('core')` returns `[]`).

## Locating plugins

`static` App::**pluginPath**(string $plugin)

Return type: `string`

Plugins can be located with App as well. Using `App::pluginPath('DebugKit');`
for example, will give you the full path to the DebugKit plugin:

```php
$path = App::pluginPath('DebugKit');
```

## Locating themes

`static` App::**themePath**(string $theme)

Return type: `string`

Themes can be found `App::themePath('purple');`, would give the full path to the
`purple` theme.

<a id="app-import"></a>

## Including files with App::import()

`static` App::**import**(mixed $type = null, string $name = null, mixed $parent = true, array $search = [], string $file = null, boolean $return = false)

Return type: `boolean`

At first glance `App::import` seems complex, however in most use
cases only 2 arguments are required.

> [!NOTE]
> This method is equivalent to `require`'ing the file.
> It is important to realize that the class subsequently needs to be initialized.

```php
// The same as require('Controller/UsersController.php');
App::import('Controller', 'Users');

// We need to load the class
$Users = new UsersController();

// If we want the model associations, components, etc to be loaded
$Users->constructClasses();
```

Keep in mind:

- The method does not look for files recursively, it strictly uses the
  paths defined in `App::build()`.
- It cannot load core classes; they are autoloaded.
- Importing a non-existent file, supplying a wrong type or package name, or
  null values for `$name` and `$file` parameters will result in a false return
  value.
- Loading Vendor files does not look recursively in the vendors folder, and
  the file name is not converted to underscored form.

## Overriding classes in CakePHP

You can override almost every class in the framework, exceptions are the
`App` and `Configure` classes. Copy the file of the class from
`vendor/pieceofcake2/cakephp/src/` to the same path below `src/Cake/` of your
application, and change it there. Keep its namespace and the `class_alias()` at
the end of the file. Some examples to follow:

- To override the `Dispatcher` class, create `src/Cake/Routing/Dispatcher.php`
  (`Cake\Routing\Dispatcher`)
- To override the `CakeRoute` class, create `src/Cake/Routing/Route/CakeRoute.php`
  (`Cake\Routing\Route\CakeRoute`)
- To override the `Model` class, create `src/Cake/Model/Model.php`
  (`Cake\Model\Model`)

Composer loads your file with the classmap of `src/` of the application
skeleton, before it looks in the core. Exclude the core file with
`exclude-from-classmap`, so that `composer dump-autoload -o` does not warn about
an "Ambiguous class resolution", and run `composer dump-autoload`:

```json
"autoload": {
    "classmap": ["src/"],
    "exclude-from-classmap": [
        "vendor/pieceofcake2/cakephp/src/Routing/Dispatcher.php"
    ]
}
```

If your application loads `src/` only with PSR-4, add `"classmap": ["src/Cake/"]`.

::: pieceofcake2 Changed in version 2.13.0
The framework uses the namespaced names of its classes
([#92](https://github.com/pieceofcake2/cakephp/pull/92)), so a global class
registered with `App::uses()`, such as `Dispatcher` in
`src/Lib/Routing/Dispatcher.php`, no longer replaces the core class, and PHP
warns that the name is already in use. Move your overrides to `src/Cake/` as
shown above.
:::

## Loading Vendor Files

Libraries installed with Composer are autoloaded and need no extra code. This
section applies to files that you copy into your project by hand. Put them in
the `vendors/` directory of your project (`app/Vendor/` in the legacy layout),
which Composer does not manage.

Composer can still load the classes in `vendors/` and its subdirectories:
add the directory to the classmap in your `composer.json`, then run
`composer dump-autoload`:

```json
"autoload": {
    "classmap": ["src/", "vendors/"]
}
```

Your vendor files may not follow conventions, have a class that differs from
the file name or does not contain classes. You can load those files using
`App::import()`. The following examples illustrate how to load vendor
files from a number of path structures. These vendor files could be located in
any of the vendor folders.

To load `vendors/geshi.php`:

```php
App::import('Vendor', 'geshi');
```

> [!NOTE]
> The geshi file must be a lower-case file name as CakePHP will not
> find it otherwise.

To load `vendors/flickr/flickr.php`:

```php
App::import('Vendor', 'flickr', ['file' => 'flickr/flickr.php']);
```

To load `vendors/some.name.php`:

```php
App::import('Vendor', 'SomeName', ['file' => 'some.name.php']);
```

To load `vendors/services/well.named.php`:

```php
App::import(
    'Vendor',
    'WellNamed',
    ['file' => 'services' . DS . 'well.named.php'],
);
```

To load `plugins/Awesome/Vendor/services/well.named.php`:

```php
App::import(
    'Vendor',
    'Awesome.WellNamed',
    ['file' => 'services' . DS . 'well.named.php'],
);
```

To load `plugins/Awesome/Vendor/Folder/Foo.php`:

```php
App::import(
    'Vendor',
    'Awesome.Foo',
    ['file' => 'Folder' . DS . 'Foo.php'],
);
```

Vendor files in `src/Vendor/` are found as well.

To load `vendors/vendorName/libFile.php`:

```php
App::import(
    'Vendor',
    'aUniqueIdentifier',
    ['file' => 'vendorName' . DS . 'libFile.php'],
);
```

## App Init/Load/Shutdown Methods

`static` App::**init**( )

Return type: `void`

Initializes the cache for App, registers a shutdown function.

`static` App::**load**(string $className)

Return type: `boolean`

Method to handle the automatic class loading. It will look for each class'
package defined using `App::uses()` or found by `App::className()`, and with
this information it
will resolve the package name to a full path to load the class from. File
name for each class should follow the class name. For instance, if a class
is name `MyCustomClass` the file name should be `MyCustomClass.php`

`static` App::**shutdown**( )

Return type: `void`

Object destructor. Writes cache file if changes have been made to the
`$_map`.
