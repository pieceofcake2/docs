# CakePHP のフォルダ構造

アプリケーションの構成には、次の 2 つがあります。

- **モダンな構成** では、`config/`、`src/`、`templates/`、`tests/` などのフォルダをプロジェクトのルートに置きます。
  CakePHP 5 と同じ構成です。[pieceofcake2/app](/ja/installation/application-skeleton) スケルトンはこの構成です
  ([インストール](../installation) を参照)。
- **従来の `app/` 構成** では、コードを `app/` フォルダに置きます。CakePHP 2 の古いバージョンで作成したアプリケーションは、この構成です。

CakePHP は、アプリケーションがどちらの構成かを判別します。どちらの構成も、同じバージョンの CakePHP で動きます。

どちらの構成でも、CakePHP 本体と外部のライブラリは Composer が `vendor/` にインストールします。
このフォルダの中のファイルを勝手に編集 **しない** と誓ってください。
これはコア (心臓部分) で、ここをいじってしまうと、私たちはもう助けを差し伸べることができません。
代わりに [Application Extensions](../getting-started/cakephp-structure#application-extensions) の修正方法を調べてください。

## モダンな構成 {#the-modern-layout}

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

おもに作業するのは `src/`、`config/`、`templates/` です。

bin  
コンソールスクリプト `cake` が入ります。CakePHP やアプリケーションのシェルを実行します
(例: `bin/cake server`、`bin/cake test`)。
[シェルとタスクとコンソール](../console-and-shells) を参照してください。

config  
設定ファイルが入ります。データベース接続、ブートストラップ、コアの設定などです。
`config/define.php` では、アプリケーションの各フォルダのパスを定義します。

logs  
CakePHP のログファイルが書き込まれます。[ロギング](../core-libraries/logging) を参照してください。

plugins  
プラグインが入ります。Composer はプラグインをここにインストールします。

resources/locales  
国際化のための翻訳が入ります。

src  
アプリケーションの PHP のクラスが入ります。Composer が読み込みます。
[クラスの読み込み](../development/autoloading) を参照してください。

- `src/Console/Command/` にはシェルが、`Task/` にはシェルのタスクが入ります。
  `src/Console/Templates/` には、bake の出力を変えるためのテンプレートを置けます。
- `src/Controller/` にはコントローラが、`Component/` にはコンポーネントが入ります。
- `src/Lib/` には、自分で作ったライブラリが入ります。外部のライブラリと分けて置けます。
- `src/Model/` にはモデルが、`Behavior/` にはビヘイビアが、`Datasource/` にはデータソースが入ります。
- `src/View/Helper/` にはヘルパーが入ります。

templates  
テンプレートが入ります。

- ビューは、コントローラごとのフォルダに置きます (例: `templates/Posts/index.ctp`)
- エレメントは `Elements/` に置きます
- エラーページは `Errors/` に置きます
- レイアウトは `Layouts/` に置きます
- メールのテンプレートは `Emails/` に置きます

::: pieceofcake2 Added in version 2.13.0
`.php` のテンプレートと、CakePHP 5 のディレクトリ `layout/`、`element/`、`email/`、`Error/`。
[ビュー](../views#templates-in-the-cakephp-5-style) を参照してください。
:::

tests  
テストが入ります。`tests/TestCase/` はアプリケーションと同じ構成にして、クラスごとに 1 つ以上のテストケースを置きます。
`tests/Fixture/` にはフィクスチャを置きます。[テスト](../development/testing) を参照してください。

tmp  
CakePHP が一時的なデータを保管します。モデルの内容データ、ビューのキャッシュ、ときにはセッションなどです。
何を保管するかは設定しだいです。

このフォルダが存在し、書き込めるようにしてください。
そうしないと、アプリケーションのパフォーマンスがひどく落ちます。
デバッグモードでは、フォルダが無いか書き込めないときに CakePHP が警告します。

vendor  
Composer でインストールしたパッケージが入ります。CakePHP 本体 (`vendor/pieceofcake2/cakephp`) もここです。
ライブラリは `composer require` で追加します。ここにあるファイルを手で編集したり追加したりしないでください。

webroot  
開発時も運用時 (*production*) も、アプリケーションのドキュメントルートです。
Web サーバーで公開するのは、このフォルダだけにします。CSS スタイルシートや画像、JavaScript ファイルもここに置きます。

## 従来の `app/` 構成 {#the-legacy-app-layout}

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

各フォルダには、モダンな構成と同じものが入ります。

app/Config  
設定ファイルが入ります。`config/` にあたります。

app/Console  
`Command/` にシェルとシェルのタスクが、そしてコンソールスクリプト `cake` が入ります。

app/Controller、app/Lib、app/Model  
コントローラ、ライブラリ、モデルが入ります。`src/` の各フォルダにあたります。

app/Locale  
翻訳が入ります。`resources/locales/` にあたります。

app/Plugin  
プラグインが入ります。CakePHP は、プロジェクトのルートの `plugins/` からもプラグインを探します。

app/Test  
`Case/` にテストケースが、`Fixture/` にフィクスチャが入ります。

app/tmp  
一時的なデータが入ります。ログファイルは `logs/` に入ります。

app/Vendor  
Composer でインストールしない外部のライブラリのファイルが入ります。

app/View  
テンプレートが入ります。フォルダは `templates/` と同じです。
`View/Helper/` にはヘルパーが入るので、テンプレートとヘルパーのクラスが同じフォルダに入ります。

app/webroot  
ドキュメントルートです。

## 構成の対応 {#how-the-layouts-correspond}

| 従来の構成 (`app/`)         | モダンな構成                        |
|---------------------------|------------------------------------|
| `app/Config`              | `config/`                          |
| `app/Console`             | `src/Console/`                     |
| `app/Console/cake`        | `bin/cake`                         |
| `app/Controller`          | `src/Controller/`                  |
| `app/Lib`                 | `src/Lib/`                         |
| `app/Locale`              | `resources/locales/`               |
| `app/Model`               | `src/Model/`                       |
| `app/Plugin`              | `plugins/`                         |
| `app/Test/Case`、`app/Test/Fixture` | `tests/TestCase/`、`tests/Fixture/` |
| `app/tmp`                 | `tmp/`                             |
| `app/tmp/logs`            | `logs/`                            |
| `app/Vendor`              | `vendors/` (Composer のパッケージは `vendor/`) |
| `app/View` (`.ctp` ファイル) | `templates/`                     |
| `app/View/Helper`         | `src/View/Helper/`                 |
| `app/webroot`             | `webroot/`                         |

従来の構成のアプリケーションは、動かしながら少しずつモダンな構成に移せます。
[モダンな構成への移行](../appendices/migrating-to-the-modern-layout) を参照してください。
