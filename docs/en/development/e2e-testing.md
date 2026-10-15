# End-to-End Testing {#e2e-testing}

End-to-End tests open your application in a real browser and use it the way a
user does: they click links, fill in forms, press buttons and check what
appears on the screen. The
[pieceofcake2/panther](https://pieceofcake2.github.io/panther/) package brings
[Symfony Panther](https://symfony.com/doc/current/testing/end_to_end.html) to
CakePHP 2 applications, set up the way Symfony applications use it:
`static::createPantherClient()` starts your application with PHP's built-in
web server and drives Chrome, and the test case is a `CakeTestCase`, so
`$fixtures` work as in your other tests.

This page is a short guide to the setup. The package documentation
([website](https://pieceofcake2.github.io/panther/),
[Markdown](https://github.com/pieceofcake2/panther/blob/1.x/docs/README.md))
is the full reference for the configuration, the client and the assertions,
and the [Symfony documentation](https://symfony.com/doc/current/testing/end_to_end.html)
applies as it is.

## When to Use End-to-End Tests {#when-to-use-e2e-tests}

CakePHP applications are usually tested on several levels (see
[Types of Tests](/development/testing#types-of-tests)):

| | Unit tests | Controller tests (`testAction()`) | [Integration tests](/development/integration-testing) | End-to-End tests |
| --- | --- | --- | --- | --- |
| Runs | In the PHPUnit process | In the PHPUnit process, no HTTP | In the PHPUnit process, no HTTP | A real HTTP server and a real Chrome |
| Speed | Milliseconds | Milliseconds per request | Milliseconds per request | Seconds per test |
| JavaScript, CSS, layout | No | No | No | Yes |
| Cookies, sessions, redirects | - | Simulated | Simulated through the real dispatcher | Real |
| `SecurityComponent` tokens | - | Usually disabled or mocked | Generated from the posted data | Real tokens from the real form |
| View variables, return values | Yes | Yes | Yes | No, only what a user sees |
| Setup | None | None | None | Chrome, chromedriver, a test database for the web server |

End-to-End tests are the right tool for:

- a user flow across several pages, such as signing up, checking out or a
  multi-step form;
- anything that depends on JavaScript, the layout, responsive design or the
  browser;
- forms protected by `SecurityComponent` and the helpers used in templates:
  controller tests usually bypass the token validation, and integration tests
  compute the tokens from the posted data instead of the real form, so
  mistakes in the form only show up in a browser.

Test the logic of models, components and controllers, status codes and edge
cases with unit and integration tests. Keep the End-to-End tests few and
focused on the main flows: each of them runs the whole application in a
browser.

## Requirements

- PHP 8.0 or later (Panther 2.1 on PHP 8.0, Panther 2.4 on PHP 8.1 and later)
- pieceofcake2/cakephp 2.12 or later and PHPUnit 9.6
- Google Chrome or Chromium, and a chromedriver of the same major version

## Installation

Install the package and [bdi](https://github.com/dbrekelmans/bdi) as
development dependencies, and let bdi download the chromedriver that matches
your Chrome:

```bash
composer require --dev pieceofcake2/panther dbrekelmans/bdi
vendor/bin/bdi detect drivers
```

bdi saves the driver to `drivers/`, where Panther looks for it; add `/drivers/`
to your `.gitignore`. Run `vendor/bin/bdi detect drivers` again after Chrome
updates itself. A chromedriver of your package manager works as well.

The package is a library (namespace `PieceOfCake2\Panther\`), not a CakePHP
plugin, so there is nothing to load with `CakePlugin::load()`.

## Setup

### PHPUnit configuration {#phpunit-configuration}

As in a Symfony application, the End-to-End tests live next to your other
tests, in `tests/TestCase/EndToEnd/` (`app/Test/Case/EndToEnd/` in the legacy
layout), run from the same `phpunit.xml.dist`, and are configured with Panther's
`PANTHER_*` variables as `<server>` elements. The
[pieceofcake2/app](https://pieceofcake2.github.io/app/) skeleton already has
this configuration; uncomment the extension there:

```xml
<phpunit bootstrap="./tests/bootstrap.php"
         backupGlobals="true"
         defaultTestSuite="App"
         stderr="true"
         colors="true">
    <php>
        <!-- The web server uses the test datasource, where the fixtures are -->
        <server name="PANTHER_APP_ENV" value="test"/>
        <!-- Screenshots of the failed End-to-End tests -->
        <server name="PANTHER_ERROR_SCREENSHOT_DIR" value="./tmp/error-screenshots"/>
    </php>
    <!-- Keeps the web server and the browser running for the whole run -->
    <extensions>
        <extension class="PieceOfCake2\Panther\ServerExtension"/>
    </extensions>
    <listeners>
        <listener class="CakeFixtureInjector" file="./vendor/pieceofcake2/cakephp/src/Cake/TestSuite/Fixture/CakeFixtureInjector.php"/>
    </listeners>
    <testsuites>
        <testsuite name="App">
            <directory>./tests/TestCase/</directory>
            <exclude>./tests/TestCase/EndToEnd/</exclude>
        </testsuite>
        <testsuite name="EndToEnd">
            <directory>./tests/TestCase/EndToEnd/</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

- `defaultTestSuite="App"` keeps `vendor/bin/phpunit` fast and free of Chrome:
  it runs only the `App` suite. Delete this attribute to make
  `vendor/bin/phpunit` run everything, as a Symfony application does.
- Nothing starts until a test calls `createPantherClient()`, so a run of the
  `App` suite starts no browser. The extension keeps the web server and the
  browser from the first End-to-End test to the last one.
- A `<server>` element replaces a shell variable of the same name. Put
  per-machine values, such as `PANTHER_NO_HEADLESS=1`, in your shell or in a
  `phpunit.xml` copy of `phpunit.xml.dist`.

The other variables, such as `PANTHER_WEB_SERVER_PORT` and
`PANTHER_WEB_SERVER_DIR`, and the options of `createPantherClient()` are listed
in the [configuration page](https://pieceofcake2.github.io/panther/configuration)
of the package documentation
([Markdown](https://github.com/pieceofcake2/panther/blob/1.x/docs/configuration.md)).

### The test database

The web server is a separate PHP process, and the browser can only see data
that it reads. It must use the datasource where `CakeTestCase` loads the
fixtures, `test`. `PANTHER_APP_ENV=test` gives the web server the environment
variable `APP_ENV=test`; in the constructor of `DATABASE_CONFIG` in
`config/database.php`, use the `test` datasource as `default` then:

```php
public function __construct()
{
    if (env('APP_ENV') === 'test') {
        $this->default = $this->test;
    }
}
```

`config/database.php.default` of pieceofcake2/app has these lines; copy them
into your `config/database.php` if it is older. The web server inherits the
environment variables of the test process, so it gets the same `DB_TEST_*`
values.

- Never point the tests to your development database.
- List every table that the visited pages use in `$fixtures`.
- The test and the web server share the database: create the records a test
  needs with models before opening the page, and check the result of a form in
  the database as well as on the page.
- SQLite `:memory:` cannot be shared between processes. Use a database file or
  a database server.

See [Test Database](https://pieceofcake2.github.io/panther/test-database) in
the package documentation for details.

## Writing Tests

Extend `PieceOfCake2\Panther\TestSuite\PantherTestCase` and get the browser
with `static::createPantherClient()`:

```php
<?php
declare(strict_types=1);

namespace App\Test\TestCase\EndToEnd;

use ClassRegistry;
use PieceOfCake2\Panther\TestSuite\PantherTestCase;

class ArticlesTest extends PantherTestCase
{
    /**
     * @var array<string>
     */
    public $fixtures = ['app.article'];

    /**
     * @return void
     */
    public function testIndex(): void
    {
        $client = static::createPantherClient();
        $client->request('GET', '/articles');

        $this->assertPageTitleContains('Articles');
        $this->assertSelectorTextContains('table', 'First article');
    }

    /**
     * @return void
     */
    public function testAdd(): void
    {
        $client = static::createPantherClient();
        $client->request('GET', '/articles/add');
        $client->submitForm('Submit', [
            'data[Article][title]' => 'Written in the browser',
            'data[Article][body]' => 'Hello',
        ]);

        $this->assertSelectorTextContains('.message', 'The article has been saved.');
        $this->assertSame(1, ClassRegistry::init('Article')->find('count', [
            'conditions' => ['Article.title' => 'Written in the browser'],
        ]));
    }
}
```

The client is Panther's: `request()`, `clickLink()`, `submitForm()`,
`waitFor()`, `executeScript()`, `takeScreenshot()` and the rest work as in the
[Symfony documentation](https://symfony.com/doc/current/testing/end_to_end.html).
The assertions are those of Symfony's and Panther's `WebTestAssertionsTrait`,
such as `assertSelectorTextContains()`, `assertSelectorIsVisible()` and
`assertSelectorWillContain()`. A few points for CakePHP:

- Give `submitForm()` the full field names of `FormHelper`, such as
  `data[User][email]`. Forms protected by `SecurityComponent` work, because
  the browser submits the real form.
- The browser is kept from one test to the next, with its cookies. A test that
  must start logged out clears them with `$client->getCookieJar()->clear()`.
- Wait for the results of JavaScript with `waitFor()` or the
  `assertSelectorWill*()` assertions; never use `sleep()`.

[Writing Tests](https://pieceofcake2.github.io/panther/writing-tests) in the
package documentation covers forms, waiting, several browsers, a mobile
viewport and the list of assertions.

## Failure Screenshots {#failure-screenshots-and-html}

With `PANTHER_ERROR_SCREENSHOT_DIR` and the extension, a test that fails or
errors after it created a browser saves a screenshot of each browser to
`tmp/error-screenshots/`. Add `/tmp/error-screenshots/` to your `.gitignore`
(pieceofcake2/app has it).

![Screenshot saved for a failed test of a contact form](/e2e-failure-screenshot.png)

## Running the Tests

```bash
# The App suite only (defaultTestSuite): no web server, no browser
vendor/bin/phpunit

# The End-to-End tests
vendor/bin/phpunit --testsuite EndToEnd

# Everything
vendor/bin/phpunit --testsuite App,EndToEnd

# One test case
vendor/bin/phpunit --testsuite EndToEnd --filter ArticlesTest

# Watch the browser, and pause after a failure until you press Enter
PANTHER_NO_HEADLESS=1 vendor/bin/phpunit --testsuite EndToEnd --debug
```

`--filter` searches only the selected suites: without `--testsuite EndToEnd`,
it searches the `App` suite and finds no End-to-End test. A path argument runs
that file or directory regardless of `defaultTestSuite`.

## Continuous Integration

On GitHub Actions, run the End-to-End tests in a separate job: the unit test
job keeps running `vendor/bin/phpunit` (the `App` suite) without a browser.
The `ubuntu-latest` runner has Chrome, and `vendor/bin/bdi detect drivers`
installs the matching chromedriver. The main steps are:

```yaml
  end-to-end:
    name: End-to-End
    runs-on: ubuntu-latest

    steps:
      # ... checkout, setup-php, composer install, the test database

      - name: Install chromedriver for the installed Chrome
        run: vendor/bin/bdi detect drivers

      - name: Run End-to-End tests
        env:
          PANTHER_NO_SANDBOX: 1
        run: vendor/bin/phpunit --testsuite EndToEnd

      - name: Upload the error screenshots
        if: ${{ failure() }}
        uses: actions/upload-artifact@v7
        with:
          name: error-screenshots
          path: tmp/error-screenshots/
          retention-days: 7
          if-no-files-found: ignore
```

`PANTHER_NO_SANDBOX=1` is needed on the GitHub runners. If your build creates
CSS or JavaScript files, build them before the End-to-End step. If you removed
`defaultTestSuite` from `phpunit.xml.dist`, run the unit test job with
`--testsuite App`.

The complete job is in
[Continuous Integration](/development/continuous-integration#e2e-tests), and
the package has a
[workflow with a job summary](https://github.com/pieceofcake2/panther/blob/1.x/docs/examples/end-to-end.yml)
([CI with GitHub Actions](https://pieceofcake2.github.io/panther/ci)).

## Troubleshooting

**"This version of ChromeDriver only supports Chrome version N"**: Chrome was
updated. Run `vendor/bin/bdi detect drivers` again.

**Chrome crashes at startup in Docker or CI**: set `PANTHER_NO_SANDBOX=1`.

**`--filter` finds no tests**: add `--testsuite EndToEnd`. With
`defaultTestSuite="App"`, `--filter` searches only the `App` suite.

**"Missing Database Table" in the browser**: the web server does not use the
test datasource, or a table is missing from `$fixtures`. Check
`PANTHER_APP_ENV` in `phpunit.xml.dist` and the `APP_ENV` lines of
`config/database.php`.

See [Troubleshooting](https://pieceofcake2.github.io/panther/troubleshooting)
in the package documentation for more.
