# Configuration

Configuring a CakePHP application is a piece of cake. After you
have installed CakePHP, creating a basic web application requires
only that you setup a database configuration.

There are, however, other optional configuration steps you can take
in order to take advantage of CakePHP flexible architecture. You
can easily add to the functionality inherited from the CakePHP
core, configure additional/different URL mappings (routes), and
define additional/different inflections.

The application's configuration files live in the `config/` directory of
your project:

- `config/core.php` - core settings such as debug level, error handling,
  sessions, security salts and the internal caches.
- `config/bootstrap.php` - application bootstrapping: cache configurations,
  log configurations, plugin loading, custom inflections.
- `config/database.php` - database connections (copy it from
  `config/database.php.default`).
- `config/routes.php` - URL routing.
- `config/email.php` - email transport configurations (copy it from
  `config/email.php.default`).
- `config/acl.php`, `config/acl.ini.php` - ACL settings.
- `config/Schema/` - schema files, e.g. for database sessions and i18n.

The `CONFIG` constant points to this directory. In the legacy layout the same
files are in `app/Config/`, and `CONFIG` points there when the project has no
top-level `config/` directory. See [CakePHP Folder Structure](/getting-started/cakephp-folder-structure)
for both layouts.

<div class="index">

database.php, database.php.default

</div>

<a id="database-configuration"></a>

## Database Configuration

CakePHP expects database configuration details to be in a file at
`config/database.php`. An example database configuration file can
be found at `config/database.php.default`. A finished
configuration should look something like this:

```php
// config/database.php
class DATABASE_CONFIG
{
    public $default = [
        'datasource' => 'Database/Mysql',
        'persistent' => false,
        'host' => 'localhost',
        'login' => 'cakephpuser',
        'password' => 'c4k3roxx!',
        'database' => 'my_cakephp_project',
        'prefix' => '',
        'encoding' => 'utf8mb4',
    ];
}
```

The \$default connection array is used unless another connection is
specified by the `$useDbConfig` property in a model. For example, if
my application has an additional legacy database in addition to the
default one, I could use it in my models by creating a new \$legacy
database connection array similar to the \$default array, and by
setting `public $useDbConfig = 'legacy';` in the appropriate models.

Fill out the key/value pairs in the configuration array to best
suit your needs.

