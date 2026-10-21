# プラグインの作成

動作サンプルとして、 [プラグインの使い方](../plugins/how-to-use-plugins) セクションを参考に ContactManager
プラグインを作りましょう。

プラグインはプロジェクトルートの `plugins/` ディレクトリに置きます。従来の `app/` 構成のアプリケーションでは
`app/Plugin/` も使えます。プラグインフォルダーの名前 **ContactManager** は、プラグインの名前と同じにします。

プラグインの構成には、次の 2 つがあります。

- CakePHP 2 の [従来のプラグインの構成](#the-legacy-plugin-layout)
- アプリケーションや CakePHP 5 と同じ [モダンなプラグインの構成](#the-modern-plugin-layout)

CakePHP はモダンな構成のディレクトリを先に探し、従来の構成のディレクトリをその後に探します。
そのため、プラグインを 1 ディレクトリずつモダンな構成に移せます。

新しくプラグインを作るときは、 [pieceofcake2/plugin](https://github.com/pieceofcake2/plugin)
スケルトンをひな形にするとよいでしょう。

## 従来のプラグインの構成 {#the-legacy-plugin-layout}

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

- クラスはグローバルなクラスで、`Controller/`、`Model/`、`View/Helper/` などに置きます。
  プラグインの `composer.json` のクラスマップで Composer が読み込みます ([プラグインの公開](#publish-your-plugin) を参照)。
- テンプレートは、`View/Helper/` と並べて `View/` に置きます。
- `Config/` には、`bootstrap.php`、`routes.php`、`Configure::load('ContactManager.file')` で読む設定ファイル、
  `Config/Schema/` のスキーマを置きます。
- `Locale/` には翻訳を、`Vendor/` にはベンダーのファイルを置きます。
- テストは `Test/Case/` と `Test/Fixture/`、または `tests/TestCase/` と `tests/Fixture/` に置きます。

## モダンなプラグインの構成 {#the-modern-plugin-layout}

::: pieceofcake2 Added in version 2.13.0
モダンなプラグインの構成 ([#88](https://github.com/pieceofcake2/cakephp/pull/88))。
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

- `src/` には、CakePHP 5 と同じく名前空間のあるクラスだけを置きます。たとえば
  `src/Controller/Component/ContactsComponent.php` には `ContactManager\Controller\Component\ContactsComponent` を置きます。
  - プラグインの `composer.json` の PSR-4 (`"ContactManager\\": "src/"`) で Composer が読み込みます。
  - 設定には、プラグインの接頭辞を付けた名前 (`'ContactManager.Contacts'`) か `ContactsComponent::class` を書きます。
    [名前空間のあるクラス](/ja/development/namespaced-classes) を参照してください。
- プラグインのグローバルなクラスは、従来の構成と同じく `Controller/`、`Model/`、`View/Helper/` などに置きます。
- `templates/` にはテンプレートを置きます。CakePHP は `View/` より先にここを探します。
- `config/` には、従来の構成の `Config/` と同じファイルを置きます。`config/` があると、CakePHP はプラグインの設定を
  `Config/` ではなく `config/` から読みます。
- `resources/locales/` には翻訳を置きます。CakePHP は `Locale/` より先にここを探します。
- `vendor/` にはベンダーのファイルを置きます。CakePHP は `Vendor/` より先にここを探します。

> [!NOTE]
> `App::uses()` は `src/` の名前空間のあるクラスを読み込みません。Composer が読み込みます。
> `src/` はテンプレート、翻訳、ベンダーのファイルを探す場所でもないので、`src/View/` のビュークラスを
> テンプレートとして描画することはありません。

> [!WARNING]
> `templates/`、`resources/locales/` のファイルは、`View/`、`Locale/` にある同じ名前のファイルの代わりに使われます。
> これらのディレクトリや `config/`、`vendor/` をプラグインに足す前に、別の目的ですでに使っていないかを確かめてください。
> たとえば、開発用にチェックアウトしたプラグインには Composer の `vendor/` があることがあります。

## プラグインのフォルダ

プラグインは CakePHP アプリケーションによく似ていて、実際ほぼ同じものです。使わないフォルダは作る必要はありません。
コンポーネントとビヘイビアだけを定義するプラグインでは、`View/` や `templates/` をまったく持たなくても構いません。

> [!NOTE]
> URL でプラグインにアクセスできるようにしたい場合、AppController と AppModel への
> 定義が必要です。この 2 つの特別なクラスにはプラグインにちなんだ名前を付け、
> アプリケーションの AppController と AppModel を継承します。
> ContactManager の例ではこうなります。

```php
// plugins/ContactManager/Controller/ContactManagerAppController.php の中で
class ContactManagerAppController extends AppController
{
}
```

```php
// plugins/ContactManager/Model/ContactManagerAppModel.php の中で
class ContactManagerAppModel extends AppModel
{
}
```

次のクラスは、`App::uses()` が無くても CakePHP が見つけます。

- アプリケーションの `AppController`、`AppModel`、`AppHelper`、`AppShell`
- プラグインの `ContactManagerAppController`、`ContactManagerAppModel`

プラグインのそのほかのクラスは Composer に読み込ませてください。
[クラスの読み込み](/ja/development/autoloading#plugins) を参照してください。

もしこれらの特別なクラスの定義を忘れると、”Missing Controller” エラーがでます。

プラグインの作成は、 [Bake プラグイン](https://pieceofcake2.github.io/bake/ja/) を使えば非常に簡単です。

プラグインを bake するのは以下のコマンドになります。

```bash
bin/cake bake plugin ContactManager
```

そうすると、いつも通りの bake ができます。 例えば controllers を bake するには、次のようにします。

```bash
bin/cake bake controller Contacts --plugin ContactManager
```

Bake のインストール方法と使い方は、
[Bakeでコード生成](../console-and-shells/code-generation-with-bake)
のチャプターを参照してください。

> [!WARNING]
> プラグインは名前空間ではありません。すべてのプラグインとアプリケーションのグローバルなクラスは、同じ名前の集まりを共有します。
> 2 つのプラグインに、同じ名前のグローバルなクラスやファイルを置くことはできません。
> たとえばプラグイン名を接頭辞にして名前を重ならないようにするか、
> [モダンなプラグインの構成](#the-modern-plugin-layout) の `src/` に名前空間のあるクラスを置いてください。

## プラグインコントローラ

ContactManager プラグインのコントローラーは、
`plugins/ContactManager/Controller/` に設置されます。 主にやりたい事は
contacts の管理ですので、このプラグインには ContactsController が必要です。

そこで ContactsController を `plugins/ContactManager/Controller` に設置し、
このように書きます。

```php
// plugins/ContactManager/Controller/ContactsController.php の中で

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
> このコントローラは、親アプリケーションの AppController ではなく、 （
> ContactManagerAppController という名前の）プラグインの
> AppController を継承します。
>
> モデルの名前の頭にプラグイン名がつくことにも注意してください。 これは、
> プラグイン内のモデルとメインのアプリケーション内のモデルの区別が必要だからです。
>
> 今回の例では、ContactManager.Contact はこのコントローラのデフォルトの
> モデルなのですから、 \$uses 配列に書く必要は無かったかもしれませんが、
> プラグイン名を正しく頭につける方法を示すためにここでは書いています。

これまで行ってきたものにアクセスしたい場合、 /contact_manager/contacts
にアクセスします。 Contact モデルをまだ定義してないので、 “Missing Model”
エラーがでるはずです。

## プラグインモデル {#plugin-models}

プラグインのモデルは `plugins/ContactManager/Model` に設置されます。
プラグインの ContactsController は既に定義してあるので、そのモデルを作成します。

```php
// plugins/ContactManager/Model/Contact.php の中で

class Contact extends ContactManagerAppModel
{
}
```

/contact_manager/contacts に（contacts テーブルがある状態で）
今アクセスすると、“Missing View” エラーが発生します。 次にこれを作ります。

> [!NOTE]
> もしプラグイン内のモデルを参照したいなら、ドットで区切られた、
> モデル名といっしょのプラグイン名を含む必要があります。

例えば:

```php
// plugins/ContactManager/Model/Contact.php の中で

class Contact extends ContactManagerAppModel
{
    public $hasMany = ['ContactManager.AltName'];
}
```

プラグインの接頭語との連携の無い配列キーを参照したいなら、代わりのシンタックスを使います。

```php
// plugins/ContactManager/Model/Contact.php の中で

class Contact extends ContactManagerAppModel
{
    public $hasMany = [
        'AltName' => [
            'className' => 'ContactManager.AltName',
        ],
    ];
}
```

## プラグインビュー

ビューは通常のアプリケーション内での動作として振る舞います。
`plugins/[PluginName]/View/` フォルダー (モダンな構成では `templates/`) 内に設置するだけです。
ContactManager プラグインでは、ContactsController::index()
アクションのビューが必要になるので、 このような内容になります。

```html
<!-- plugins/ContactManager/View/Contacts/index.ctp: -->
<h1>Contacts</h1>
<p>Following is a sortable list of your contacts</p>
<!-- A sortable list of contacts would go here....-->
```

> [!NOTE]
> プラグインからのエレメントの使い方に関する情報は、 [View Elements](../views#view-elements)
> を参照してください。

### アプリケーション内でのプラグインビューのオーバーライド

プラグインのビューはあるパスを使ってオーバーライドできます。 ContactManager
という名のプラグインがあるなら、 `templates/Plugin/[Plugin]/[Controller]/[view].ctp`
(従来の `app/` レイアウトでは `app/View/Plugin/...`)
というテンプレートを作成することでオーバーライドできます。 Contacts コントローラーには
このファイルを作ります。

    templates/Plugin/ContactManager/Contacts/index.ctp

このファイルを作れば、
`plugins/ContactManager/View/Contacts/index.ctp`
を上書きできます。

## プラグインアセット

プラグインのウェブアセット（php ファイルではない）は、 プラグインの
’webroot’ ディレクトリを通して受け取られます。

    plugins/ContactManager/webroot/
                                   css/
                                   js/
                                   img/
                                   pdf/

通常の webroot と同じようにどのディレクトリにどんなファイルでも置くことができます。

ただ、プラグインの静的アセットや画像や JavaScript または CSS は、 ディスパチャーを
経由しますが、非常に効率が悪くなることを覚えておいてください。 ですので、本番環境では
それらにシンボリックリンクを張っておくことを強くおすすめします。
例えばこのようにします。

```bash
ln -s ../../plugins/YourPlugin/webroot/css/yourplugin.css webroot/css/yourplugin.css
```

### プラグイン内のアセットへのリンク

プラグイン内のアセットへのリクエストの始めは、単に /plugin_name/
を頭に付けるだけで、アプリケーションの webroot として動作します。

例えば、`/contact_manager/js/some_file.js` へのリンクは、
`plugins/ContactManager/webroot/js/some_file.js` で受け取れます。

> [!NOTE]
> アセットのパスの前に `/your_plugin/` に付けるのが重要です。
> 魔法のようなことが起きます！

::: info Changed in version 2.1
アセットのリクエストには `プラグイン記法` を使用してください。 View での利用方法:
:::

```php
echo $this->Html->css('ContactManager.style');
```

## コンポーネント、ヘルパーとビヘイビア

コンポーネント、ヘルパーやビヘイビアを持つプラグインは、通常の CakePHP アプリケーションの
ようなものです。 コンポーネントだけ、または、ヘルパーやビヘイビアだけを含むプラグインも
作る事が可能で、 他のプロジェクトで簡単に使えるような、再利用できるコンポーネントを作る
すばらしい方法にもなり得ます。

このようなコンポーネントを作る事は、実際、通常のアプリケーションとして作る事と同じであり、
特別な命名規則もありません。

プラグインの内部や外部からコンポーネントを参照する方法は、
コンポーネント名の前にプラグイン名を付けるだけです。 例えば:

```php
// 'ContactManager' プラグインのコンポーネントとして定義

class ExampleComponent extends Component
{
}

// あなたのコントローラで下記のように呼び出す

public $components = ['ContactManager.Example'];
```

同じテクニックはヘルパーとビヘイビアにも使えます。

> [!NOTE]
> プラグインの AppController や AppModel と同じく、プラグインのヘルパーも `App::uses()`
> を書かずに `AppHelper` を継承できます。

<a id="registering-dsn-schemes"></a>

## DSN スキームの登録

データソースとメールのトランスポートは、`'dsn'` キーに書いた DSN で設定できます
([データベースの DSN](/ja/development/configuration#database-url) と
[メーラー DSN](/ja/core-utility-libraries/email#mailer-dsn) を参照)。DSN のスキームは DSN クラスマップから探されます。CakePHP 3.x 以降の
`setDsnClassMap()` と同じく、プラグインはこのマップに独自のスキームを追加できます。
`ConnectionManager` と `CakeEmail` がそれぞれマップを持っています。

| メソッド | |
|----------|---|
| `setDsnClassMap($map)` | スキームを追加するか、組み込みのスキームを置き換えます。`null` を指定するとスキームを削除します。スキームの大文字と小文字は区別されません |
| `getDsnClassMap()` | 組み込みのスキームを含むマップを返します |
| `parseDsn($dsn, $config = [])` | DSN の設定のキーを返します |

::: pieceofcake2
DSN クラスマップは v2.12.7 で追加されます。
:::

スキームはプラグインの `Config/bootstrap.php` で登録します。アプリケーションはこのファイルを
`CakePlugin::load('MyPlugin', ['bootstrap' => true])` で読み込みます。

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

マップの値はクラス名か callable です。

- プラグイン記法の **クラス名** は `datasource` か `transport` に設定され、DSN の
  各部分はその種類の通常のキーになります。その後にクエリパラメーターが追加され、こちらが
  優先されます。

  | マップ | キー |
  |--------|------|
  | `ConnectionManager` | `login`、`password`、`host` (IPv6 アドレスの角括弧は除かれます)、`port`、`database` (先頭の `/` を除いたパス) |
  | `CakeEmail` | `username`、`password`、`host` (ホスト `default` は省かれ、トランスポートのデフォルトが使われます)、`port` |

- クロージャや `[$class, 'method']` のような文字列ではない **callable** は、DSN の各部分と、
  `'dsn'` キーを除いた設定を引数に呼ばれ、クラスを含む設定のキーを返します。各部分は
  `scheme`、`username`、`password`、`host`、`port`、`path` (存在するものだけ。URL デコード
  済み) と `query` (常に配列) です。どのクエリパラメーターを使うかは callable が決めます。
  文字列は常にクラス名として扱われ、関数名にはなりません。

組み込みのスキームも同じマップのエントリー (データベースはクラス名、メールは callable)
なので、プラグインはこれを置き換えることもできます。たとえば `mysql` を独自のデータソースに
向けられます。DSN のクエリパラメーターの `datasource` と `transport` は、引き続きクラス名を
上書きし、登録されていないスキームも使えるようにします。

DSN クラスマップの無い `pieceofcake2/cakephp` のリリースにもプラグインが対応する場合は、
呼び出しを `method_exists()` でガードしてください。

<a id="when-to-register-the-schemes"></a>

### スキームを登録するタイミング

`ConnectionManager` は接続を作成するとき (最初の `getDataSource()` の呼び出し。通常は最初の
クエリ) に、`CakeEmail` は設定を適用するときにマップを読みます。`config/database.php` と
`config/email.php` に対しては、プラグインのブートストラップで登録すれば間に合います。

## プラグインの拡張

この例は、プラグインを作るための一つの良い開始方法であって、他にも色んな方法があります。
通常のルールでは、つまりアプリケーションでできることは、プラグインでもできます。

まずは、Composer でサードパーティのライブラリを導入し、 cake console に新しい shell
を追加します。 さらに、利用者が自動で出来る、プラグインの機能をテストするためのテストケースを
作成する事を忘れないでください。

ContactManager の例だと、ContactsController 内に add/remove/edit/delete
アクションを作り、 Contact モデルに validation を作成し、contact 管理機能を追加します。
プラグインの改良の仕方もあなた次第で決めれます。 コミュニティ内でコード共有を忘れないので
ください。 その誰もが、あなたの素晴らしい、再利用可能なコンポーネントの恩恵を受けることができます！

## プラグイン Tips

一度、プラグインを `plugins/` にインストールすると、
/plugin_name/controller_name/action というURLでアクセスできます。
ContactManager の例だと、ContactsController には
/contact_manager/contacts でアクセスできます。

CakePHP アプリケーションで動作するプラグインの最後の tips です。

- \[Plugin\]AppController と \[Plugin\]AppModel が無ければ、
  プラグインコントローラにアクセスしようとすると、 missing Controller エラーになります。
- プラグインのレイアウトは定義可能で、 `plugins/[Plugin]/View/Layouts` (モダンな構成では `templates/Layouts`) に含まれます。
  一方でプラグインは、デフォルトは `templates/Layouts` フォルダからレイアウトを利用します。
- コントローラ内で `$this->requestAction('/plugin_name/controller_name/action');`
  と書くと 内部プラグインとコミュニケーションができます。
- requestAction を使う際は、コントローラ名とモデル名がユニークであることを確認してください。
  そうしないと、”redefined class ...” エラーが発生します。
- 拡張子であなたのプラグインへのルーティングを追加するとき、アプリケーションのルーティングを
  上書きせずに、必ず `Router::setExtensions()` を使用してください。

## プラグインの公開 {#publish-your-plugin}

プラグインを共有するには、 `composer.json` ファイルを作成し、あなたのプラグインを
[packagist.org](https://packagist.org/) に公開します。
これで Composer を通して簡単にあなたのプラグインが使用できるようになります。`composer.json` では次のように設定します。

- パッケージの `type` を `cakephp-plugin` にします。
- アプリケーションのプラグインディレクトリにインストールされるように、`composer/installers` を require に追加します。
- `installer-name` にプラグインのフォルダ名を指定します。
- Composer がプラグインのクラスを読み込むように、`autoload` の classmap にクラスのあるフォルダを並べます。

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

classmap には、プラグインにあるフォルダだけを並べてください。無いフォルダがあると、
`composer dump-autoload` がエラーで止まります。モダンな構成の `src/` にある名前空間のあるクラスには、PSR-4 の設定を追加します。
[クラスの読み込み](/ja/development/autoloading#plugins) を参照してください。

パッケージ名にセマンティックな意味のある名前を選んでください。これは、理想を言えば、
"cakephp" をフレームワークとして依存関係を設定するべきです。
ベンダー名は、通常あなたの GitHub ユーザー名になります。
CakePHP 名前空間 (cakephp) を **使用しない** でください。
これは、CakePHP 自身のプラグインのために予約されています。
小文字と区切り文字のダッシュを使用することが決まりです。

もし、あなたの GitHub アカウントが "FooBar" で "Logging" プラグインを作成する場合、
`foo-bar/cakephp-logging` と名付けるといいでしょう。
そして、CakePHP 自身の "Localized" プラグインは、 `cakephp/localized` で見つけられます。
その PHP 8 対応フォークは [pieceofcake2/localized](https://pieceofcake2.github.io/localized/ja/) でメンテナンスされています。
