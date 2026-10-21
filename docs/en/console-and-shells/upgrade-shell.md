<a id="upgrade-shell"></a>

# Upgrade shell

The upgrade shell does most of the mechanical work of upgrading an application
from CakePHP 1.3 to 2.0: it moves files to their 2.0 locations and rewrites
removed APIs, helper calls, constants and so on. It is provided by the
[pieceofcake2/upgrade](https://pieceofcake2.github.io/upgrade/) plugin.

::: pieceofcake2
`UpgradeShell` was removed from the core and moved to the
pieceofcake2/upgrade plugin. Most applications never need it.
:::

The shell works on applications in the legacy `app/` layout, so the examples below run it
through `vendor/bin/cake`. Install the plugin as a development dependency and load it in
`app/Config/bootstrap.php`:

```bash
composer require --dev pieceofcake2/upgrade
```

Composer installs it into `plugins/Upgrade/` when your `composer.json` has the
`installer-paths` setting described in
[Installing a plugin with Composer](/plugins/how-to-install-plugins#composer).

```php
// app/Config/bootstrap.php
CakePlugin::load('Upgrade');
```

Make a backup of your application (or commit it to version control) before you
start. To run all upgrade steps:

```bash
vendor/bin/cake upgrade all
```

If you would like to see what the shell will do without modifying files perform
a dry run first with `--dry-run`:

```bash
vendor/bin/cake upgrade all --dry-run
```

To upgrade your plugin run the command:

```bash
vendor/bin/cake upgrade all --plugin YourPluginName
```

You are able to run each upgrade step individually (`tests`, `locations`,
`i18n`, `helpers`, `basics`, `request`, `configure`, `constants`,
`controller_redirects`, `components` and `exceptions`). To see all the steps
and options available run the command:

```bash
vendor/bin/cake upgrade --help
```

The upgrade shell only covers the 1.3 to 2.0 changes. An application converted
this way must then be brought up to CakePHP 2.10.24 before it can run on the
Piece of Cake 2 fork; see [Migrating from CakePHP 2.10](/appendices/migrating-from-cakephp-2-10) and the
migration guides in the appendices.

## Rector rules {#rector-rules}

The `rector` command runs a set of [Rector](https://getrector.com/) rules for a
change of pieceofcake2/cakephp that your code must follow. It is not part of
`all`. Install and load the plugin as above (in `config/bootstrap.php` in the
CakePHP 5 layout), then give the directories or files to change:

```bash
# See the changes
bin/cake upgrade rector --rules variadic-arguments src tests --dry-run

# Apply them
bin/cake upgrade rector --rules variadic-arguments src tests
```

| Rules | Change |
| --- | --- |
| `variadic-arguments` | Adds the variadic parameter of CakePHP to the methods that override a method that has one, e.g. `query($sql)` of an `AppModel` becomes `query($sql, ...$params)`. See [Variadic arguments](/appendices/migrating-from-cakephp-2-10#variadic-arguments). |

The changed code also runs on the versions of pieceofcake2/cakephp before the
change, so you can run the rules before you update. See the
[documentation of the plugin](https://pieceofcake2.github.io/upgrade/) for the
details.
