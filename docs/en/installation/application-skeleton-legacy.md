# Legacy Application Skeleton

[pieceofcake2/app-legacy](https://github.com/pieceofcake2/app-legacy) is the
application skeleton for CakePHP 2 applications whose classes are global and
whose templates are `.ctp` files, as in the original CakePHP 2. It has the same
directory layout as [pieceofcake2/app](application-skeleton), compatible with
CakePHP 5 (`config/`, `src/`, `templates/`, `tests/`, `webroot/`), the same
configuration and `.env` files, and the same home page, layouts, error pages
and styles. Only the form of the code differs:

| | pieceofcake2/app-legacy | [pieceofcake2/app](application-skeleton) |
| --- | --- | --- |
| Application classes | global (`AppController`) | in the `App` namespace (`App\Controller\AppController`) |
| Autoloading of `src/` | classmap: run `composer dump-autoload` after you add a class file | PSR-4 (`App\` from `src/`, `App\Test\` from `tests/`) |
| Templates | `.ctp` files in `Layouts/`, `Elements/`, `Emails/` and `Errors/` | `.php` files in `layout/`, `element/`, `email/` and `Error/` |
| Coding standard | `CakePHP2` | `CakePHP2Namespace` |

Use this skeleton when your code should look like the code of an existing
CakePHP 2 application, for example to move the application into the skeleton
step by step. You can move the classes into a namespace later (see
[Moving to Namespaced Classes](#moving-to-namespaced-classes)).

> [!WARNING]
> CakePHP 2.x is for the maintenance of existing applications. For new
> projects, use the current major version of CakePHP instead.

## Installation

Create a new application with Composer's `create-project` command:

```bash
composer create-project pieceofcake2/app-legacy my_app
```

The requirements, the installed packages and the configuration steps are the
same as for pieceofcake2/app; see [Requirements](application-skeleton#requirements)
and [Installation](application-skeleton#installation). In short: copy
`config/database.php.default` to `config/database.php` and set
`DATABASE_URL` in `config/.env.local` (and the test database in
`config/.env.test.local`), and change the security salts (see
[Generating the Security Salts](application-skeleton#generating-the-security-salts)).

The Bake and DebugKit plugins are development dependencies.
`config/bootstrap.php` loads them when they are installed, so `bin/cake bake`
works out of the box and `composer install --no-dev` still works in
production. To show the DebugKit toolbar, add the component in your
`AppController` as described in
[Development Plugins](application-skeleton#development-plugins), without the
`namespace` line.

## Directory Layout

```text
my_app/
├── bin/
│   └── cake            the console
├── config/             configuration (core.php, bootstrap.php, routes.php, database.php, ...)
├── logs/               log files
├── plugins/            plugins installed by Composer
├── resources/
│   └── locales/        translation files
├── src/                PHP code
│   ├── Console/        shells, tasks and bake templates
│   ├── Controller/
│   ├── Lib/
│   ├── Model/
│   └── View/           helpers
├── templates/          view templates (.ctp)
│   ├── Elements/       elements (Flash/ for the flash messages)
│   ├── Emails/         email templates (html/, text/)
│   ├── Errors/         error pages
│   ├── Layouts/        layouts (Emails/ for the email layouts)
│   └── Pages/          pages of PagesController
├── tests/
│   ├── Fixture/
│   └── TestCase/
├── tmp/                cache, sessions and test files
├── vendor/             Composer packages, including CakePHP
└── webroot/            document root (index.php, test.php, css/, js/, img/)
```

See [Directory Layout](application-skeleton#directory-layout) for how CakePHP
finds the directories, and
[CakePHP Folder Structure](/getting-started/cakephp-folder-structure).

## Classes and Autoloading

The classes of `src/` are global, and Composer loads them with a classmap:

```json
"autoload": {
    "classmap": ["src/"]
}
```

The classmap finds a class in any folder of `src/`, but only after
`composer dump-autoload`: run it every time you add a class file. The classes
of CakePHP are imported with `use` statements, as in pieceofcake2/app:

```php
// src/Controller/AppController.php
use Cake\Controller\Controller;

class AppController extends Controller
{
}
```

See [Autoloading](/development/autoloading).

## Templates

The templates are `.ctp` files with the directory names of CakePHP 2:

- `templates/Layouts/default.ctp`: the top navigation bar and the content
  container. While `debug` is 2, the SQL log is shown below the content.
- `templates/Layouts/error.ctp`: a minimal layout for the error pages in
  production.
- `templates/Layouts/flash.ctp`: the page rendered by `Controller::flash()`.
- `templates/Elements/Flash/`: `default`, `success`, `error`, `warning` and
  `info` flash messages.
- `templates/Pages/home.ctp`: the home page, shown at `/` while `debug` is
  enabled. Replace it with your own page before turning `debug` off.
- `templates/Errors/error400.ctp` and `error500.ctp`: the error pages.

They look the same as the templates of pieceofcake2/app; see
[Pages and Styles](application-skeleton#pages-and-styles) for the stylesheets
and the screenshots of the error pages and the flash messages.

![Home page](/skeleton-home.png)

To use the new look in an application created from an earlier version of the
skeleton, copy `templates/Layouts/`, `templates/Elements/Flash/`,
`templates/Errors/`, `webroot/css/`, `webroot/font/` and
`webroot/img/cake.logo.svg` into your application.

## Configuration and Development

Everything else is the same as in pieceofcake2/app:

- [Configuration](application-skeleton#configuration) and
  [Environment Variables (.env)](application-skeleton#environment-variables-env)
- [Database and Mailer DSNs](application-skeleton#database-and-mailer-dsns)
- [Running the Application](application-skeleton#running-the-application)
- [Running Tests](application-skeleton#running-tests) and
  [End-to-End Tests](application-skeleton#end-to-end-tests). The test cases in
  `tests/TestCase/` and the fixtures in `tests/Fixture/` are global classes.
- [Static Analysis](application-skeleton#static-analysis)

`vendor/bin/phpcs` checks the code against the `CakePHP2` standard of
[pieceofcake2/phpcs-cakephp2](https://pieceofcake2.github.io/phpcs-cakephp2/)
(see [A phpcs.xml for Your Application](/contributing/cakephp-coding-conventions#a-phpcs-xml-for-your-application)).

## Moving to Namespaced Classes

To move an application created from this skeleton to the form of
pieceofcake2/app, run the `layout` and `namespace` commands of
[pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/). In an
application that already has the modern layout, `layout` renames the
templates to `.php` files in the directories of CakePHP 5, and `namespace`
moves the classes into the `App` namespace. See
[Migrating to the Modern Layout](/appendices/migrating-to-the-modern-layout).

## Origin

This package is based on the `app/` directory of
[cakephp/cakephp](https://github.com/cakephp/cakephp) (`cakephp/cakephp` on
Packagist, last upstream version 2.10.24), originally by the CakePHP community
(Cake Software Foundation, Inc.). It was extracted into a separate package and
is maintained for CakePHP 2 on modern PHP by
[pieceofcake2](https://github.com/pieceofcake2). Changes from the original are
listed in the
[commit history](https://github.com/pieceofcake2/app-legacy/commits/2.x).