A connection can also be set with one DSN (a URL) in the `'dsn'` key, for
example `'dsn' => env('DATABASE_URL')`. The `config/database.php.default` file
of the application skeleton reads `DATABASE_URL`, which is the test database
in the tests. See
[Database DSNs](#database-url) and
[Environment Variables](#environment-variables).

dsn (*optional*)  
A DSN such as `mysql://user:password@localhost/my_app?encoding=utf8mb4`
that sets the datasource and the connection keys in one string. See
[Database DSNs](#database-url). Added in v2.12.7.

datasource  
The name of the datasource this configuration array is for.
The built-in database drivers are Database/Mysql, Database/Postgres,
Database/Sqlite and Database/Sqlserver. All of them use PDO, so the
matching PDO extension (`pdo_mysql`, `pdo_pgsql`, `pdo_sqlite` or `pdo_sqlsrv`)
must be installed.
You can use `plugin syntax` to indicate plugin datasource to use.

persistent  
Whether or not to use a persistent connection to the database. The
Sqlserver datasource does not support persistent connections and throws
an exception if this is set to true.

host  
The database server's hostname (or IP address).

login  
The username for the account.

password  
The password for the account.

database  
The name of the database for this connection to use.

prefix (*optional*)  
The string that prefixes every table name in the database. If your
tables don't have prefixes, set this to an empty string.

port (*optional*)  
The TCP port or Unix socket used to connect to the server.
For SQL Server the port is appended to the host when connecting.

encoding  
Indicates the character set to use for the connection. This defaults to
the database's default encoding. If you wish to use UTF-8 encoding
with MySQL you must use 'utf8' or 'utf8mb4' without the hyphen.
The character set is passed in the PDO DSN when connecting
(`charset=...` for MySQL, `options='--client_encoding=...'` for PostgreSQL),
so no `SET NAMES` query is needed. For SQL Server, 'utf8' selects the
driver's UTF-8 encoding.

flags (*optional*)  
An array of PDO attributes passed to the PDO constructor. As of PHP 8.1,
the MySQL and SQLite PDO drivers return integer and float columns as PHP
integers and floats. If your application relies on the old behavior of
receiving strings, set `'flags' => [PDO::ATTR_STRINGIFY_FETCHES => true]`.

schema  
The schema to use. Used by PostgreSQL (defaults to `public`) and
SQL Server (defaults to `dbo`).

sslmode (*optional*)  
PostgreSQL only. The `sslmode` value added to the DSN, e.g. `'require'`.

options (*optional*)  
SQL Server only. An array of additional connection options that are added to
the DSN, e.g. `['TrustServerCertificate' => 'yes', 'Encrypt' => 'no']`.

::: pieceofcake2
The Sqlserver datasource targets SQL Server
2022 and newer. It reads the port from `port`, defaults `schema` to `dbo` and
accepts SSL/TLS settings in the `options` array. See the [Migrating from CakePHP 2.10](/appendices/migrating-from-cakephp-2-10).
:::

unix_socket  
Used by drivers that support it to connect via unix socket files. If you are
using PostgreSQL and want to use unix sockets, leave the host key blank.

ssl_key  
The file path to the SSL key file. Must be combined with `ssl_cert`.
(Only supported by MySQL.)

ssl_cert  
The file path to the SSL certificate file. Must be combined with `ssl_key`.
(Only supported by MySQL.)

ssl_ca  
The file path to the SSL certificate authority. (Only supported by MySQL.)

settings  
An array of key/value pairs that should be sent to the database server as
`SET` commands when the connection is created. This option is only
supported by the Mysql, Postgres, and Sqlserver datasources at this time.

::: info Changed in version 2.4
The `settings`, `ssl_key`, `ssl_cert` and `ssl_ca` keys was added in 2.4.
:::

> [!NOTE]
> The prefix setting is for tables, **not** models. For example, if
> you create a join table for your Apple and Flavor models, you name
> it prefix_apples_flavors (**not**
> prefix_apples_prefix_flavors), and set your prefix setting to
> 'prefix\_'.

At this point, you might want to take a look at the
[CakePHP Conventions](../getting-started/cakephp-conventions). The correct
naming for your tables (and the addition of some columns) can score
you some free functionality and help you avoid configuration. For
example, if you name your database table big_boxes, your model
BigBox, your controller BigBoxesController, everything just works
together automatically. By convention, use underscores, lower case,
and plural forms for your database table names - for example:
bakers, pastry_stores, and savory_cakes.

### Database DSNs {#database-url}

::: pieceofcake2
The `'dsn'` key was added in v2.12.7; earlier releases ignore it silently. It
is a backport of the DSN support of CakePHP 3.x and later, where the key is
called `url`, and uses the `DATABASE_URL` format of CakePHP, Doctrine and Symfony.
:::

A connection can be set with one DSN in the `'dsn'` key. This keeps the
credentials of each environment in one environment variable:

```php
// config/database.php
class DATABASE_CONFIG
{
    public $default = [
        'datasource' => 'Database/Mysql',
        'persistent' => false,
        'host' => 'localhost',
        'login' => 'user',
        'password' => 'password',
        'database' => 'my_app',
        'prefix' => '',
    ];

    public function __construct()
    {
        // Property defaults cannot call functions, so set the DSN here
        $this->default['dsn'] = env('DATABASE_URL');
    }
}
```

```dotenv
# config/.env
DATABASE_URL="mysql://my_app:secret@127.0.0.1:3306/my_app?encoding=utf8mb4"
```

The DSN is parsed when the connection is created (the first
`ConnectionManager::getDataSource()` call, usually the first query), and its
parts are mapped to the connection keys:

| DSN part | Connection key |
|----------|----------------|
| scheme `mysql` | `datasource` `Database/Mysql` |
| scheme `postgres`, `postgresql` or `pgsql` | `datasource` `Database/Postgres` |
| scheme `sqlite` | `datasource` `Database/Sqlite` |
| scheme `sqlserver`, `mssql` or `sqlsrv` | `datasource` `Database/Sqlserver` |
| user and password | `login` and `password` |
| host and port | `host` (without the brackets of an IPv6 address) and `port` |
| path, without the leading `/` | `database` |
| query parameters | keys of the same name |

For example:

```dotenv
# MySQL
DATABASE_URL="mysql://user:password@localhost:3306/my_app?encoding=utf8mb4"
# PostgreSQL
DATABASE_URL="postgres://user:password@localhost:5432/my_app?schema=public"
# SQLite in memory
DATABASE_URL="sqlite:///:memory:"
# SQLite file, relative to the working directory (three slashes)
DATABASE_URL="sqlite:///tmp/my_app.sqlite"
# SQLite file, absolute path (four slashes)
DATABASE_URL="sqlite:////var/db/my_app.sqlite"
# SQL Server
DATABASE_URL="sqlserver://user:password@localhost:1433/my_app?encoding=utf8&options[TrustServerCertificate]=yes"
```

- **Precedence**: The keys taken from the DSN replace the same keys of the
  array, and the other keys of the array are kept, as in CakePHP 3.x and later.
  The array therefore holds the defaults (`persistent`, `prefix`, `encoding`,
  ...) and the DSN decides the connection. A DSN without a password keeps the
  `password` of the array; write `user:@host` for an empty password.
- **Empty values**: A `'dsn'` that is `null`, `''` or `false` is ignored, so
  `env('DATABASE_URL')` falls back to the array when the variable is not set.
  The `'dsn'` key itself is removed from the configuration that the datasource
  gets.
- **Encoding**: The user, password, host and database are URL-decoded. Encode
  reserved characters of a password: `%40` for `@`, `%3F` for `?`, `%23` for
  `#` and `%25` for `%`.
- **Query parameters**: They become keys as they are, e.g. `encoding`,
  `schema`, `prefix`, `persistent=true`, `flags[1002]=SET%20NAMES%20utf8mb4`
  or `options[Encrypt]=no`. The values `true`, `false` and `null` become the
  PHP values. `datasource=MyPlugin.Database/Custom` overrides the datasource
  of the scheme and also allows a scheme that is not registered. `1002` in
  `flags[1002]` is the value of `PDO::MYSQL_ATTR_INIT_COMMAND`; a DSN cannot
  contain constant names.
- **IPv6**: Write the host in brackets, e.g.
  `mysql://user:password@[::1]:3306/my_app`.
- **Errors**: An invalid DSN or an unknown scheme throws a `CakeException` that
  names the connection. The message does not contain the user name or
  password.
- **Schemes of plugins**: A plugin can add schemes, or replace the built-in
  ones, with `ConnectionManager::setDsnClassMap()`. See
  [Registering DSN Schemes](/plugins/how-to-create-plugins#registering-dsn-schemes).

The `'dsn'` key also works with `ConnectionManager::create()`, and
`ConnectionManager::parseDsn()` returns the keys of a DSN:

```php
ConnectionManager::create('reporting', ['dsn' => env('REPORTING_DATABASE_URL')]);

ConnectionManager::parseDsn('postgres://user:secret@db:5432/app');
// ['login' => 'user', 'password' => 'secret', 'host' => 'db', 'port' => '5432',
//  'database' => 'app', 'datasource' => 'Database/Postgres']
```

## Environment Variables

Settings that differ between environments, such as the debug level, the
security salts and the database credentials, can be read from environment
variables with `env()` instead of being written into the configuration files.
In production, set the variables in the configuration of the web server,
container or process manager. For local development, the application
skeleton can load them from a `.env` file.

### Loading .env Files {#loading-env-files}

Applications created from the
[pieceofcake2/app](https://pieceofcake2.github.io/app/) skeleton load
environment variables from `.env` files at the top of `config/core.php`, using
[josegonzalez/dotenv](https://github.com/josegonzalez/php-dotenv). Because
`config/core.php` is read first, the variables are available to the rest of
the configuration, the console and the tests.

This follows the `.env` support of the CakePHP 3+/5 application skeleton,
backported for CakePHP 2: [cakephp/app](https://github.com/cakephp/app) loads
`config/.env` with the same library in `config/bootstrap.php`. As in CakePHP 5,
the framework itself does not load `.env` files; the application does.

::: pieceofcake2
`.env` loading is part of the `pieceofcake2/app` skeleton since v2.4.0. For
other applications, see
[Adding .env Support to an Existing Application](#adding-env-support-to-an-existing-application).
:::

The skeleton loads these files if they exist, in the order of Symfony's
`.env` files:

| File | Loaded | In Git |
|------|--------|--------|
| `config/.env.{APP_ENV}.local` | when `APP_ENV` is set | ignored |
| `config/.env.{APP_ENV}`, e.g. `config/.env.test` | when `APP_ENV` is set | committed |
| `config/.env.local` | always | ignored |
| `config/.env`, the location used by the CakePHP 3+/5 application skeleton | always | ignored |
| `.env` in the project root | always | ignored |

A variable that is already set is never overwritten, so an earlier file wins
over a later one, and real environment variables (web server, shell,
container, CI) win over all files. `APP_ENV` is read from the real
environment. The tests set it to `test` in `tests/bootstrap.php`, so they load
`config/.env.test`, which is committed and points `DATABASE_URL` to the test
database (see
[Testing](/development/testing#the-test-database-of-the-skeleton)).
Put values for your machine only in `config/.env.local` or
`config/.env.test.local`.

::: pieceofcake2
Earlier skeletons load only `config/.env` and the root `.env`.
:::

The loaded values are written to `putenv()`, `$_ENV` and `$_SERVER` as
strings, so you can read them with `env()`, `getenv()`, `$_ENV` or `$_SERVER`:

```php
// config/core.php
Configure::write('Security.salt', env('SECURITY_SALT') ?: 'a-default-salt');
```

To start, copy the example file that ships with the skeleton and edit it:

```bash
cp config/.env.example config/.env
```

`config/.env`, `config/.env.local`, `config/.env.*.local` and `/.env` are
listed in the skeleton's `.gitignore`. Keep `.env` files with credentials out
of version control and commit an example file, such as `config/.env.example`,
instead. `config/.env.test` holds only the settings of the tests and is
committed, as in Symfony.

A `.env` file is meant for development. In production, set real environment
variables in the web server, container or process manager configuration; they
always take precedence over the files.

### .env File Syntax

A `.env` file contains one `NAME="value"` assignment per line, the common
dotenv style also used by Symfony and Laravel. Lines starting with `#` are
comments. The parser also accepts an `export ` prefix, but it is not needed:

```dotenv
# config/.env
DEBUG="2"
SECURITY_SALT="a-long-random-string"
DATABASE_URL="mysql://my_app:secret@127.0.0.1:3306/my_app"
```

Always quote the values. josegonzalez/dotenv converts unquoted values to
booleans, numbers or `null`: `007` becomes `7`, and a long digit string such
as a cipher seed overflows to a different number, and in an unquoted value a
`#` after a space starts a comment. Quoted values are kept as they are.
Inside double quotes, `${NAME}` is replaced with the value of a variable
defined earlier in the file.

### Adding .env Support to an Existing Application

Applications that use the legacy `app/` layout, or that were created from an
earlier version of the skeleton, can add the same loader. Install the library:

```bash
composer require josegonzalez/dotenv
```

Then add the following code at the top of `app/Config/core.php`
(`config/core.php` in the modern layout): the `use` statement after the
opening file comment, and the loop before the first `Configure::write()` call.
In the legacy layout, `CONFIG` is `app/Config/` and `ROOT` is the directory
that contains `app/`, so the files are `app/Config/.env` and `.env` in the
project root.

```php
// app/Config/core.php

use josegonzalez\Dotenv\Loader;

$appEnv = (string)env('APP_ENV');
$dotenvFiles = [CONFIG . '.env.local', CONFIG . '.env', ROOT . DS . '.env'];
if ($appEnv !== '') {
    array_unshift($dotenvFiles, CONFIG . ".env.{$appEnv}.local", CONFIG . ".env.{$appEnv}");
}
foreach ($dotenvFiles as $dotenvFile) {
    if (!is_file($dotenvFile)) {
        continue;
    }
    $dotenv = new Loader([$dotenvFile]);
    foreach ($dotenv->parse()->toArray() as $key => $value) {
        if (getenv($key) !== false || isset($_ENV[$key]) || isset($_SERVER[$key])) {
            continue;
        }
        // The parser converts some values to booleans, numbers or null; keep them as strings.
        if (is_bool($value)) {
            $value = $value ? 'true' : 'false';
        }
        $value = (string)$value;
        putenv($key . '=' . $value);
        $_ENV[$key] = $value;
        $_SERVER[$key] = $value;
    }
    unset($dotenv, $key, $value);
}
unset($appEnv, $dotenvFiles, $dotenvFile);
```

Add the `.env` files to `.gitignore`, and read the variables in your
configuration with `env()`, as shown in the next section.

### Environment Variables of the Application Skeleton

The `config/core.php`, `config/bootstrap.php`, `config/database.php.default`
and `config/email.php.default` files of the `pieceofcake2/app` skeleton read
the following variables. Its `config/.env.example` file lists them.

| Variable | Setting | When not set |
|----------|---------|--------------|
| `DEBUG` | `debug`: `0`, `1` or `2` (`true` is 2, `false` is 0) | `2` |
| `SECURITY_SALT` | `Security.salt` | the value in `config/core.php` |
| `SECURITY_CIPHER_SEED` | `Security.cipherSeed` (1 to 4294967295; see [cipherSeed](#cipher-seed)) | the value in `config/core.php` |
| `COOKIE_KEY` | `Cookie.key`, the `CookieComponent` key for the `'aes'` type (see [Migrating to 'aes'](#migrating-to-aes)) | `null`: `CookieComponent` uses `Security.salt` |
| `DATABASE_URL` | the `'dsn'` of the `default` connection, and with `APP_ENV=test` also of the `test` connection (see [Database DSNs](#database-url)) | the values in `config/database.php` |
| `MAILER_DSN` | the `'dsn'` of the `default` email profile (see [Mailer DSN](../core-utility-libraries/email#mailer-dsn)) | the values in `config/email.php` |
| `APP_ENV` | loads `config/.env.{APP_ENV}` and `config/.env.{APP_ENV}.local`; `test` in `tests/bootstrap.php` | only the other `.env` files are loaded |

Empty variables are treated as not set. Without any `.env` file or environment
variable, the application uses the values in the configuration files. The keys
of a DSN replace the same keys of the configuration, and the other keys are
kept as defaults. The `'dsn'` key needs `pieceofcake2/cakephp` 2.12.7 or later.

If your `config/database.php` or `config/email.php` was created from an
earlier skeleton, copy the `__construct()` method of
`config/database.php.default` or `config/email.php.default` into it, and
remove an `APP_ENV` branch that used the `test` connection as `default`
(`config/.env.test` sets `DATABASE_URL` to the test database instead). Copy
the `App.env` line of `config/core.php` and `tests/bootstrap.php` from the
skeleton too. Earlier
skeletons read the `DB_DATASOURCE`, `DB_HOST`, `DB_PORT`, `DB_USERNAME`,
`DB_PASSWORD` and `DB_DATABASE` variables (and `DB_TEST_*` for the `test`
connection) with an `applyEnv()` method; one `DATABASE_URL` replaces them:

```dotenv
# Before
DB_HOST="127.0.0.1"
DB_USERNAME="my_app"
DB_PASSWORD="secret"
DB_DATABASE="my_app"
# After
DATABASE_URL="mysql://my_app:secret@127.0.0.1/my_app"
```

For example, a `config/.env` file for development:

```dotenv
# config/.env
DEBUG="2"
DATABASE_URL="mysql://my_app:secret@127.0.0.1:3306/my_app?encoding=utf8mb4"
MAILER_DSN="smtp://127.0.0.1:1025"
```

and a `config/.env.test.local` file with the test database of your machine,
which wins over `config/.env.test` and `config/.env` in the tests:

```dotenv
# config/.env.test.local
DATABASE_URL="mysql://my_app:secret@127.0.0.1:3306/test_my_app?encoding=utf8mb4"
```

## Class Loading and Additional Class Paths

CakePHP framework classes are loaded by Composer's autoloader, so you never
need `App::uses()` for them. Your own classes can be autoloaded the same way by
adding a classmap to your application's `composer.json`, as the
[pieceofcake2/app](https://pieceofcake2.github.io/app/) skeleton does:

```json
"autoload": {
    "classmap": ["src/"]
}
```

With this in place, `App::uses()` calls for your controllers, models,
helpers and other classes in `src/` can be removed. Run
`composer dump-autoload` every time you add a new class file, so that the
classmap includes it. Plugins that do not support Composer autoloading still
need `App::uses()` (for example `App::uses('MyPluginHelper', 'MyPlugin.View/Helper')`).

::: pieceofcake2
The framework is loaded through Composer's
classmap instead of include paths and its own `App::uses()` calls were removed.
Application code that still uses `App::uses()` keeps working.
:::

`App::uses()` and `App::build()` are still available. When `App::uses()` is
used, CakePHP looks for classes in the application directory (`src/`, or
`app/` in the legacy layout). Templates are also found in `templates/`,
locales in `resources/locales/` and plugins in `plugins/`.

It's occasionally useful to be able to share MVC classes between
applications on the same system. If you want the same controller in
both applications, you can use CakePHP's `config/bootstrap.php` to bring
these additional classes into view.

By using `App::build()` in `config/bootstrap.php` we can define additional
paths where CakePHP will look for classes:

```php
App::build([
    'Model' => [
        '/path/to/models',
        '/next/path/to/models',
    ],
    'Model/Behavior' => [
        '/path/to/behaviors',
        '/next/path/to/behaviors',
    ],
    'Model/Datasource' => [
        '/path/to/datasources',
        '/next/path/to/datasources',
    ],
    'Model/Datasource/Database' => [
        '/path/to/databases',
        '/next/path/to/database',
    ],
    'Model/Datasource/Session' => [
        '/path/to/sessions',
        '/next/path/to/sessions',
    ],
    'Controller' => [
        '/path/to/controllers',
        '/next/path/to/controllers',
    ],
    'Controller/Component' => [
        '/path/to/components',
        '/next/path/to/components',
    ],
    'Controller/Component/Auth' => [
        '/path/to/auths',
        '/next/path/to/auths',
    ],
    'Controller/Component/Acl' => [
        '/path/to/acls',
        '/next/path/to/acls',
    ],
    'View' => [
        '/path/to/views',
        '/next/path/to/views',
    ],
    'View/Helper' => [
        '/path/to/helpers',
        '/next/path/to/helpers',
    ],
    'Console' => [
        '/path/to/consoles',
        '/next/path/to/consoles',
    ],
    'Console/Command' => [
        '/path/to/commands',
        '/next/path/to/commands',
    ],
    'Console/Command/Task' => [
        '/path/to/tasks',
        '/next/path/to/tasks',
    ],
    'Lib' => [
        '/path/to/libs',
        '/next/path/to/libs',
    ],
    'Locale' => [
        '/path/to/locales',
        '/next/path/to/locales',
    ],
    'Vendor' => [
        '/path/to/vendors',
        '/next/path/to/vendors',
    ],
    'Plugin' => [
        '/path/to/plugins',
        '/next/path/to/plugins',
    ],
]);
```

> [!NOTE]
> All additional path configuration should be done at the top of your application's
> `bootstrap.php`. This will ensure that the paths are available for the rest of your
> application.

<div class="index">

core.php, configuration

</div>

## Core Configuration

Each application in CakePHP contains a configuration file,
`config/core.php`, to determine CakePHP's internal behavior.
This file is a collection of Configure class
variable definitions and constant definitions that determine how
your application behaves. Before we dive into those particular
variables, you'll need to be familiar with `Configure`, CakePHP's
configuration registry class.

### CakePHP Core Configuration

The `Configure` class is used to manage a set of core CakePHP
configuration variables. These variables can be found in
`config/core.php`. Below is a description of each variable and
how it affects your CakePHP application.

debug  
Changes CakePHP debugging output.

- 0 = Production mode. No output.
- 1 = Show errors and warnings.
- 2 = Show errors, warnings, and SQL. \[SQL log is only shown when you
  add \$this-\>element('sql_dump') to your view or layout.\]

The application skeleton reads the debug level from the `DEBUG` environment
variable. See [Environment Variables](#environment-variables).

Error  
Configure the Error handler used to handle errors for your application.
By default `ErrorHandler::handleError()` is used. It will display
errors using `Debugger`, when debug \> 0
and log errors with `CakeLog` when debug = 0.

Sub-keys:

- `handler` - callback - The callback to handle errors. You can set this to any
  callback type, including anonymous functions.
- `level` - int - The level of errors you are interested in capturing.
- `trace` - boolean - Include stack traces for errors in log files.

Exception  
Configure the Exception handler used for uncaught exceptions. By default,
ErrorHandler::handleException() is used. It will display a HTML page for
the exception, and while debug \> 0, framework errors like
Missing Controller will be displayed. When debug = 0,
framework errors will be coerced into generic HTTP errors.
For more information on Exception handling, see the [Exceptions](exceptions)
section.

<div id="core-configuration-baseurl">

App.baseUrl  
If you don't want or can't get mod_rewrite (or some other
compatible module) up and running on your server, you'll need to
use CakePHP's built-in pretty URLs. In `config/core.php`,
uncomment the line that looks like:

```php
Configure::write('App.baseUrl', env('SCRIPT_NAME'));
```

Also remove the `webroot/.htaccess` file. In the legacy layout, remove
these `.htaccess` files:

    /.htaccess
    /app/.htaccess
    /app/webroot/.htaccess

Keep the document root set to `webroot/` (`app/webroot/`). Without the
`.htaccess` files, a document root that points to the project directory
exposes the configuration, logs and source code; see
[Deployment](../deployment#set-document-root).

This will make your URLs look like
www.example.com/index.php/controllername/actionname/param rather
than www.example.com/controllername/actionname/param.

If you are installing CakePHP on a webserver besides Apache, you
can find instructions for getting URL rewriting working for other
servers under the [Web Server Configuration](../installation/url-rewriting) section.

App.fullBaseUrl  
The fully qualified domain name (including protocol) to your application’s root.
To configure CakePHP to use a particular domain URL for any URL
generation inside the application, set this configuration
variable. This will override the automatic detection of full
base URL and can be useful when generating links from the CLI
(e.g. sending emails). If the application runs in a subfolder,
you should also set `App.base`.

App.base  
The base directory the app resides in. Should be used if the application runs
in a subfolder and `App.fullBaseUrl` is set.

App.encoding  
Define what encoding your application uses. This encoding
is used to generate the charset in the layout, and encode entities.
It should match the encoding values specified for your database.

Routing.prefixes  
Un-comment this definition if you'd like to take advantage of
CakePHP prefixed routes like admin. Set this variable with an array
of prefix names of the routes you'd like to use. More on this
later.

Cache.disable  
When set to true, persistent caching is disabled site-wide.
This will make all read/writes to `Cache` fail.

Cache.check  
If set to true, enables view caching. Enabling is still needed in
the controllers, but this variable enables the detection of those
settings.

Session  
Contains an array of settings to use for session configuration. The defaults key is
used to define a default preset to use for sessions, any settings declared here will override
the settings of the default config.

Sub-keys

- `cookie` - The name of the cookie to use. Defaults to 'CAKEPHP'
- `timeout` - The number of minutes you want sessions to live for.
  This timeout is handled by CakePHP
- `cookieTimeout` - The number of minutes you want session cookies to live for.
- `checkAgent` - Do you want the user agent to be checked when starting sessions?
  You might want to set the value to false for clients that change their user
  agent string during a session.
- `defaults` - The default configuration set to use as a basis for your session.
  There are four builtins: php, cake, cache, database.
- `handler` - Can be used to enable a custom session handler.
  Expects an array of callables, that can be used with `session_save_handler`,
  or an array with an `engine` key naming a class that implements
  `CakeSessionHandlerInterface`.
  Using this option will automatically add `session.save_handler` to the ini array.
- `autoRegenerate` - Enabling this setting, turns on automatic renewal
  of sessions, and sessionids that change frequently.
  See `CakeSession::$requestCountdown`.
- `cacheLimiter` - The cache control headers used for the session cookie.
  See [session_cache_limiter()](https://www.php.net/session_cache_limiter) for
  accepted values.
- `ini` - An associative array of additional ini values to set.

The built-in defaults are:

- 'php' - Uses settings defined in your `php.ini`.
- 'cake' - Saves session files in CakePHP's `tmp/sessions` directory.
- 'database' - Uses CakePHP's database sessions.
- 'cache' - Use the Cache class to save sessions.

To define a custom session handler, save it at `src/Model/Datasource/Session/<name>.php`.
Make sure the class implements `CakeSessionHandlerInterface`
and set `Session.handler.engine` to \<name\>.

To use database sessions, create the table from the `config/Schema/sessions.php`
schema using the cake shell command: `bin/cake schema create Sessions`

See [Sessions](sessions) for more details.

Security.salt  
A random string used in security hashing. It is also used to sign CSRF
tokens and form tampering tokens, and as the default key for
`CookieComponent`. Change it to a secret value unique to your application;
CakePHP shows a notice while the default value from the skeleton is in use.
The skeleton reads it from the `SECURITY_SALT` environment variable when it
is set. See [Generating the Security Salts](#security-salts) for how to
generate it and what changing it affects.

::: pieceofcake2
CSRF tokens issued by `SecurityComponent`
are signed with an HMAC using `Security.salt` (CVE-2020-15400). Changing the salt
invalidates outstanding tokens. See [Security Fixes](/security/fixes).
:::

Security.cipherSeed  
A random numeric string (digits only) used to encrypt/decrypt
strings with `Security::cipher()`. That method is used only by the
`'cipher'` encryption type of `CookieComponent`, which is weak and kept
for backwards compatibility. Use the `'aes'` type for cookies instead, and
make sure `Security.useOpenSsl` is `true` (the app skeleton sets it).
The skeleton reads it from the `SECURITY_CIPHER_SEED` environment variable
when it is set. Only numbers from 1 to 4294967295 are meaningful; longer
values, including the 29-digit default of the original `config/core.php`, all
end up as the same seed `0`. No
value makes the `'cipher'` type secure. See
[cipherSeed and the 'cipher' Cookie Type](#cipher-seed).

Security.useOpenSsl  
When true, `Security::encrypt()` and `Security::decrypt()` (and therefore
the `'aes'` cookie encryption type) use OpenSSL instead of mcrypt. The mcrypt
extension is not bundled with PHP 7.2 and later, so enable it in `config/core.php`:

```php
Configure::write('Security.useOpenSsl', true);
```

The default is false. Applications created from the
[pieceofcake2/app](https://pieceofcake2.github.io/app/) skeleton (v2.3.1 and
later) already have this line; add it to existing applications.

Asset.timestamp  
Appends a timestamp which is last modified time of the particular
file at the end of asset files URLs (CSS, JavaScript, Image) when
using proper helpers.
Valid values:
(boolean) false - Doesn't do anything (default)
(boolean) true - Appends the timestamp when debug \> 0
(string) 'force' - Appends the timestamp when debug \>= 0

Acl.classname, Acl.database  
Constants used for CakePHP's Access Control List functionality. See
the Access Control Lists chapter for more information.

</div>

> [!NOTE]
> Cache configuration is also found in `core.php` — We'll be covering
> that later on, so stay tuned.

The `Configure` class can be used to read and write core
configuration settings on the fly. This can be especially handy if
you want to turn the debug setting on for a limited section of
logic in your application, for instance.

### Generating the Security Salts {#security-salts}

`Security.salt` must be a secret value that is unique to each application.
Generate it with a cryptographically secure random generator; do not type a
value yourself or reuse a value from an example. The commands below work in any
POSIX shell (`sh`, `bash`, `zsh`) on Linux and macOS.

`Security.salt` is 32 random bytes, written as 64 hexadecimal characters:

::: code-group

```bash [PHP]
php -r 'echo bin2hex(random_bytes(32)), PHP_EOL;'
```

```bash [OpenSSL]
openssl rand -hex 32
```

:::

Use a salt of at least 32 characters. It is the default key of
`CookieComponent`, and the `'aes'` encryption type (`Security::encrypt()`)
throws an exception for keys shorter than 32 bytes.

`Security.cipherSeed` is used only by the legacy `'cipher'` cookie type and
must be a number from 1 to 4294967295. Longer values, including the 29-digit
default of the original `config/core.php`, all end up as the same seed, and no
value makes `'cipher'` secure. Read
[cipherSeed and the 'cipher' Cookie Type](#cipher-seed) before you set it.

On Windows, run the PHP commands in Command Prompt or PowerShell with double
quotes instead of single quotes, for example
`php -r "echo bin2hex(random_bytes(32)), PHP_EOL;"`.

In an application created from the `pieceofcake2/app` skeleton, set the
values as the `SECURITY_SALT` and `SECURITY_CIPHER_SEED`
[environment variables](#environment-variables). For development, this
appends a salt and an in-range seed to `config/.env` (copy
`config/.env.example` first; when a variable is set twice, the last line
wins):

::: code-group

```bash [PHP]
php -r 'printf("SECURITY_SALT=\"%s\"\nSECURITY_CIPHER_SEED=\"%d\"\n", bin2hex(random_bytes(32)), random_int(1, 4294967295));' >> config/.env
```

```bash [Shell]
printf 'SECURITY_SALT="%s"\nSECURITY_CIPHER_SEED="%s"\n' "$(openssl rand -hex 32)" "$(od -An -N4 -tu4 /dev/urandom | tr -dc 0-9)" >> config/.env
```

:::

Keep the values secret and separate:

- Use different values in each environment (development, staging,
  production) and in each application.
- Do not commit them. `config/.env` and `.env` are ignored by Git in the
  skeleton; in production, set real environment variables instead of using a
  `.env` file. If you write the values in `config/core.php`, keep that file out
  of public repositories.
- If a salt has been exposed (committed, shared, or left at the example
  value), replace it.

#### What Depends on the Salts

Changing `Security.salt` affects the following:

- **Passwords hashed with the salt.** `SimplePasswordHasher`, the default
  password hasher of `FormAuthenticate` and `BasicAuthenticate`, and the
  deprecated `AuthComponent::password()` prepend the salt to the password
  (`Security::hash()` with `$salt = true`). Stored hashes no longer match, and
  users cannot log in. Bcrypt hashes (`BlowfishPasswordHasher`,
  `Security::hash($password, 'blowfish')`) do not use the salt and are not
  affected.
- **Encrypted cookies.** `CookieComponent` uses the salt as its key unless you
  set `$this->Cookie->key`, and the `'aes'` type also mixes the salt into its
  key even when you set your own key. Cookies written before the change can no
  longer be read, except `'cipher'` and `'rijndael'` cookies that use their own
  key.
- **Data encrypted with `Security::encrypt()`.** When `$hmacSalt` is not given,
  the salt is part of the encryption key, and `Security::decrypt()` returns
  `false` for values encrypted before the change.
- **Sessions.** When `Session.checkAgent` is not `false` (the default), the
  session stores a hash of the user agent and the salt, so existing sessions
  are discarded and users are logged out.
- **CSRF and form tampering tokens.** Tokens issued by `SecurityComponent` and
  `FormHelper` are signed with the salt. Forms rendered before the change are
  rejected once.

`Security.cipherSeed` is used only by `Security::cipher()`. Changing it makes
values encrypted with `Security::cipher()` unreadable, which in practice
means the cookies written by the default `'cipher'` type of `CookieComponent`.
See [cipherSeed and the 'cipher' Cookie Type](#cipher-seed).

#### Rotating the Salts

Sessions, CSRF tokens and form tokens recover by themselves: users log in
again and resubmit forms, so change the salt at a quiet time. Before you
change it, move the long-lived values away from it:

1. Move salted password hashes to bcrypt (`'passwordHasher' => 'Blowfish'`).
   A hash cannot be converted, so check the old hash when a user logs in and
   save a bcrypt hash of the password. Change the salt only after the salted
   hashes are gone, or ask the remaining users to reset their passwords.
2. Re-encrypt stored values that were encrypted with `Security::encrypt()`
   and the default `$hmacSalt`, passing the old salt to `Security::decrypt()`.
   Better, pass a dedicated `$hmacSalt` that does not change with
   `Security.salt`:

   ```php
   $plain = Security::decrypt($encrypted, $key, $oldSalt);
   $encrypted = Security::encrypt($plain, $key, $hmacSalt);
   ```

3. Encrypted cookies cannot be carried over: the `'aes'` type mixes
   `Security.salt` into its key, so its cookies become unreadable when the
   salt changes, and users with a remember-me cookie have to log in again
   once. If you still use the `'cipher'` type, move to `'aes'` with its own
   key at the same time, so that cookies are lost only once, and make sure
   `Security.useOpenSsl` is `true` (the app skeleton sets it; see
   [Migrating to 'aes'](#migrating-to-aes)).

#### Other Secrets

- Keys that you pass to `Security::encrypt()` and `Security::decrypt()`, or set
  as the `CookieComponent` key, must be at least 32 bytes long. Generate them
  like `Security.salt`, and read them into the configuration from environment
  variables, for example `Cookie.key` from `COOKIE_KEY` (see
  [Migrating to 'aes'](#migrating-to-aes)).
- Database and email passwords and API keys belong in environment variables or
  in ignored files such as `config/database.php` and `config/email.php`, not in
  committed code.

### cipherSeed and the 'cipher' Cookie Type {#cipher-seed}

`Security.cipherSeed` is used only by `Security::cipher()`. In practice that
means `CookieComponent`, whose default encryption type is `'cipher'`. The
`'cipher'` type has always been documented as insecure and kept only for
backwards compatibility. This section describes a known weakness of the seed,
what it means for your application, and how to move to the `'aes'` type.

#### How the Seed Is Used

`Security::cipher()` seeds PHP's Mersenne Twister generator with
`mt_srand((int)(float)$seed)` and XORs the value with numbers drawn from
`mt_rand()`. The key (`Security.salt`, unless you set the `CookieComponent`
key) is mixed in as well.

`mt_srand()` takes a 32-bit seed, so only numbers from 1 to 4294967295 are
meaningful. Longer values do not add anything:

- Values with more than 19 digits do not fit in a PHP integer, and the
  conversion loses bits.
- Every value with 27 or more digits, and about 95% of the values with 26
  digits, ends up as seed `0`.

Well-known values that end up as seed `0` include:

- `76859309657453542496749683645`, the default value in `config/core.php` of
  the original CakePHP 2 and of this fork up to v2.12.5, also used as the
  fallback in applications created from earlier versions of the
  `pieceofcake2/app` skeleton
- `7485712659625147843639846751`, the example value of the blog tutorial in
  the original book
- the 30-digit values generated by the command that earlier versions of
  `config/.env.example` in the `pieceofcake2/app` skeleton suggested

#### An In-Range Seed Is Not Enough

A seed from 1 to 4294967295 avoids the collapse to `0`, but it does not make
`'cipher'` secure:

- There are only about 4.3 billion seeds, so all of them can be tried.
- The keystream comes from Mersenne Twister, which is not a cryptographic
  generator, and the value is only XORed with it.
- `Security.salt` (or the `CookieComponent` key) is still mixed in, but that
  does not turn the construction into real encryption.

Do not rely on `'cipher'` to keep a value secret, whatever the seed.

#### Impact

- Only values encrypted with `Security::cipher()` are affected: cookies that
  `CookieComponent` writes with the `'cipher'` type (the default), and your
  own calls to `Security::cipher()`. The `'aes'` and `'rijndael'` cookie types,
  `Security::encrypt()`, password hashes, sessions and CSRF tokens do not use
  the seed.
- In applications that use one of the values above, or any other value with
  27 or more digits, the seed adds no secret at all, and every such
  application uses the same seed.
- Changing the seed to any other value makes existing `'cipher'` cookies
  unreadable, in the same way as switching to `'aes'` does.

#### Which Should I Do?

- **New applications:** use the `'aes'` type with its own key and
  `Security.useOpenSsl` set to `true` (the app skeleton sets it), as described
  in [Migrating to 'aes'](#migrating-to-aes), and never use `'cipher'`.
- **Existing applications with `'cipher'` cookies:** migrate to `'aes'`.
  Existing cookie values are lost once: for example, users who logged in with
  a remember-me cookie have to log in again, and preferences stored in cookies
  are reset. Follow the steps in [Migrating to 'aes'](#migrating-to-aes).
- **Applications that must keep reading old cookies for a transition
  period:** there is no seamless path. Old cookies stay readable only as long
  as you keep the old seed, which keeps the weakness. Setting an in-range seed
  changes the keystream as well, so old cookies become unreadable anyway, and
  `'cipher'` is still not secure afterwards. Plan a date on which the cookies
  are lost once, and switch to `'aes'` on that date instead of only changing
  the seed.
- **Applications that do not use `CookieComponent` or `Security::cipher()`,**
  or that use only the `'aes'` type: nothing to do beyond setting
  `Security.cipherSeed` to a valid value from 1 to 4294967295.

#### Migrating to 'aes' {#migrating-to-aes}

1. Generate a cookie key of 32 random bytes. Do not reuse `Security.salt`:

   ::: code-group

   ```bash [PHP]
   php -r 'echo bin2hex(random_bytes(32)), PHP_EOL;'
   ```

   ```bash [OpenSSL]
   openssl rand -hex 32
   ```

   :::

   Set it as the `COOKIE_KEY` environment variable: in `config/.env`
   for development, and as a real environment variable in production.

2. Make sure `Security.useOpenSsl` is `true` (the app skeleton sets it). The
   `'aes'` type uses `Security::encrypt()` and `Security::decrypt()`, which use
   OpenSSL only when this setting is `true`; otherwise they call `mcrypt`,
   which does not exist on PHP 8, and `'aes'` cookies fail. Older applications
   may not have the line, so add it to `config/core.php` (see
   [Enable OpenSSL](/appendices/migrating-from-cakephp-2-10#enable-openssl)):

   ```php
   Configure::write('Security.useOpenSsl', true);
   ```

3. Configure the type and the key of `CookieComponent`. How depends on the
   version of `pieceofcake2/cakephp`; see below.

::: pieceofcake2
Since v2.12.6, `Cookie.type` and `Cookie.key` can be set in configuration.

**v2.12.6 and later:** set them in `config/core.php`, next to the salts. No
controller changes are needed. Newer versions of the `pieceofcake2/app`
skeleton have these lines; add them if your `config/core.php` does not:

```php
// config/core.php
/**
 * Default encryption type and key of CookieComponent. The key is a random
 * value of at least 32 bytes.
 *
 * The key can be set with the COOKIE_KEY environment variable.
 */
Configure::write('Cookie.type', 'aes');
Configure::write('Cookie.key', env('COOKIE_KEY') ?: null);
```

`CookieComponent` uses the `'type'` setting or `$this->Cookie->type()` first,
then `Cookie.type`, then `'cipher'`. For the key, it uses the `'key'` setting or
`$this->Cookie->key` first, then `Cookie.key`, then `Security.salt`.

**v2.12.5 and earlier:** `CookieComponent` does not read `Cookie.type` and
`Cookie.key`, so the lines above have no effect on it. Keep the `Cookie.key`
line in `config/core.php`, and pass the type and the key in the controller:
the type in the component settings, and the key in `beforeFilter()`, before
any cookie is read. Do this in `AppController` or in every controller that
uses the component:

```php
// src/Controller/AppController.php
public $components = [
    'Cookie' => ['type' => 'aes'],
];

public function beforeFilter()
{
    parent::beforeFilter();
    $key = Configure::read('Cookie.key');
    if ($key) {
        $this->Cookie->key = $key;
    }
}
```

Assign the key only when it is set, because a `null` key would replace the
default key and encryption would fail. The `'type'` setting is honored since
v2.12.3; on older versions, call `$this->Cookie->type('aes');` in
`beforeFilter()`. This code keeps working on v2.12.6 and later, because
explicit settings take precedence.
:::

The `'aes'` type needs a key of at least 32 bytes. When `COOKIE_KEY` is not
set, `Cookie.key` is `null` and the key is `Security.salt`. If the salt is
shorter than 32 bytes, `Security::encrypt()` and `Security::decrypt()` throw
a `CakeException` whenever an encrypted cookie is written or read.

4. Optionally, give the cookie a new name with `$this->Cookie->name`, so that
   old `'cipher'` cookies are ignored instead of being decrypted with the new
   type. They expire by themselves.

5. Deploy. Users with a remember-me cookie log in again once.

The `'aes'` type uses `Security::encrypt()`, which also mixes `Security.salt`
into the key. Changing the salt later still makes `'aes'` cookies unreadable.

#### Generating a cipherSeed

Only if you still use `'cipher'`, set `Security.cipherSeed` to a number from 1
to 4294967295. This avoids the collapse to `0`, but `'cipher'` stays insecure,
and changing the seed makes existing `'cipher'` cookies unreadable:

::: code-group

```bash [PHP]
php -r 'echo random_int(1, 4294967295), PHP_EOL;'
```

```bash [Shell]
od -An -N4 -tu4 /dev/urandom | tr -dc 0-9; echo
```

:::

#### Warnings Since v2.12.6 {#cipher-warnings}

::: pieceofcake2
Since v2.12.6, `pieceofcake2/cakephp` reports these problems without changing
behaviour:

- `'cipher'` is deprecated. The first use of the `'cipher'` type or of
  `Security::cipher()` in a process (one request or one console run) writes a
  `warning` to the log, also when debug is off. With debug on, the same
  message is also triggered as `E_USER_DEPRECATED`. The message names the
  source and the fix: set `Cookie.type` to `'aes'` and `Cookie.key` to a
  random value of at least 32 bytes, with `Security.useOpenSsl` enabled.
- When `Security.cipherSeed` is not a digits-only number from 1 to
  4294967295, the first `Security::cipher()` call in a process also logs a
  `warning`. The seed itself is never logged. With debug on,
  `Debugger::checkSecurityKeys()` (called by `AuthComponent`) also triggers an
  `E_USER_NOTICE` for such a seed.
- The fallback `config/core.php` of the core, used only when the application
  has no `config/core.php`, sets the in-range seed `'2659758734'` (the same
  default as the app skeleton) and `Security.useOpenSsl`. The notice asking you
  to change the default `Security.cipherSeed` fires for both the former
  default and this new one.
- `Cookie.type` and `Cookie.key` set the default type and key of
  `CookieComponent` (see [Migrating to 'aes'](#migrating-to-aes)).

Values are encrypted and decrypted exactly as before, so existing cookies stay
readable, and nothing throws. Do not change `Security.cipherSeed` only to
silence the warning: a new seed makes existing `'cipher'` cookies unreadable,
and `'cipher'` stays insecure with any seed. Switch to `'aes'` instead.

If your test suite turns deprecations or notices into exceptions, the first
`'cipher'` use in a test run, or `AuthComponent` with debug on and an
out-of-range seed, now fails. Pass `'aes'` as the mode of
`cookieEncrypted()` / `assertCookieEncrypted()` of `IntegrationTestTrait`
once the application uses `'aes'`.

In v2.12.5 and earlier, nothing is reported; the weakness is the same.
:::

### Configuration Constants

While most configuration options are handled by Configure, there
are a few constants that CakePHP uses during runtime.

`constant` **LOG_ERROR**

Error constant. Used for differentiating error logging and
debugging. Currently PHP supports LOG_DEBUG.

### Core Cache Configuration

CakePHP uses two cache configurations internally. `_cake_model_` and `_cake_core_`.
`_cake_core_` is used to store file paths, and object locations. `_cake_model_` is
used to store schema descriptions, and source listings for datasources. Using a fast
cache storage like APCu or Memcached is recommended for these configurations, as
they are read on every request. By default both of these configurations expire every
10 seconds when debug is greater than 0. Both are defined at the end of
`config/core.php`.

The available cache engines are File, Apc (which uses the APCu extension when it
is loaded), Memcached, Memcache and Redis.

::: pieceofcake2
The Xcache and Wincache cache engines were
removed. Switch any configuration that uses them to one of the engines above.
:::

As with all cached data stored in `Cache` you can clear data using
`Cache::clear()`.

## Configure Class

`class` **Configure**

Despite few things needing to be configured in CakePHP, it's
sometimes useful to have your own configuration rules for your
application. In the past you may have defined custom configuration
values by defining variable or constants in some files. Doing so
forces you to include that configuration file every time you needed
to use those values.

CakePHP's Configure class can be used to store and retrieve
application or runtime specific values. Be careful, this class
allows you to store anything in it, then use it in any other part
of your code: a sure temptation to break the MVC pattern CakePHP
was designed for. The main goal of Configure class is to keep
centralized variables that can be shared between many objects.
Remember to try to live by "convention over configuration" and you
won't end up breaking the MVC structure we've set in place.

This class can be called from
anywhere within your application, in a static context:

```php
Configure::read('debug');
```

`static` Configure::**write**($key, $value)

- `string $key`: The key to write, can be a `dot notation` value.
- `mixed $value`: The value to store.

Use `write()` to store data in the application's configuration:

```php
Configure::write('Company.name', 'Pizza, Inc.');
Configure::write('Company.slogan', 'Pizza for your body and soul');
```

> [!NOTE]
> The `dot notation` used in the `$key` parameter can be used to
> organize your configuration settings into logical groups.

The above example could also be written in a single call:

```php
Configure::write(
    'Company',
    [
        'name' => 'Pizza, Inc.',
        'slogan' => 'Pizza for your body and soul',
    ],
);
```

You can use `Configure::write('debug', $int)` to switch between
debug and production modes on the fly. This is especially handy for
JSON or XML responses where debugging information can cause
parsing problems.

`static` Configure::**read**($key = null)

- `string $key`: The key to read, can be a `dot notation` value

Used to read configuration data from the application. Defaults to
CakePHP's important debug value. If a key is supplied, the data is
returned. Using our examples from write() above, we can read that
data back:

```php
Configure::read('Company.name'); // yields: 'Pizza, Inc.'
Configure::read('Company.slogan'); // yields: 'Pizza for your body and soul'

Configure::read('Company');

// yields:
['name' => 'Pizza, Inc.', 'slogan' => 'Pizza for your body and soul'];
```

If \$key is left null, all values in Configure will be returned. If the value
corresponding to the specified \$key does not exist then null will be
returned.

`static` Configure::**consume**($key)

- `string $key`: The key to read, can use be a `dot notation` value

Read and delete a key from Configure. This is useful when you want to
combine reading and deleting values in a single operation.

`static` Configure::**check**($key)

- `string $key`: The key to check.

Used to check if a key/path exists and has not-null value.

::: info Added in version 2.3
`Configure::check()` was added in 2.3
:::

`static` Configure::**delete**($key)

- `string $key`: The key to delete, can be a `dot notation` value

Used to delete information from the application's configuration:

```php
Configure::delete('Company.name');
```

`static` Configure::**version**()

Returns the CakePHP version for the current application.

`static` Configure::**config**($name, $reader)

- `string $name`: The name of the reader being attached.
- `ConfigReaderInterface $reader`: The reader instance being attached.

Attach a configuration reader to Configure. Attached readers can
then be used to load configuration files. See [Loading Configuration Files](#loading-configuration-files)
for more information on how to read configuration files.

`static` Configure::**configured**($name = null)

- `string $name`: The name of the reader to check, if null
  a list of all attached readers will be returned.

Either check that a reader with a given name is attached, or get
the list of attached readers.

`static` Configure::**drop**($name)

Drops a connected reader object.

## Reading and writing configuration files

CakePHP comes with two built-in configuration file readers.
`PhpReader` is able to read PHP config files, in the same
format that Configure has historically read. `IniReader` is
able to read ini config files. See the [PHP documentation](https://www.php.net/parse_ini_file)
for more information on the specifics of ini files.
To use a core config reader, you'll need to attach it to Configure
using `Configure::config()`:

```php
// Read config files from CONFIG (config/)
Configure::config('default', new PhpReader());

// Read config files from another path.
Configure::config('default', new PhpReader('/path/to/your/config/files/'));
```

Both readers read from the `CONFIG` directory by default, and from a plugin's
`Config/` directory when you use plugin syntax such as
`Configure::load('MyPlugin.my_file')`.

You can have multiple readers attached to Configure, each reading
different kinds of configuration files, or reading from
different types of sources. You can interact with attached readers
using a few other methods on Configure. To see which reader
aliases are attached you can use `Configure::configured()`:

```php
// Get the array of aliases for attached readers.
Configure::configured();

// Check if a specific reader is attached
Configure::configured('default');
```

You can also remove attached readers. `Configure::drop('default')`
would remove the default reader alias. Any future attempts to load configuration
files with that reader would fail.

<a id="loading-configuration-files"></a>

### Loading configuration files

`static` Configure::**load**($key, $config = 'default', $merge = true)

- `string $key`: The identifier of the configuration file to load.
- `string $config`: The alias of the configured reader.
- `boolean $merge`: Whether or not the contents of the read file
  should be merged, or overwrite the existing values.

Once you've attached a config reader to Configure you can load configuration files:

```php
// Load config/my_file.php using the 'default' reader object.
Configure::load('my_file', 'default');
```

Loaded configuration files merge their data with the existing runtime configuration
in Configure. This allows you to overwrite and add new values
into the existing runtime configuration. By setting `$merge` to false, values
will not ever overwrite the existing configuration.

### Creating or modifying configuration files

`static` Configure::**dump**($key, $config = 'default', $keys = [])

- `string $key`: The name of the file/stored configuration to be created.
- `string $config`: The name of the reader to store the data with.
- `array $keys`: The list of top-level keys to save. Defaults to all
  keys.

Dumps all or some of the data in Configure into a file or storage system
supported by a config reader. The serialization format
is decided by the config reader attached as \$config. For example, if the
'default' adapter is a `PhpReader`, the generated file will be a PHP
configuration file loadable by the `PhpReader`

Given that the 'default' reader is an instance of PhpReader.
Save all data in Configure to the file `my_config.php`:

```php
Configure::dump('my_config.php', 'default');
```

Save only the error handling configuration:

```php
Configure::dump('error.php', 'default', ['Error', 'Exception']);
```

`Configure::dump()` can be used to either modify or overwrite
configuration files that are readable with `Configure::load()`

::: info Added in version 2.2
`Configure::dump()` was added in 2.2.
:::

### Storing runtime configuration

`static` Configure::**store**($name, $cacheConfig = 'default', $data = null)

- `string $name`: The storage key for the cache file.
- `string $cacheConfig`: The name of the cache configuration to store the
  configuration data with.
- `mixed $data`: Either the data to store, or leave null to store all data
  in Configure.

You can also store runtime configuration values for use in a future request.
Since configure only remembers values for the current request, you will
need to store any modified configuration information if you want to
use it in subsequent requests:

```php
// Store the current configuration in the 'user_1234' key in the 'default' cache.
Configure::store('user_1234', 'default');
```

Stored configuration data is persisted in the `Cache` class. This allows
you to store Configuration information in any storage engine that `Cache` can talk to.

### Restoring runtime configuration

`static` Configure::**restore**($name, $cacheConfig = 'default')

- `string $name`: The storage key to load.
- `string $cacheConfig`: The cache configuration to load the data from.

Once you've stored runtime configuration, you'll probably need to restore it
so you can access it again. `Configure::restore()` does exactly that:

```php
// restore runtime configuration from the cache.
Configure::restore('user_1234', 'default');
```

When restoring configuration information it's important to restore it with
the same key, and cache configuration as was used to store it. Restored
information is merged on top of the existing runtime configuration.

## Creating your own Configuration readers

Since configuration readers are an extensible part of CakePHP,
you can create configuration readers in your application and plugins.
Configuration readers need to implement the `ConfigReaderInterface`.
This interface defines a `read()` and a `dump()` method.
If you really like XML files, you could create a simple Xml config
reader for you application:

```php
// src/Lib/Configure/MyXmlReader.php
class MyXmlReader implements ConfigReaderInterface
{
    protected $_path;

    public function __construct($path = null)
    {
        if (!$path) {
            $path = CONFIG;
        }
        $this->_path = $path;
    }

    public function read($key)
    {
        $xml = Xml::build($this->_path . $key . '.xml');

        return Xml::toArray($xml);
    }

    /**
     * As of 2.3 a dump() method is also required
     */
    public function dump($key, $data)
    {
        // code to dump data to file
    }
}
```

In your `config/bootstrap.php` you could attach this reader and use it:

```php
Configure::config('xml', new MyXmlReader());
// ...

Configure::load('my_xml');
```

With Composer classmap autoloading for `src/`, the reader class is found
automatically after `composer dump-autoload`. Without it, load the class
first with `App::uses('MyXmlReader', 'Configure');`.

> [!WARNING]
> It is not a good idea to call your custom configure class `XmlReader` because that
> class name is an internal PHP one already:
> [XMLReader](https://www.php.net/manual/en/book.xmlreader.php)

The `read()` method of a config reader, must return an array of the configuration information
that the resource named `$key` contains.

`interface` **ConfigReaderInterface**

Defines the interface used by classes that read configuration data and
store it in `Configure`

`method` ConfigReaderInterface::**read**($key)

`method` ConfigReaderInterface::**dump**($key, $data)

::: info Added in version 2.3
`ConfigReaderInterface::dump()` was added in 2.3.
:::

`exception` **ConfigureException**

Thrown when errors occur when loading/storing/restoring configuration data.
`ConfigReaderInterface` implementations should throw this exception when they
encounter an error.

### Built-in Configuration readers

`class` **PhpReader**

`class` **IniReader**

<a id="inflection-configuration"></a>

## Inflection Configuration

CakePHP's naming conventions can be really nice - you can name your
database table big_boxes, your model BigBox, your controller
BigBoxesController, and everything just works together
automatically. The way CakePHP knows how to tie things together is
by *inflecting* the words between their singular and plural forms.

There are occasions (especially for our non-English speaking
friends) where you may run into situations where CakePHP's
`Inflector` (the class that pluralizes, singularizes, camelCases, and
under_scores) might not work as you'd like. If CakePHP won't
recognize your Foci or Fish, you can tell CakePHP about your
special cases.

### Loading custom inflections

You can use `Inflector::rules()` in the file
`config/bootstrap.php` to load custom inflections:

```php
Inflector::rules('singular', [
    'rules' => [
        '/^(bil)er$/i' => '\1',
        '/^(inflec|contribu)tors$/i' => '\1ta',
    ],
    'uninflected' => ['singulars'],
    'irregular' => ['spins' => 'spinor'],
]);
```

or:

```php
Inflector::rules('plural', ['irregular' => ['phylum' => 'phyla']]);
```

Will merge the supplied rules into the inflection sets defined in
the framework's `src/Cake/Utility/Inflector.php`
(`vendor/pieceofcake2/cakephp/src/Cake/Utility/Inflector.php`), with the
added rules taking precedence over the core rules.

## Bootstrapping CakePHP

If you have any additional configuration needs, use CakePHP's
bootstrap file, found in `config/bootstrap.php`. This file is
executed just after CakePHP's core bootstrapping.

This file is ideal for a number of common bootstrapping tasks:

- Defining convenience functions.
- Registering global constants.
- Defining additional model, view, and controller paths.
- Creating cache configurations.
- Configuring inflections.
- Loading configuration files.

Be careful to maintain the MVC software design pattern when you add
things to the bootstrap file: it might be tempting to place
formatting functions there in order to use them in your
controllers.

Resist the urge. You'll be glad you did later on down the line.

You might also consider placing things in the `AppController` class.
This class is a parent class to all of the controllers in your
application. `AppController` is a handy place to use controller
callbacks and define methods to be used by all of your
controllers.
