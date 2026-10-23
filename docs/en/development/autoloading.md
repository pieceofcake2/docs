# Autoloading

Composer loads the classes of CakePHP, of your application and of its plugins
when they are first used. You don't include their files yourself, and you
don't need `App::uses()`.

Composer finds a class in one of two ways:

- A **classmap** is a list of the classes in some folders. Composer builds it
  when you run `composer install`, `composer update` or
  `composer dump-autoload`. Run `composer dump-autoload` after you add a class
  file to one of these folders.
- **PSR-4** maps a namespace to a folder, for example `App\` to `src/`.
  Composer finds a new class file without `composer dump-autoload`, unless you
  build an optimized autoloader with `composer dump-autoload -o`.

Run `composer dump-autoload` also after you change the `autoload` section of
`composer.json`.

This page shows which of them each part of an application uses.

## The Core {#the-core}

The classes of CakePHP are in the `Cake` namespace, in
`vendor/pieceofcake2/cakephp/src/`. The `composer.json` of
`pieceofcake2/cakephp` loads them with PSR-4 (`"Cake\\": "src/"`).

The global names of CakePHP 2, such as `Controller` or `CakeRequest`, are
aliases of the namespaced classes. When your code uses a global name first,
an autoloader of the core loads the namespaced class. So both names work, and
you need nothing in your own `composer.json`. See
[The Classes of the Framework](namespaced-classes#the-classes-of-the-framework).

## An Application in the Modern Layout {#modern-layout}

The [pieceofcake2/app-legacy](/installation/application-skeleton-legacy) skeleton loads
`src/` with a classmap:

```json
"autoload": {
    "classmap": ["src/"]
}
```

The classmap finds the global classes in `src/`, such as `AppController` in
`src/Controller/AppController.php`, whatever folder they are in.

For namespaced classes, add a PSR-4 entry, for example `"App\\": "src/"`.
Set [`App.namespace`](namespaced-classes#app-namespace) too, so that CakePHP
finds your controllers, models and other classes by their short names:

```json
"autoload": {
    "classmap": ["src/"],
    "psr-4": {
        "App\\": "src/"
    }
}
```

The [pieceofcake2/app](/installation/application-skeleton#namespace) skeleton,
whose classes are all in the `App` namespace, loads `src/` with PSR-4 only. See
[Namespaced Classes](namespaced-classes).

## An Application in the Legacy `app/` Layout {#legacy-layout}

List the folders of your classes in the classmap:

```json
"autoload": {
    "classmap": ["app/Console/", "app/Controller/", "app/Lib/", "app/Model/", "app/View/"]
}
```

Then run `composer dump-autoload` and delete the `App::uses()` lines.

`App::uses()` is deprecated as of 2.13.0, but it still works. An application
without a classmap keeps running. The first call in a process logs a warning,
and with `debug` above 0 it also triggers `E_USER_DEPRECATED`. See
[Removing App::uses() calls](../core-utility-libraries/app#removing-app-uses).

## Plugins {#plugins}

Composer loads the classes of a plugin when the `composer.json` of the plugin
has `autoload` settings. When you write a plugin, add them as follows.

- In the **modern layout**, `src/` holds the namespaced classes. Load them with
  PSR-4:

  ```json
  "autoload": {
      "psr-4": {
          "ContactManager\\": "src/"
      }
  }
  ```

- In the **legacy layout**, the global classes are in `Controller/`, `Model/`,
  `View/Helper/` and so on. Load them with a classmap:

  ```json
  "autoload": {
      "classmap": ["Controller/", "Lib/", "Model/", "View/"]
  }
  ```

  List only the folders that the plugin has. `composer dump-autoload` stops
  with an error for a folder that does not exist.

A plugin that uses both layouts needs both entries.

For a plugin without these settings, add its folder to the classmap of your
application, for example
`"classmap": ["src/", "plugins/MyPlugin/"]`.

See [How To Create Plugins](../plugins/how-to-create-plugins#publish-your-plugin).

## Vendor Files {#vendor-files}

Libraries installed with Composer are autoloaded and need no extra code.

Files that you copy into your project by hand go in `vendors/` at the project
root (`app/Vendor/` in the legacy layout). Composer does not manage this
folder, but it can load its classes. Add the folder to the classmap:

```json
"autoload": {
    "classmap": ["src/", "vendors/"]
}
```

Load files that do not contain a class, or whose class differs from the file
name, with `App::import()`. See
[Loading Vendor Files](../core-utility-libraries/app#loading-vendor-files).

## Overriding Core Classes {#overriding-core-classes}

A copy of a core class in `src/Cake/`, at the path of its namespace, replaces
the class of the core. Exclude the core file with `exclude-from-classmap`. See
[Overriding classes in CakePHP](../core-utility-libraries/app#overriding-classes-in-cakephp).
