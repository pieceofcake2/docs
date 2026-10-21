# How To Create Plugins

As a working example from the [How To Use Plugins](../plugins/how-to-use-plugins) section, let's
begin to create a ContactManager plugin.

Plugins live in the `plugins/` directory at the root of your project. In an
application with the legacy `app/` layout, you can also use `app/Plugin/`.
The name of the plugin folder, **ContactManager**, must be the same as the
name of the plugin.

A plugin can use one of two layouts:

- the [legacy plugin layout](#the-legacy-plugin-layout) of CakePHP 2
- the [modern plugin layout](#the-modern-plugin-layout), the layout of the
  application and of CakePHP 5

CakePHP searches the directories of the modern layout first, and the
directories of the legacy layout after them. So a plugin can move to the
modern layout one directory at a time.

The [pieceofcake2/plugin](https://github.com/pieceofcake2/plugin) skeleton is a
good starting point for a new plugin.

## The Legacy Plugin Layout {#the-legacy-plugin-layout}

```text
plugins/
└── ContactManager/
    ├── Config/
    │   ├── bootstrap.php
    │   └── routes.php
    ├── Console/
    ├── Controller/
    │   └── Component/
    ├── Lib/
    ├── Locale/
    ├── Model/
    │   └── Behavior/
    ├── Test/
    │   ├── Case/
    │   └── Fixture/
    ├── Vendor/
    ├── View/
    │   ├── Helper/
    │   └── Layouts/
    ├── webroot/
    └── composer.json
```

- The classes are global classes, in `Controller/`, `Model/`, `View/Helper/`
  and so on. Composer loads them with a classmap in the plugin's
  `composer.json` (see [Publish Your Plugin](#publish-your-plugin)).
- The templates are in `View/`, next to `View/Helper/`.
- `Config/` holds `bootstrap.php`, `routes.php`, the configuration files read
  with `Configure::load('ContactManager.file')` and the schemas in
  `Config/Schema/`.
- `Locale/` holds the translations, and `Vendor/` the vendor files.
- The tests are in `Test/Case/` and `Test/Fixture/`, or in `tests/TestCase/`
  and `tests/Fixture/`.

## The Modern Plugin Layout {#the-modern-plugin-layout}

::: pieceofcake2 Added in version 2.13.0
The modern plugin layout ([#88](https://github.com/pieceofcake2/cakephp/pull/88)).
:::

```text
plugins/
└── ContactManager/
    ├── config/
    │   ├── bootstrap.php
    │   └── routes.php
    ├── resources/
    │   └── locales/
    ├── src/
    │   ├── Controller/
    │   │   └── Component/
    │   ├── Model/
    │   │   └── Behavior/
    │   └── View/
    │       └── Helper/
    ├── templates/
    │   ├── Contacts/
    │   └── Layouts/
    ├── tests/
    │   ├── Fixture/
    │   └── TestCase/
    ├── webroot/
    └── composer.json
```

- `src/` holds only namespaced classes, as in CakePHP 5. For example,
  `src/Controller/Component/ContactsComponent.php` holds
  `ContactManager\Controller\Component\ContactsComponent`.
  - Composer loads them with the PSR-4 entry of the plugin's `composer.json`
    (`"ContactManager\\": "src/"`).
  - Name them in settings with the plugin prefix (`'ContactManager.Contacts'`)
    or with `ContactsComponent::class`. See
    [Namespaced Classes](/development/namespaced-classes).
- The global classes of the plugin stay in `Controller/`, `Model/`,
  `View/Helper/` and so on, as in the legacy layout.
- `templates/` holds the templates. CakePHP searches it before `View/`.
- `config/` holds the same files as `Config/` of the legacy layout. When
  `config/` exists, CakePHP reads the configuration of the plugin from it
  instead of `Config/`.
- `resources/locales/` holds the translations. CakePHP searches it before
  `Locale/`.
- `vendor/` holds the vendor files. CakePHP searches it before `Vendor/`.

> [!NOTE]
> `App::uses()` does not load the namespaced classes in `src/`; Composer loads
> them. CakePHP does not search `src/` for templates, translations or vendor
> files either, so the view classes in `src/View/` are never rendered as
> templates.

> [!WARNING]
> A file in `templates/` or `resources/locales/` is used instead of the file of
> the same name in `View/` or `Locale/`. Before you add one of these
> directories, or `config/` or `vendor/`, check that the plugin does not
> already have it for another purpose. For example, a plugin checked out for
> development may have a Composer `vendor/` directory.

## Plugin Folders

A plugin looks a lot like a CakePHP application, and that's basically what it
is. You don't have to include the folders that you do not use. Some plugins
only define a Component and a Behavior, and can omit `View/` or `templates/`
completely.

> [!NOTE]
> If you want to be able to access your plugin with a URL, defining
> an AppController and AppModel for the plugin is required. These
> two special classes are named after the plugin, and extend the
> parent application's AppController and AppModel. Here's what they
> should look like for our ContactManager example:

```php
// In plugins/ContactManager/Controller/ContactManagerAppController.php
class ContactManagerAppController extends AppController
{
}
```

```php
// In plugins/ContactManager/Model/ContactManagerAppModel.php
class ContactManagerAppModel extends AppModel
{
}
```

CakePHP finds these classes without `App::uses()`:

- `AppController`, `AppModel`, `AppHelper` and `AppShell` of the application
- `ContactManagerAppController` and `ContactManagerAppModel` of the plugin

Let Composer load the other classes of the plugin. See
[Autoloading](/development/autoloading#plugins).

If you forgot to define these special classes, CakePHP will hand
you "Missing Controller" errors until you've done so.

Please note that the process of creating plugins can be greatly
simplified by using the [Bake plugin](https://pieceofcake2.github.io/bake/).

In order to bake a plugin please use the following command:

```bash
bin/cake bake plugin ContactManager
```

Now you can bake using the same conventions which apply to the rest
of your app. For example - baking controllers:

```bash
bin/cake bake controller Contacts --plugin ContactManager
```

Please refer to the chapter
[Code Generation with Bake](../console-and-shells/code-generation-with-bake) for
how to install and use Bake.

> [!WARNING]
> A plugin is not a namespace. Global classes of all plugins and of the
> application share one set of names. Two plugins cannot have global classes or
> files with the same name. Use unique names, for example with the plugin name
> as a prefix, or namespaced classes in `src/` of the
> [modern plugin layout](#the-modern-plugin-layout).

## Plugin Controllers

Controllers for our ContactManager plugin will be stored in
`plugins/ContactManager/Controller/`. Since the main thing we'll
be doing is managing contacts, we'll need a ContactsController for
this plugin.

So, we place our new ContactsController in
`plugins/ContactManager/Controller` and it looks like so:

```php
// In plugins/ContactManager/Controller/ContactsController.php

class ContactsController extends ContactManagerAppController
{
    public $uses = ['ContactManager.Contact'];

    public function index()
    {
        // ...
    }
}
```

> [!NOTE]
> This controller extends the plugin's AppController (called
> ContactManagerAppController) rather than the parent application's
> AppController.
>
> Also note how the name of the model is prefixed with the name of
> the plugin. This is required to differentiate between models in
> the plugin and models in the main application.
>
> In this case, the \$uses array would not be required as
> ContactManager.Contact would be the default model for this
> controller, however it is included to demonstrate how to
> properly prepend the plugin name.

If you want to access what we've got going thus far, visit
/contact_manager/contacts. You should get a "Missing Model" error
because we don't have a Contact model defined yet.

<a id="plugin-models"></a>

## Plugin Models

Models for the plugin are stored in `plugins/ContactManager/Model`.
We've already defined a ContactsController for this plugin, so let's
create the model for that controller, called Contact:

```php
// In plugins/ContactManager/Model/Contact.php

class Contact extends ContactManagerAppModel
{
}
```

Visiting /contact_manager/contacts now (given you've got a table in your
database called 'contacts') should give us a "Missing View" error.
Let's create that next.

> [!NOTE]
> If you need to reference a model within your plugin, you need to
> include the plugin name with the model name, separated with a dot.

For example:

```php
// In plugins/ContactManager/Model/Contact.php

class Contact extends ContactManagerAppModel
{
    public $hasMany = ['ContactManager.AltName'];
}
```

If you would prefer that the array keys for the association not
have the plugin prefix on them, use the alternative syntax:

```php
// In plugins/ContactManager/Model/Contact.php

class Contact extends ContactManagerAppModel
{
    public $hasMany = [
        'AltName' => [
            'className' => 'ContactManager.AltName',
        ],
    ];
}
```

## Plugin Views

Views behave exactly as they do in normal applications. Just place
them in the right folder inside of the `plugins/[PluginName]/View/`
folder (`templates/` in the modern layout). For our ContactManager plugin,
we'll need a view for our ContactsController::index() action, so let's
include that as well:

```html
<!-- plugins/ContactManager/View/Contacts/index.ctp: -->
<h1>Contacts</h1>
<p>Following is a sortable list of your contacts</p>
<!-- A sortable list of contacts would go here....-->
```

> [!NOTE]
> For information on how to use elements from a plugin, look up
> [View Elements](../views#view-elements)

### Overriding Plugin Views From Inside Your Application

You can override any plugin views from inside your app using
special paths. If you have a plugin called 'ContactManager' you
can override the view files of the plugin with more application
specific view logic by creating files using the following template
`templates/Plugin/[Plugin]/[Controller]/[view].ctp`
(`app/View/Plugin/...` in the legacy `app/` layout). For the
Contacts controller you could make the following file:

    templates/Plugin/ContactManager/Contacts/index.ctp

Creating this file, would allow you to override
`plugins/ContactManager/View/Contacts/index.ctp`.

<a id="plugin-assets"></a>

## Plugin Assets

A plugin's web assets (but not PHP files) can be served through the
plugin's 'webroot' directory, just like the main application's assets:

    plugins/ContactManager/webroot/
                                   css/
                                   js/
                                   img/
                                   pdf/

You may put any type of file in any directory, just like a regular
webroot.

But keep in mind that handling static assets, such as images, Javascript
and CSS files of plugins, through the Dispatcher is incredibly inefficient.
It is strongly recommended to symlink them for production.
For example like this:

```bash
ln -s ../../plugins/YourPlugin/webroot/css/yourplugin.css webroot/css/yourplugin.css
```

### Linking to Assets in Plugins

Simply prepend /plugin_name/ to the beginning of a request for an
asset within that plugin, and it will work as if the asset were
in your application's webroot.

For example, linking to `/contact_manager/js/some_file.js`
would serve the asset
`plugins/ContactManager/webroot/js/some_file.js`.

> [!NOTE]
> It is important to note the `/your_plugin/` prefix before the
> asset path. That makes the magic happen!

::: info Changed in version 2.1
Use `plugin syntax` to request assets. For example in your View:
:::

```php
echo $this->Html->css('ContactManager.style');
```

## Components, Helpers and Behaviors

A plugin can have Components, Helpers and Behaviors just like a
regular CakePHP application. You can even create plugins that
consist only of Components, Helpers or Behaviors which can be a
great way to build reusable components that can easily be
dropped into any project.

Building these components is exactly the same as building it within
a regular application, with no special naming convention.

Referring to your component from inside or outside of your plugin
requires only that you prefix the plugin name before the name of the
component. For example:

```php
// Component defined in 'ContactManager' plugin

class ExampleComponent extends Component
{
}

// Within your controllers:

public $components = ['ContactManager.Example'];
```

The same technique applies to Helpers and Behaviors.

> [!NOTE]
> As with the plugin's AppController and AppModel, a helper of the plugin
> can extend `AppHelper` without `App::uses()`.

## Registering DSN Schemes

Datasources and email transports can be configured with a DSN in the `'dsn'`
key (see [Database DSNs](/development/configuration#database-url) and
[Mailer DSN](/core-utility-libraries/email#mailer-dsn)). The scheme of the DSN
is looked up in a DSN class map, which a plugin can extend with its own
schemes, as with `setDsnClassMap()` in CakePHP 3.x and later.
`ConnectionManager` and `CakeEmail` each have one:

| Method | |
|--------|---|
| `setDsnClassMap($map)` | Adds schemes or replaces built-in ones. `null` removes a scheme. Schemes are case-insensitive |
| `getDsnClassMap()` | Returns the map, including the built-in schemes |
| `parseDsn($dsn, $config = [])` | Returns the configuration keys of a DSN |

::: pieceofcake2
The DSN class maps were added in v2.12.7.
:::

Register the schemes in the plugin's `Config/bootstrap.php`, which the
application loads with `CakePlugin::load('MyPlugin', ['bootstrap' => true])`:

```php
// plugins/MyPlugin/Config/bootstrap.php

// mydb://user:pass@host:1234/name uses the MyPlugin.Database/MyDb datasource
// with login, password, host, port and database
ConnectionManager::setDsnClassMap([
    'mydb' => 'MyPlugin.Database/MyDb',
]);

// Keep working with releases that have no DSN class maps
if (method_exists('CakeEmail', 'setDsnClassMap')) {
    CakeEmail::setDsnClassMap([
        // myapi+api://KEY@default?region=eu uses the MyPlugin.MyApi transport
        'myapi+api' => function (array $parts, array $config) {
            return [
                'transport' => 'MyPlugin.MyApi',
                'apiKey' => $parts['username'] ?? null,
                'region' => $parts['query']['region'] ?? 'eu',
            ];
        },
    ]);
}
```

A value of the map is either a class name or a callable:

- A **class name** in plugin dot syntax is set as the `datasource` or
  `transport`, and the parts of the DSN become the usual keys of that kind,
  followed by the query parameters, which win:

  | Map | Keys |
  |-----|------|
  | `ConnectionManager` | `login`, `password`, `host` (without the brackets of an IPv6 address), `port`, `database` (the path without the leading `/`) |
  | `CakeEmail` | `username`, `password`, `host` (the host `default` is left out, so the transport uses its default), `port` |

- A **callable** that is not a string, such as a closure or
  `[$class, 'method']`, is called with the parts of the DSN and the
  configuration without the `'dsn'` key, and returns the configuration keys,
  including the class. The parts are `scheme`, `username`, `password`, `host`,
  `port`, `path` (only those present, URL-decoded) and `query` (always an
  array). The callable decides which query parameters to use. A string is
  always a class name, never a function name.

The built-in schemes are entries of the same maps (class names for the
databases, callables for email), so a plugin can replace them, for example to
point `mysql` to a datasource of its own. A `datasource` or `transport` query
parameter in the DSN still overrides a class name, and allows a scheme that is
not registered at all.

Guard the calls with `method_exists()` when the plugin also supports releases
of `pieceofcake2/cakephp` without DSN class maps.

### When to Register the Schemes

`ConnectionManager` reads the map when a connection is created (the first
`getDataSource()` call, usually the first query), and `CakeEmail` when a
configuration is applied. A plugin bootstrap is early enough for
`config/database.php` and `config/email.php`.

## Expand Your Plugin

This example created a good start for a plugin, but there is a lot
more that you can do. As a general rule, anything you can do with your
application, you can do inside of a plugin instead.

Go ahead, require some third-party libraries with Composer, add some
new shells to the cake console, and don't forget to create test cases
so your plugin users can automatically test your plugin's functionality!

In our ContactManager example, we might create add/remove/edit/delete
actions in the ContactsController, implement validation in the Contact
model, and implement the functionality one might expect when managing
their contacts. It's up to you to decide what to implement in your
plugins. Just don't forget to share your code with the community so
that everyone can benefit from your awesome, reusable components!

## Plugin Tips

Once a plugin has been installed in `plugins/`, you can access it
at the URL /plugin_name/controller_name/action. In our ContactManager
plugin example, we'd access our ContactsController at
/contact_manager/contacts.

Some final tips on working with plugins in your CakePHP
applications:

- When you don't have a \[Plugin\]AppController and
  \[Plugin\]AppModel, you'll get missing Controller errors when trying
  to access a plugin controller.
- You can define your own layouts for plugins, inside
  `plugins/[Plugin]/View/Layouts` (`templates/Layouts` in the modern
  layout). Otherwise, plugins will use the
  layouts from the `templates/Layouts` folder by default.
- You can do inter-plugin communication by using
  `$this->requestAction('/plugin_name/controller_name/action');` in your
  controllers.
- If you use requestAction, make sure controller and model names
  are as unique as possible. Otherwise you might get PHP "redefined
  class ..." errors.
- When adding routes with extensions to your plugin, ensure you use
  `Router::setExtensions()` so you do not override application routing.

## Publish Your Plugin

To share your plugin, create a `composer.json` file and publish your plugin at
[packagist.org](https://packagist.org/). Then it can easily be used through
Composer. In `composer.json`:

- Set the package `type` to `cakephp-plugin`.
- Require `composer/installers`, so that the plugin is installed into the
  plugin directory of the application.
- Set `installer-name` to the folder name of the plugin.
- List the folders with the classes of the plugin in the classmap of
  `autoload`, so that Composer loads them.

```json
{
    "name": "foo-bar/cakephp-logging",
    "type": "cakephp-plugin",
    "require": {
        "php": ">=8.0",
        "composer/installers": "*",
        "pieceofcake2/cakephp": "^2.12"
    },
    "autoload": {
        "classmap": ["Controller/", "Lib/", "Model/", "View/"]
    },
    "extra": {
        "installer-name": "Logging"
    }
}
```

List only the folders that the plugin has: `composer dump-autoload` stops
with an error for a folder that does not exist. For the namespaced classes in
`src/` of the modern layout, add a PSR-4 entry. See [Autoloading](/development/autoloading#plugins).

Choose a semantically meaningful name for the package name. This should ideally
be prefixed with the dependency, in this case "cakephp" as the framework.
The vendor name will usually be your GitHub username.
Do **not** use the CakePHP namespace (cakephp) as this is reserved to CakePHP
owned plugins.
The convention is to use lowercase letters and dashes as separator.

So if you created a plugin "Logging" with your GitHub account "FooBar", a good
name would be `foo-bar/cakephp-logging`.
And the CakePHP owned "Localized" plugin can be found under `cakephp/localized`
respectively. Its PHP 8 compatible fork is maintained at
[pieceofcake2/localized](https://pieceofcake2.github.io/localized/).
