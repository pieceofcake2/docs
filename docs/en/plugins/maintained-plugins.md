# Maintained Plugins

The [Piece of Cake 2](https://github.com/pieceofcake2) project maintains the
framework fork together with related repositories and forks of popular
CakePHP 2 plugins. All maintained plugins are compatible with PHP 8.x, and
their CI checks them with [PHPStan](/development/static-analysis) at level 5.
Names link to the documentation site of each package, or to the GitHub
repository where the package has no documentation site. "Markdown" links to
the Markdown source of the documentation on GitHub, which AI tools can read
directly.

## Core repositories

- [CakePHP](https://github.com/pieceofcake2/cakephp) - CakePHP 2.x framework
- [App Skeleton](https://pieceofcake2.github.io/app/) ([Markdown](https://github.com/pieceofcake2/app/blob/2.x/docs/README.md)) - Application skeleton
- [Plugin Skeleton](https://github.com/pieceofcake2/plugin) - Plugin skeleton
- [Bake Plugin](https://pieceofcake2.github.io/bake/) ([Markdown](https://github.com/pieceofcake2/bake/blob/1.x/docs/README.md)) - Code generation tool
- [Upgrade Plugin](https://pieceofcake2.github.io/upgrade/) ([Markdown](https://github.com/pieceofcake2/upgrade/blob/1.x/docs/README.md)) - Upgrade utility (CakePHP 1.3 to 2.0)

## Development tools

- [PHPStan CakePHP](https://pieceofcake2.github.io/phpstan-cakephp2/) ([Markdown](https://github.com/pieceofcake2/phpstan-cakephp2/blob/main/docs/README.md)) - PHPStan extension for static analysis
  (see [Static Analysis](/development/static-analysis))
- [PHPCS CakePHP2](https://pieceofcake2.github.io/phpcs-cakephp2/) ([Markdown](https://github.com/pieceofcake2/phpcs-cakephp2/blob/main/docs/README.md)) - PHP_CodeSniffer standard for CakePHP 2
  (see [Coding Standards](/contributing/cakephp-coding-conventions))
- [Panther](https://pieceofcake2.github.io/panther/) ([Markdown](https://github.com/pieceofcake2/panther/blob/1.x/docs/README.md)) - End-to-End tests with Symfony Panther
  (see [End-to-End Testing](/development/e2e-testing))

## Maintained plugins

### Debugging

- [DebugKit Plugin](https://pieceofcake2.github.io/debug_kit/) ([Markdown](https://github.com/pieceofcake2/debug_kit/blob/2.x/docs/README.md)) - Debug toolbar

### Infrastructure

- [Aws Plugin](https://pieceofcake2.github.io/aws/) ([Markdown](https://github.com/pieceofcake2/aws/blob/1.x/docs/README.md)) - AWS SDK integration
- [AsyncAws Plugin](https://pieceofcake2.github.io/async-aws/) ([Markdown](https://github.com/pieceofcake2/async-aws/blob/1.x/docs/README.md)) - AsyncAws integration, a lighter alternative to the Aws plugin
- [Bref Plugin](https://github.com/pieceofcake2/bref) - AWS Lambda deployment with Bref

### Logging & Monitoring

- [Monolog Plugin](https://pieceofcake2.github.io/monolog/) ([Markdown](https://github.com/pieceofcake2/monolog/blob/2.x/docs/README.md)) - Monolog integration
- [Sentry Plugin](https://pieceofcake2.github.io/sentry/) ([Markdown](https://github.com/pieceofcake2/sentry/blob/1.x/docs/README.md)) - Error tracking and monitoring

### Templating

- [TwigView Plugin](https://pieceofcake2.github.io/twig-view/) ([Markdown](https://github.com/pieceofcake2/twig-view/blob/2.x/docs/README.md)) - Twig template engine
- [CsvView Plugin](https://pieceofcake2.github.io/csv-view/) ([Markdown](https://github.com/pieceofcake2/csv-view/blob/2.x/docs/README.md)) - CSV view class

### View Helpers

- [Markdown Plugin](https://pieceofcake2.github.io/markdown/) ([Markdown](https://github.com/pieceofcake2/markdown/blob/2.x/docs/README.md)) - Markdown helper
- [MenuBuilder Plugin](https://pieceofcake2.github.io/menu_builder/) ([Markdown](https://github.com/pieceofcake2/menu_builder/blob/2.x/docs/README.md)) - Navigation menu helper
- [Vite Plugin](https://pieceofcake2.github.io/vite/) ([Markdown](https://github.com/pieceofcake2/vite/blob/1.x/docs/README.md)) - Vite asset helper with dev server support
- [WebpackEncore Plugin](https://pieceofcake2.github.io/webpack-encore/) ([Markdown](https://github.com/pieceofcake2/webpack-encore/blob/1.x/docs/README.md)) - Webpack Encore asset helper

### Forms & Validation

- [Xform Plugin](https://pieceofcake2.github.io/xform/) ([Markdown](https://github.com/pieceofcake2/xform/blob/2.x/docs/README.md)) - Extended form helper
- [Cakeplus Plugin](https://pieceofcake2.github.io/cakeplus/) ([Markdown](https://github.com/pieceofcake2/cakeplus/blob/2.x/docs/README.md)) - Form utilities and enhancements
- [Captcha Plugin](https://pieceofcake2.github.io/captcha/) ([Markdown](https://github.com/pieceofcake2/captcha/blob/1.x/docs/README.md)) - CAPTCHA widget and
  server-side verification with Cloudflare Turnstile (default), Google
  reCAPTCHA v2/v3 or hCaptcha
- [Purifiable Plugin](https://pieceofcake2.github.io/purifiable/) ([Markdown](https://github.com/pieceofcake2/purifiable/blob/1.x/docs/README.md)) - HTML sanitization of model fields with HTML Purifier

### Database

- [Datasources Plugin](https://pieceofcake2.github.io/datasources/) ([Markdown](https://github.com/pieceofcake2/datasources/blob/2.x/docs/README.md)) - Additional datasources (CSV / TSV files, arrays and more)

### Authentication and Authorization

- [Authenticate Plugin](https://pieceofcake2.github.io/authenticate/) ([Markdown](https://github.com/pieceofcake2/authenticate/blob/1.x/docs/README.md)) - Authentication classes
- [AclExtras Plugin](https://pieceofcake2.github.io/acl_extras/) ([Markdown](https://github.com/pieceofcake2/acl_extras/blob/2.x/docs/README.md)) - Console tools for managing ACL tables
- [AclMenu Plugin](https://pieceofcake2.github.io/acl_menu/) ([Markdown](https://github.com/pieceofcake2/acl_menu/blob/1.x/docs/README.md)) - Navigation menus of the actions allowed by ACL
- [Totp Plugin](https://pieceofcake2.github.io/totp/) ([Markdown](https://github.com/pieceofcake2/totp/blob/1.x/docs/README.md)) - Two-factor authentication with one-time passwords (TOTP)

### Files

- [Upload Plugin](https://pieceofcake2.github.io/upload/) ([Markdown](https://github.com/pieceofcake2/upload/blob/2.x/docs/README.md)) - File upload handling
- [Imagine Plugin](https://pieceofcake2.github.io/imagine/) ([Markdown](https://github.com/pieceofcake2/imagine/blob/1.x/docs/README.md)) - Image processing library wrapper
- [CakePdf Plugin](https://pieceofcake2.github.io/cakepdf/) ([Markdown](https://github.com/pieceofcake2/cakepdf/blob/1.x/docs/README.md)) - PDF generation

### SEO

- [Seo Plugin](https://pieceofcake2.github.io/seo/) ([Markdown](https://github.com/pieceofcake2/seo/blob/6.x/docs/README.md)) - Search engine optimization
- [Sitemap Plugin](https://pieceofcake2.github.io/sitemap/) ([Markdown](https://github.com/pieceofcake2/sitemap/blob/1.x/docs/README.md)) - Sitemap generation

### I18n

- [Localized Plugin](https://pieceofcake2.github.io/localized/) ([Markdown](https://github.com/pieceofcake2/localized/blob/2.x/docs/README.md)) - Localization utilities

## Installing a plugin

Install plugins with Composer and load them as usual with `CakePlugin::load()`.
See each plugin's documentation site for the package name and setup. With the
[modern layout](/appendices/migrating-to-the-modern-layout), Composer installs plugins
into `plugins/` through the `installer-paths` setting.

Plugins that don't support Composer autoloading still need `App::uses()` for
their classes, for example `App::uses('MyPluginHelper', 'MyPlugin.View/Helper')`.

## Origins {#origins}

Most of these packages continue an existing CakePHP project. The table lists where each
one comes from: a fork continues the history of the original repository, a package based
on another project was extracted from it, and an original package was created by
pieceofcake2. The README of each package has the same information in its Origin section,
and its LICENSE file keeps the copyright notices of the original authors.

| Package | Origin | Original authors | Last upstream version |
| --- | --- | --- | --- |
| [pieceofcake2/cakephp](https://github.com/pieceofcake2/cakephp) | Fork of [cakephp/cakephp](https://github.com/cakephp/cakephp), with the PHP 8 work of [kamilwylegala/cakephp2-php8](https://github.com/kamilwylegala/cakephp2-php8) | CakePHP community (Cake Software Foundation, Inc.) | 2.10.24 |
| [pieceofcake2/app](https://github.com/pieceofcake2/app) | Based on [cakephp/cakephp](https://github.com/cakephp/cakephp) (`app/`) | CakePHP community (Cake Software Foundation, Inc.) | 2.10.24 |
| [pieceofcake2/plugin](https://github.com/pieceofcake2/plugin) | Original | - | - |
| [pieceofcake2/bake](https://github.com/pieceofcake2/bake) | Based on [cakephp/cakephp](https://github.com/cakephp/cakephp) (BakeShell) | CakePHP community (Cake Software Foundation, Inc.) | 2.10.24 |
| [pieceofcake2/upgrade](https://github.com/pieceofcake2/upgrade) | Based on [cakephp/cakephp](https://github.com/cakephp/cakephp) (UpgradeShell) | CakePHP community (Cake Software Foundation, Inc.) | 2.10.24 |
| [pieceofcake2/phpstan-cakephp2](https://github.com/pieceofcake2/phpstan-cakephp2) | Fork of [sidz/phpstan-cakephp2](https://github.com/sidz/phpstan-cakephp2) | Oleg Zhulnev | 0.2.0 |
| [pieceofcake2/phpcs-cakephp2](https://github.com/pieceofcake2/phpcs-cakephp2) | Original | - | - |
| [pieceofcake2/panther](https://github.com/pieceofcake2/panther) | Original | - | - |
| [pieceofcake2/debug_kit](https://github.com/pieceofcake2/debug_kit) | Fork of [cakephp/debug_kit](https://github.com/cakephp/debug_kit) | Mark Story and the CakePHP community (Cake Software Foundation, Inc.) | 2.2.9 |
| [pieceofcake2/aws](https://github.com/pieceofcake2/aws) | Original | - | - |
| [pieceofcake2/async-aws](https://github.com/pieceofcake2/async-aws) | Original | - | - |
| [pieceofcake2/bref](https://github.com/pieceofcake2/bref) | Original | - | - |
| [pieceofcake2/monolog](https://github.com/pieceofcake2/monolog) | Fork of [jadb/cakephp-monolog](https://github.com/jadb/cakephp-monolog) | Jad Bitar | 2.0.0 |
| [pieceofcake2/sentry](https://github.com/pieceofcake2/sentry) | Original | - | - |
| [pieceofcake2/twig-view](https://github.com/pieceofcake2/twig-view) | Fork of [predominant/TwigView](https://github.com/predominant/TwigView) | Kjell Bublitz and Graham Weldon | v0.6 |
| [pieceofcake2/csv-view](https://github.com/pieceofcake2/csv-view) | Fork of [FriendsOfCake/cakephp-csvview](https://github.com/FriendsOfCake/cakephp-csvview) | Jose Diaz-Gonzalez | 1.3.0 |
| [pieceofcake2/markdown](https://github.com/pieceofcake2/markdown) | Fork of [chronon/CakePHP-Markdown-Plugin](https://github.com/chronon/CakePHP-Markdown-Plugin) | Gregory Gaskill | 2.1.0 |
| [pieceofcake2/menu_builder](https://github.com/pieceofcake2/menu_builder) | Fork of [torifat/cake-menu_builder](https://github.com/torifat/cake-menu_builder) | Rifat Nabi | v2.0.3 |
| [pieceofcake2/vite](https://github.com/pieceofcake2/vite) | Original | - | - |
| [pieceofcake2/webpack-encore](https://github.com/pieceofcake2/webpack-encore) | Original | - | - |
| [pieceofcake2/xform](https://github.com/pieceofcake2/xform) | Fork of [ichikaway/xformHelper](https://github.com/ichikaway/xformHelper) | Yasushi Ichikawa | 2.0.0 |
| [pieceofcake2/cakeplus](https://github.com/pieceofcake2/cakeplus) | Fork of [ichikaway/cakeplus](https://github.com/ichikaway/cakeplus) | Yasushi Ichikawa | v2.0 |
| [pieceofcake2/captcha](https://github.com/pieceofcake2/captcha) | Original | - | - |
| [pieceofcake2/datasources](https://github.com/pieceofcake2/datasources) | Fork of [cakephp/datasources](https://github.com/cakephp/datasources) | CakePHP community (Cake Software Foundation, Inc.) | commit [`b741921`](https://github.com/cakephp/datasources/commit/b741921) (`master`) |
| [pieceofcake2/authenticate](https://github.com/pieceofcake2/authenticate) | Fork of [FriendsOfCake/Authenticate](https://github.com/FriendsOfCake/Authenticate) | M. Ypes (Ceeram) | 1.0.2 |
| [pieceofcake2/acl_extras](https://github.com/pieceofcake2/acl_extras) | Fork of [markstory/acl_extras](https://github.com/markstory/acl_extras) | Mark Story | 1.2.1 |
| [pieceofcake2/upload](https://github.com/pieceofcake2/upload) | Fork of [FriendsOfCake/cakephp-upload](https://github.com/FriendsOfCake/cakephp-upload) | Jose Diaz-Gonzalez | 1.3.1 |
| [pieceofcake2/imagine](https://github.com/pieceofcake2/imagine) | Fork of [burzum/cakephp-imagine-plugin](https://github.com/burzum/cakephp-imagine-plugin) | Florian Krämer | 1.0.6 |
| [pieceofcake2/cakepdf](https://github.com/pieceofcake2/cakepdf) | Fork of [FriendsOfCake/CakePdf](https://github.com/FriendsOfCake/CakePdf) | Ceeram | 1.0.3 |
| [pieceofcake2/seo](https://github.com/pieceofcake2/seo) | Fork of [webtechnick/CakePHP-Seo-Plugin](https://github.com/webtechnick/CakePHP-Seo-Plugin) | Nick Baker, Alan Blount and Matt Reishus | v6.1.0 |
| [pieceofcake2/sitemap](https://github.com/pieceofcake2/sitemap) | Fork of [loadsys/CakePHP-Sitemap](https://github.com/loadsys/CakePHP-Sitemap) | Loadsys Web Strategies | 1.0.2 |
| [pieceofcake2/localized](https://github.com/pieceofcake2/localized) | Fork of [cakephp/localized](https://github.com/cakephp/localized) | CakePHP community (Cake Software Foundation, Inc.) | 2.7.0 |
| [pieceofcake2/docs](https://github.com/pieceofcake2/docs) | Fork of [cakephp/docs](https://github.com/cakephp/docs) | CakePHP community (Cake Software Foundation, Inc.) | commit [`6b9d755`](https://github.com/cakephp/docs/commit/6b9d755) (`2.x`) |
| [pieceofcake2/docs-skeleton](https://github.com/pieceofcake2/docs-skeleton) | Fork of [cakephp/docs-skeleton](https://github.com/cakephp/docs-skeleton) | CakePHP community (Cake Software Foundation, Inc.) | commit [`f256691`](https://github.com/cakephp/docs-skeleton/commit/f256691) (`main`) |

## Contributing

The project welcomes bug reports (especially PHP 8.x compatibility issues),
security vulnerability reports and pull requests.
