# クラスの読み込み

CakePHP、アプリケーション、プラグインのクラスは、最初に使うときに Composer が読み込みます。
ファイルを自分でインクルードする必要はなく、`App::uses()` も要りません。

Composer がクラスを探す方法は 2 つあります。

- **クラスマップ** は、いくつかのフォルダにあるクラスの一覧です。`composer install`、`composer update`、
  `composer dump-autoload` を実行したときに Composer が作ります。これらのフォルダにクラスのファイルを
  追加したら、`composer dump-autoload` を実行してください。
- **PSR-4** は、名前空間をフォルダに対応させます。たとえば `App\` を `src/` に対応させます。
  新しいクラスのファイルは、`composer dump-autoload` を実行しなくても見つかります。
  ただし、`composer dump-autoload -o` で最適化したオートローダーを作ったときは別です。

`composer.json` の `autoload` を変更したときも、`composer dump-autoload` を実行してください。

このページでは、アプリケーションの各部分がどちらを使うかを説明します。

## コア {#the-core}

CakePHP のクラスは `Cake` 名前空間にあり、ファイルは `vendor/pieceofcake2/cakephp/src/` にあります。
`pieceofcake2/cakephp` の `composer.json` が PSR-4 (`"Cake\\": "src/"`) で読み込みます。

`Controller` や `CakeRequest` のような CakePHP 2 のグローバルな名前は、名前空間のあるクラスの別名です。
コードが先にグローバルな名前を使うと、コアのオートローダーが名前空間のあるクラスを読み込みます。
そのため、どちらの名前も使えます。アプリケーションの `composer.json` には何も書く必要はありません。
[フレームワークのクラス](namespaced-classes#the-classes-of-the-framework) を参照してください。

## モダンな構成のアプリケーション {#modern-layout}

[pieceofcake2/app](https://pieceofcake2.github.io/app/ja/) スケルトンは、`src/` をクラスマップで読み込みます。

```json
"autoload": {
    "classmap": ["src/"]
}
```

クラスマップは、`src/Controller/AppController.php` の `AppController` のような `src/` のグローバルな
クラスを、どのフォルダにあっても見つけます。

名前空間のあるクラスには、`"App\\": "src/"` のような PSR-4 の設定を追加します。
あわせて [`App.namespace`](namespaced-classes#app-namespace) も設定すると、CakePHP がコントローラーや
モデルなどのクラスを短い名前で見つけます。

```json
"autoload": {
    "classmap": ["src/"],
    "psr-4": {
        "App\\": "src/"
    }
}
```

[名前空間のあるクラス](namespaced-classes) を参照してください。

## 従来の `app/` 構成のアプリケーション {#legacy-layout}

クラスのあるフォルダをクラスマップに書きます。

```json
"autoload": {
    "classmap": ["app/Console/", "app/Controller/", "app/Lib/", "app/Model/", "app/View/"]
}
```

それから `composer dump-autoload` を実行し、`App::uses()` の行を削除します。

`App::uses()` は 2.13.0 で非推奨になりましたが、今も動きます。クラスマップの無いアプリケーションも
そのまま動きます。プロセスで最初の呼び出しのときに警告をログに書き、`debug` が 0 より大きいときは
`E_USER_DEPRECATED` も発生させます。
[`App::uses()` の呼び出しを削除する](../core-utility-libraries/app#removing-app-uses) を参照してください。

## プラグイン {#plugins}

プラグインの `composer.json` に `autoload` の設定があれば、プラグインのクラスは Composer が読み込みます。
プラグインを作るときは、次のように設定を書きます。

- **モダンな構成** では、`src/` に名前空間のあるクラスを置きます。PSR-4 で読み込みます。

  ```json
  "autoload": {
      "psr-4": {
          "ContactManager\\": "src/"
      }
  }
  ```

- **従来の構成** では、グローバルなクラスは `Controller/`、`Model/`、`View/Helper/` などにあります。
  クラスマップで読み込みます。

  ```json
  "autoload": {
      "classmap": ["Controller/", "Lib/", "Model/", "View/"]
  }
  ```

  プラグインにあるフォルダだけを書いてください。存在しないフォルダがあると、
  `composer dump-autoload` はエラーで止まります。

両方の構成を使うプラグインには、両方の設定が要ります。

この設定の無いプラグインは、アプリケーションのクラスマップにそのフォルダを追加します。
たとえば `"classmap": ["src/", "plugins/MyPlugin/"]` と書きます。

[プラグインの作成](../plugins/how-to-create-plugins#publish-your-plugin) を参照してください。

## ベンダーのファイル {#vendor-files}

Composer でインストールしたライブラリは自動で読み込まれるので、コードは要りません。

手でプロジェクトにコピーしたファイルは、プロジェクトのルートの `vendors/` に置きます
(従来の構成では `app/Vendor/`)。このフォルダは Composer が管理しませんが、クラスは Composer で読み込めます。
フォルダをクラスマップに追加してください。

```json
"autoload": {
    "classmap": ["src/", "vendors/"]
}
```

クラスの無いファイルや、クラス名とファイル名が違うファイルは、`App::import()` で読み込みます。
[Vendor ファイルをローディングする](../core-utility-libraries/app#loading-vendor-files) を参照してください。

## コアのクラスのオーバーライド {#overriding-core-classes}

コアのクラスを `src/Cake/` の名前空間に合ったパスにコピーすると、コアのクラスの代わりに使われます。
コアのファイルは `exclude-from-classmap` で除外します。
[CakePHP のクラスをオーバーライドする](../core-utility-libraries/app#overriding-classes-in-cakephp) を参照してください。
