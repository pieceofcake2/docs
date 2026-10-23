# Migrating to the Modern Layout

The Piece of Cake 2 fork supports two application layouts:

- **Legacy layout**: the traditional CakePHP 2.x layout with an `app/`
  directory.
- **Modern layout**: a CakePHP 5.x-style layout with `config/`, `src/`,
  `templates/`, `tests/` and so on at the project root. The
  [application skeletons](/installation/application-skeleton) use it.

Both layouts work with the same framework version; you can stay on the legacy
layout, or move to the modern one gradually. In the modern layout, you can
also move the templates to `.php` files in the directories of CakePHP 5
(`layout/`, `element/`, `email/`, `Error/`) and the application classes into
the `App` namespace, as in CakePHP 5. If you plan to upgrade to CakePHP 5 in
the future, these changes prepare your application while you stay on
CakePHP 2.

The `layout` and `namespace` commands of
[pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/) make these
changes (see [Migrating with pieceofcake2/upgrade](#migrating-with-pieceofcake2-upgrade)).
To keep the classes global, run only `layout`. The two skeletons show the
result:

- [pieceofcake2/app](/installation/application-skeleton): the modern layout,
  `.php` templates and classes in the `App` namespace.
- [pieceofcake2/app-legacy](/installation/application-skeleton-legacy): the
  modern layout with global classes and `.ctp` templates.

## Gradual Migration to CakePHP 5

Moving from CakePHP 2.x to 5.x in one step means changing code, folder
structure and APIs at the same time. The modern layout lets you split this into
smaller steps:

```text
Step 1: CakePHP 2.x with traditional structure
        ↓ (modernize folder structure only: upgrade layout)
Step 2: CakePHP 2.x with CakePHP 5.x-style structure  ← you can stop here
        ↓ (move the classes into a namespace: upgrade namespace)
Step 3: CakePHP 2.x with namespaced classes           ← or here
        ↓ (upgrade code only)
Step 4: CakePHP 5.x with CakePHP 5.x-style structure
```

- Smaller, manageable changes: folder restructuring is separate from code
  changes.
- Each step can be tested before moving on.
- You can stay on Step 2 or Step 3 indefinitely.
- Smaller changes are easier for teams to review.

## Legacy and Modern Layouts

::: code-group

```text [Legacy (app/)]
your-project/
├── app/
│   ├── Config/          (uppercase, nested)
│   ├── Controller/
│   ├── Model/
│   ├── View/            (templates + helpers mixed)
│   ├── Test/
│   ├── Plugin/
│   ├── Vendor/
│   └── tmp/
│       └── logs/
├── vendors/
└── ...
```

```text [Modern (src/)]
your-project/
├── bin/
│   └── cake
├── config/              (lowercase, top-level)
├── src/                 (all PHP code, App namespace)
│   ├── Controller/
│   ├── Model/
│   └── View/            (Helper classes only)
├── templates/           (all templates as .php files, separated)
│   ├── Error/
│   ├── element/
│   ├── email/
│   └── layout/
├── resources/
│   └── locales/
├── tests/               (lowercase, top-level)
├── plugins/             (Composer-managed)
├── vendor/              (standard Composer)
├── logs/                (separated from tmp/)
├── tmp/
└── webroot/
```

:::

With global classes and `.ctp` templates (Step 2, as in
[pieceofcake2/app-legacy](/installation/application-skeleton-legacy)), the
templates keep the directories of CakePHP 2 (`Layouts/`, `Elements/`,
`Emails/`, `Errors/`).

## File Migration Map {#file-migration-map}

| From (legacy) | To (modern) |
| --- | --- |
| `app/Config/*` | `config/*` |
| `app/Controller/*` | `src/Controller/*` |
| `app/Model/*` | `src/Model/*` |
| `app/View/**/*.ctp` | `templates/**/*.php` (or `templates/**/*.ctp`) |
| `app/View/Layouts/*` | `templates/layout/*` |
| `app/View/Layouts/Emails/*` | `templates/layout/email/*` |
| `app/View/Elements/*` | `templates/element/*` |
| `app/View/Emails/*` | `templates/email/*` |
| `app/View/Errors/*` | `templates/Error/*` |
| `app/View/Helper/*` | `src/View/Helper/*` |
| `app/Console/*` | `src/Console/*` |
| `app/Console/cake` | `bin/cake` |
| `app/Lib/<Type>/*` (e.g. `app/Lib/Error/`) | `src/<Type>/*` (`src/Error/`), with namespaced classes |
| `app/Lib/*` | `src/Lib/*` |
| `app/Locale/*` | `resources/locales/*` |
| `app/Test/*` | `tests/*` |
| `app/Plugin/*` | `plugins/*` (use Composer) |
| `app/Vendor/*` | `vendor/*` (use Composer) |
| `app/tmp/logs/*` | `logs/*` |
| `app/tmp/*` | `tmp/*` |
| `app/webroot/*` | `webroot/*` |

The `.ctp` templates can keep their names and the directories of CakePHP 2
(`templates/Layouts/`, `templates/Elements/`, ...). Since 2.13, CakePHP also
reads `.php` templates and the directories of CakePHP 5. The scaffold
templates (`Scaffolds/`) keep `.ctp`.

Inside `tests/`, the modern layout uses `tests/TestCase/` and `tests/Fixture/`
instead of `Test/Case/` and `Test/Fixture/`. The test suite and fixture loader
detect both.

## How the Framework Finds the Layout

The path constants are defined in the application's `config/define.php` and the
framework's own `config/define.php`. When a constant is not defined by the
application, the framework detects the layout:

- `CONFIG` is `ROOT/config/` if that directory exists, otherwise
  `ROOT/APP_DIR/Config/`.
- `TMP` is `ROOT/tmp/` if it exists, otherwise `APP/tmp/`.
- `LOGS` is `ROOT/logs/` when `ROOT/tmp/` exists, otherwise `TMP/logs/`.
- `TESTS` is `ROOT/tests/` if it exists, otherwise `APP/Test/`.
- Templates are also looked up in `ROOT/templates/`, and locales in
  `ROOT/resources/locales/`.

The modern skeleton's `config/define.php` sets, among others:

```php
define('ROOT', dirname(__DIR__));
define('APP_DIR', 'src'); // instead of 'app'
define('WEBROOT_DIR', 'webroot');
define('WWW_ROOT', ROOT . DS . WEBROOT_DIR . DS);
define('VENDORS', ROOT . DS . 'vendor' . DS);
```

`webroot/index.php`, `webroot/test.php` and `bin/cake` load this file. Your
existing application code (controllers, models, views) works without
modification in either layout, and the traditional file locations keep working
during the migration.

## Migrating with pieceofcake2/upgrade {#migrating-with-pieceofcake2-upgrade}

[pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/) moves the files
with `git mv` and rewrites the code with Rector. Commit your application before
you run it, and check and commit the result of each command. The changed code
needs pieceofcake2/cakephp 2.13 or later.

1. Install the plugin as a development dependency and load it in
   `app/Config/bootstrap.php` (see
   [Installation](https://pieceofcake2.github.io/upgrade/#installation) of
   pieceofcake2/upgrade):

   ```bash
   composer require --dev pieceofcake2/upgrade
   ```

   ```php
   CakePlugin::load('Upgrade');
   ```

2. Move the application to the modern layout, with the templates as `.php`
   files in the directories of CakePHP 5:

   ```bash
   app/Console/cake upgrade layout --dry-run
   app/Console/cake upgrade layout
   composer dump-autoload
   ```

3. Move the classes into the `App` namespace. The command also adds
   `App.namespace` to `config/core.php` and loads `App\` from `src/` and
   `App\Test\` from `tests/` with PSR-4:

   ```bash
   bin/cake upgrade namespace --dry-run
   bin/cake upgrade namespace
   composer dump-autoload
   ```

4. Do what the commands list for you to do by hand, run your tests, and review
   the changes with `git diff`.
5. Remove the plugin: `composer remove --dev pieceofcake2/upgrade`, and the
   `CakePlugin::load('Upgrade');` line, which is now in
   `config/bootstrap.php`.

See [Layout and Namespace](https://pieceofcake2.github.io/upgrade/#layout-and-namespace)
in the documentation of pieceofcake2/upgrade for what the commands change, the
applications without `app/`, and the plugins.

## Migrating by Hand

To migrate by hand, follow the steps below.

### 1. Update composer.json {#1-update-composer-json}

```json
{
    "require": {
        "php": ">=8.0",
        "pieceofcake2/cakephp": "^2.13"
    },
    "autoload": {
        "classmap": ["src/"]
    },
    "config": {
        "vendor-dir": "vendor/",
        "sort-packages": true,
        "allow-plugins": {
            "composer/installers": true
        }
    },
    "extra": {
        "installer-paths": {
            "plugins/{$name}/": ["type:cakephp-plugin"]
        }
    }
}
```

If your plugins were installed into `app/Plugin/`, the `installer-paths`
setting now points to `plugins/`. Remove the plugins from `app/Plugin/` and
run `composer install` to install them into `plugins/`.

### 2. Update the Dispatcher Files {#2-update-the-dispatcher-files}

Copy `config/define.php`, `webroot/index.php`, `webroot/test.php` and
`bin/cake` from [pieceofcake2/app](https://github.com/pieceofcake2/app) (or
[pieceofcake2/app-legacy](https://github.com/pieceofcake2/app-legacy) for
global classes) into your project. `config/define.php` sets the path constants
of the modern layout (see
[How the Framework Finds the Layout](#how-the-framework-finds-the-layout)),
and the other files load it.

### 3. Move the Files {#3-move-the-files}

Move the files according to the [migration map](#file-migration-map). This can
be done gradually, for example in this order:

1. Configuration: `app/Config/*` → `config/*`
2. PHP code: `app/Controller/*`, `app/Model/*` → `src/`
3. Templates: `app/View/**/*.ctp` → `templates/`. To use the names of
   CakePHP 5, rename the templates to `.php`, rename `Layouts/`, `Elements/`,
   `Emails/` and `Errors/` to `layout/`, `element/`, `email/` and `Error/`,
   and move the email layouts from `Layouts/Emails/` to `layout/email/`. The
   scaffold templates (`Scaffolds/`) keep `.ctp`.
4. Tests: `app/Test/*` → `tests/`
5. Dependencies: use Composer for plugins and vendors

### 4. Verify That the Application Still Works {#4-verify-that-the-application-still-works}

Run your tests and check the pages of the application.

With `"classmap": ["src/"]` and after `composer dump-autoload`, remove the
`App::uses()` calls from your controllers, models, helpers and shells.
`App::uses()` is deprecated as of 2.13.0.

```php
// Before
App::uses('AppController', 'Controller');

class UsersController extends AppController
{
    // ...
}
```

```php
// After (with Composer autoload)
class UsersController extends AppController
{
    // ...
}
```

::: warning
Run `composer dump-autoload` every time you create a new class file, until you
use PSR-4 in the next step. The classmap must be regenerated before new
classes can be found. Plugins still have to be loaded with
`CakePlugin::load()`. For a plugin that Composer does not autoload, add its
folder to the classmap.
:::

### 5. Move the Classes into a Namespace (Optional) {#5-move-the-classes-into-a-namespace-optional}

Declare the namespace that follows the directory of each class
(`App\Controller` for `src/Controller/`, `App\Test\TestCase\Controller` for
`tests/TestCase/Controller/`), and import the classes of CakePHP and of the
application with `use` statements:

```php
// src/Controller/UsersController.php
namespace App\Controller;

class UsersController extends AppController
{
    // ...
}
```

Tell CakePHP the namespace in `config/core.php`, before the `App.encoding`
setting (it must come before `Cache::config('_cake_core_')`):

```php
/**
 * The namespace of the classes of the application
 */
Configure::write('App.namespace', 'App');
```

Then load the classes with PSR-4 instead of the classmap, and run
`composer dump-autoload`:

```json
"autoload": {
    "psr-4": {
        "App\\": "src/"
    }
},
"autoload-dev": {
    "psr-4": {
        "App\\Test\\": "tests/"
    }
},
```

Namespaced code has no `Lib/`: move `src/Lib/<Type>/` to `src/<Type>/`, e.g.
`src/Error/AppExceptionRenderer.php` for `App\Error\AppExceptionRenderer`.
`config/` and the templates stay without a namespace and import the classes
that they name with `use` statements. Clear the caches in `tmp/cache/`, which
hold the old class names. See [Namespaced Classes](/development/namespaced-classes).

## Upgrading to CakePHP 5 (Step 4)

Once your application runs on CakePHP 2 with the modern layout, the upgrade to
CakePHP 5 is limited to code changes:

- The directory structure is already correct, so there is no need to
  reorganize files.
- The templates are already `.php` files in the directories of CakePHP 5, so
  there is no need to move them.
- The classes are already in the `App` namespace, as in CakePHP 5.
- Composer is already set up with `vendor/` and `plugins/`.

What is left: require CakePHP 5.x in `composer.json`, update the code for the
API changes, and fix compatibility issues. See the
[CakePHP 5.x Migration Guide](https://book.cakephp.org/5/en/appendices/5-0-migration-guide.html).
