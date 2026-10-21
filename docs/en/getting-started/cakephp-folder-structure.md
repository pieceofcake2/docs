# CakePHP Folder Structure

An application can use one of two layouts:

- The **modern layout** keeps `config/`, `src/`, `templates/`, `tests/` and the
  other folders at the project root, as in CakePHP 5. The
  [pieceofcake2/app](https://pieceofcake2.github.io/app/) skeleton uses it (see
  [Installation](../installation)).
- The **legacy `app/` layout** keeps the code in an `app/` folder. Applications
  created with older versions of CakePHP 2 use it.

CakePHP detects which layout an application uses. Both layouts work with the
same version of CakePHP.

In both layouts, Composer installs CakePHP itself and the third-party libraries
into `vendor/`. Make a personal commitment **not** to edit files in this
folder. We can't help you if you've modified the core. Instead, look into
modifying [Application Extensions](../getting-started/cakephp-structure#application-extensions).

## The Modern Layout {#the-modern-layout}

```text
my_app/
├── bin/
│   └── cake
├── config/
├── logs/
├── plugins/
├── resources/
│   └── locales/
├── src/
│   ├── Console/
│   ├── Controller/
│   ├── Lib/
│   ├── Model/
│   └── View/
│       └── Helper/
├── templates/
├── tests/
│   ├── Fixture/
│   └── TestCase/
├── tmp/
├── vendor/
├── webroot/
└── composer.json
```

You will mostly work in `src/`, `config/` and `templates/`.

bin  
Contains the `cake` console script. It runs the shells of CakePHP and of your
application, for example `bin/cake server` or `bin/cake test`. See
[Shells, Tasks & Console Tools](../console-and-shells).

config  
Holds the configuration files: the database connection, the bootstrap, the
core configuration and more. `config/define.php` defines the paths of the
application folders.

logs  
CakePHP writes its log files here. See [Logging](../core-libraries/logging).

plugins  
Contains the plugins. Composer installs plugins here.

resources/locales  
Stores the translations for internationalization.

src  
Contains the PHP classes of your application. Composer loads them; see
[Autoloading](../development/autoloading).

- `src/Console/Command/` contains the shells, and `Task/` the shell tasks.
  `src/Console/Templates/` can contain templates that change the output of
  bake.
- `src/Controller/` contains the controllers, and `Component/` the
  components.
- `src/Lib/` contains your own libraries. This keeps them apart from
  third-party libraries.
- `src/Model/` contains the models, `Behavior/` the behaviors and
  `Datasource/` the datasources.
- `src/View/Helper/` contains the helpers.

templates  
Contains the templates:

- views in a folder per controller, for example `templates/Posts/index.ctp`
- elements in `Elements/`
- error pages in `Errors/`
- layouts in `Layouts/`
- email templates in `Emails/`

::: pieceofcake2 Added in version 2.13.0
`.php` templates and the CakePHP 5 directories `layout/`, `element/`, `email/`
and `Error/`. See [Views](../views#templates-in-the-cakephp-5-style).
:::

tests  
Contains the tests. `tests/TestCase/` mirrors your application and contains
one or more test cases per class. `tests/Fixture/` contains the fixtures. See
[Testing](../development/testing).

tmp  
CakePHP stores temporary data here, such as model descriptions, cached views
and sometimes sessions. What it stores depends on your configuration.

Make sure that this folder exists and is writable. Otherwise the performance
of your application suffers badly. In debug mode, CakePHP warns you if the
folder is missing or not writable.

vendor  
Contains the packages that Composer installs, including CakePHP itself
(`vendor/pieceofcake2/cakephp`). Add libraries with `composer require`. Don't
edit or add files here by hand.

webroot  
The document root of your application, in development as well as in
production. Only this folder may be served by the web server. It also holds
the CSS stylesheets, images and JavaScript files.

## The Legacy `app/` Layout {#the-legacy-app-layout}

```text
my_app/
├── app/
│   ├── Config/
│   ├── Console/
│   │   ├── Command/
│   │   └── cake
│   ├── Controller/
│   ├── Lib/
│   ├── Locale/
│   ├── Model/
│   ├── Plugin/
│   ├── Test/
│   │   ├── Case/
│   │   └── Fixture/
│   ├── tmp/
│   │   └── logs/
│   ├── Vendor/
│   ├── View/
│   │   └── Helper/
│   └── webroot/
├── vendor/
└── composer.json
```

The folders hold the same things as in the modern layout:

app/Config  
The configuration files, like `config/`.

app/Console  
The shells and shell tasks in `Command/`, and the `cake` console script.

app/Controller, app/Lib, app/Model  
The controllers, libraries and models, like the folders of `src/`.

app/Locale  
The translations, like `resources/locales/`.

app/Plugin  
The plugins. CakePHP also looks for plugins in `plugins/` at the project root.

app/Test  
The test cases in `Case/` and the fixtures in `Fixture/`.

app/tmp  
The temporary data, and the log files in `logs/`.

app/Vendor  
Files of third-party libraries that are not installed with Composer.

app/View  
The templates, in the same folders as in `templates/`. `View/Helper/` holds
the helpers, so the templates and the helper classes share a folder.

app/webroot  
The document root.

## How the Layouts Correspond {#how-the-layouts-correspond}

| Legacy (`app/`)           | Modern                             |
|---------------------------|------------------------------------|
| `app/Config`              | `config/`                          |
| `app/Console`             | `src/Console/`                     |
| `app/Console/cake`        | `bin/cake`                         |
| `app/Controller`          | `src/Controller/`                  |
| `app/Lib`                 | `src/Lib/`                         |
| `app/Locale`              | `resources/locales/`               |
| `app/Model`               | `src/Model/`                       |
| `app/Plugin`              | `plugins/`                         |
| `app/Test/Case`, `app/Test/Fixture` | `tests/TestCase/`, `tests/Fixture/` |
| `app/tmp`                 | `tmp/`                             |
| `app/tmp/logs`            | `logs/`                            |
| `app/Vendor`              | `vendors/` (Composer packages are in `vendor/`) |
| `app/View` (`.ctp` files) | `templates/`                       |
| `app/View/Helper`         | `src/View/Helper/`                 |
| `app/webroot`             | `webroot/`                         |

You can move a legacy application to the modern layout gradually, while it
keeps running. See [Migrating to the Modern Layout](../appendices/migrating-to-the-modern-layout).
