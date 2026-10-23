# メンテナンスされているプラグイン

[Piece of Cake 2](https://github.com/pieceofcake2) プロジェクトは、フレームワークのフォークに加えて、
関連するリポジトリと、主要な CakePHP 2 プラグインのフォークをメンテナンスしています。
メンテナンスしているプラグインはすべて PHP 8.x に対応しており、CI で
[PHPStan](/ja/development/static-analysis) のレベル 5 によるチェックを行っています。
各名前はそのパッケージのドキュメントサイト (日本語版) にリンクしています。ドキュメントサイトがないものは
GitHub のリポジトリにリンクしています。「Markdown」は GitHub 上のドキュメント (日本語版) の Markdown ソースへのリンクで、
AI ツールから直接読み込めます。

## コアリポジトリ

- [CakePHP](https://github.com/pieceofcake2/cakephp) - CakePHP 2.x フレームワーク
- [App Skeleton](/ja/installation/application-skeleton) ([Markdown](https://github.com/pieceofcake2/docs/blob/2.x/docs/ja/installation/application-skeleton.md)) - アプリケーションスケルトン
- [App Legacy Skeleton](/ja/installation/application-skeleton-legacy) ([Markdown](https://github.com/pieceofcake2/docs/blob/2.x/docs/ja/installation/application-skeleton-legacy.md)) - グローバルなクラスと `.ctp` のテンプレートのアプリケーションスケルトン
- [Plugin Skeleton](https://github.com/pieceofcake2/plugin) - プラグインスケルトン
- [Bake Plugin](https://pieceofcake2.github.io/bake/ja/) ([Markdown](https://github.com/pieceofcake2/bake/blob/1.x/docs/ja/README.md)) - コード生成ツール
- [Upgrade Plugin](https://pieceofcake2.github.io/upgrade/ja/) ([Markdown](https://github.com/pieceofcake2/upgrade/blob/1.x/docs/ja/README.md)) - アップグレード用ユーティリティ（CakePHP 1.3 から 2.0）

## 開発ツール

- [PHPStan CakePHP](https://pieceofcake2.github.io/phpstan-cakephp2/ja/) ([Markdown](https://github.com/pieceofcake2/phpstan-cakephp2/blob/main/docs/ja/README.md)) - 静的解析用の PHPStan 拡張
  ([静的解析](/ja/development/static-analysis) を参照してください)
- [PHPCS CakePHP2](https://pieceofcake2.github.io/phpcs-cakephp2/ja/) ([Markdown](https://github.com/pieceofcake2/phpcs-cakephp2/blob/main/docs/ja/README.md)) - CakePHP 2 用の PHP_CodeSniffer 規約
  ([コーディング規約](/ja/contributing/cakephp-coding-conventions) を参照してください)
- [Panther](https://pieceofcake2.github.io/panther/ja/) ([Markdown](https://github.com/pieceofcake2/panther/blob/1.x/docs/ja/README.md)) - Symfony Panther による End-to-End テスト
  ([End-to-End テスト](/ja/development/e2e-testing) を参照してください)
- [Storybook](https://pieceofcake2.github.io/storybook/ja/) ([Markdown](https://github.com/pieceofcake2/storybook/blob/1.x/docs/ja/README.md)) - アプリケーションの Element のための Storybook
- [Report](https://pieceofcake2.github.io/report/ja/) ([Markdown](https://github.com/pieceofcake2/report/blob/1.x/docs/ja/README.md)) - ヘッドレス Chrome で印刷する A4 の PDF の報告書。Panther とセキュリティ診断で使用

## セキュリティ

- [Security Check](https://pieceofcake2.github.io/security-check/ja/) ([Markdown](https://github.com/pieceofcake2/security-check/blob/1.x/docs/ja/README.md)) - セキュリティ診断 (SCA、SAST、DAST) を 1 つのコマンドで
- [Security SCA](https://pieceofcake2.github.io/security-sca/ja/) ([Markdown](https://github.com/pieceofcake2/security-sca/blob/1.x/docs/ja/README.md)) - Composer のパッケージの既知の脆弱性を PDF の報告書に
- [Security SAST](https://pieceofcake2.github.io/security-sast/ja/) ([Markdown](https://github.com/pieceofcake2/security-sast/blob/1.x/docs/ja/README.md)) - Semgrep か PHPStan によるコードと設定の静的解析を PDF の報告書に
- [Security DAST](https://pieceofcake2.github.io/security-dast/ja/) ([Markdown](https://github.com/pieceofcake2/security-dast/blob/1.x/docs/ja/README.md)) - OWASP ZAP による動いているアプリケーションのスキャンを PDF の報告書に
- [Security Headers Plugin](https://pieceofcake2.github.io/security-headers/ja/) ([Markdown](https://github.com/pieceofcake2/security-headers/blob/1.x/docs/ja/README.md)) - すべてのレスポンスにセキュリティヘッダーを付ける。ほかのサイトや Google のタグを名前で許可できる Content Security Policy

## メンテナンスしているプラグイン

### デバッグ

- [DebugKit Plugin](https://pieceofcake2.github.io/debug_kit/ja/) ([Markdown](https://github.com/pieceofcake2/debug_kit/blob/2.x/docs/ja/README.md)) - デバッグツールバー

### インフラ

- [Aws Plugin](https://pieceofcake2.github.io/aws/ja/) ([Markdown](https://github.com/pieceofcake2/aws/blob/1.x/docs/ja/README.md)) - AWS SDK との連携
- [AsyncAws Plugin](https://pieceofcake2.github.io/async-aws/ja/) ([Markdown](https://github.com/pieceofcake2/async-aws/blob/1.x/docs/ja/README.md)) - AsyncAws との連携。Aws プラグインより軽量な代替
- [Bref Plugin](https://github.com/pieceofcake2/bref) - Bref による AWS Lambda へのデプロイ

### ログとモニタリング

- [Monolog Plugin](https://pieceofcake2.github.io/monolog/ja/) ([Markdown](https://github.com/pieceofcake2/monolog/blob/2.x/docs/ja/README.md)) - Monolog との連携
- [Sentry Plugin](https://pieceofcake2.github.io/sentry/ja/) ([Markdown](https://github.com/pieceofcake2/sentry/blob/1.x/docs/ja/README.md)) - エラーの追跡とモニタリング

### テンプレート

- [TwigView Plugin](https://pieceofcake2.github.io/twig-view/ja/) ([Markdown](https://github.com/pieceofcake2/twig-view/blob/2.x/docs/ja/README.md)) - Twig テンプレートエンジン
- [CsvView Plugin](https://pieceofcake2.github.io/csv-view/ja/) ([Markdown](https://github.com/pieceofcake2/csv-view/blob/2.x/docs/ja/README.md)) - CSV 用のビュークラス

### ビューヘルパー

- [Markdown Plugin](https://pieceofcake2.github.io/markdown/ja/) ([Markdown](https://github.com/pieceofcake2/markdown/blob/2.x/docs/ja/README.md)) - Markdown ヘルパー
- [MenuBuilder Plugin](https://pieceofcake2.github.io/menu_builder/ja/) ([Markdown](https://github.com/pieceofcake2/menu_builder/blob/2.x/docs/ja/README.md)) - ナビゲーションメニューのヘルパー
- [Vite Plugin](https://pieceofcake2.github.io/vite/ja/) ([Markdown](https://github.com/pieceofcake2/vite/blob/1.x/docs/ja/README.md)) - 開発サーバーにも対応した Vite のアセットヘルパー
- [WebpackEncore Plugin](https://pieceofcake2.github.io/webpack-encore/ja/) ([Markdown](https://github.com/pieceofcake2/webpack-encore/blob/1.x/docs/ja/README.md)) - Webpack Encore のアセットヘルパー

### フォームとバリデーション

- [Xform Plugin](https://pieceofcake2.github.io/xform/ja/) ([Markdown](https://github.com/pieceofcake2/xform/blob/2.x/docs/ja/README.md)) - 拡張フォームヘルパー
- [Cakeplus Plugin](https://pieceofcake2.github.io/cakeplus/ja/) ([Markdown](https://github.com/pieceofcake2/cakeplus/blob/2.x/docs/ja/README.md)) - フォーム関連のユーティリティと機能拡張
- [Captcha Plugin](https://pieceofcake2.github.io/captcha/ja/) ([Markdown](https://github.com/pieceofcake2/captcha/blob/1.x/docs/ja/README.md)) - CAPTCHA ウィジェットの表示とサーバー側での検証。
  Cloudflare Turnstile (デフォルト)、Google reCAPTCHA v2/v3、hCaptcha に対応
- [Purifiable Plugin](https://pieceofcake2.github.io/purifiable/ja/) ([Markdown](https://github.com/pieceofcake2/purifiable/blob/1.x/docs/ja/README.md)) - HTML Purifier によるモデルのフィールドの HTML サニタイズ

### データベース

- [Datasources Plugin](https://pieceofcake2.github.io/datasources/ja/) ([Markdown](https://github.com/pieceofcake2/datasources/blob/2.x/docs/ja/README.md)) - 追加のデータソース（CSV / TSV ファイル、配列など）

### 認証と認可

- [Authenticate Plugin](https://pieceofcake2.github.io/authenticate/ja/) ([Markdown](https://github.com/pieceofcake2/authenticate/blob/1.x/docs/ja/README.md)) - 認証クラス
- [AclExtras Plugin](https://pieceofcake2.github.io/acl_extras/ja/) ([Markdown](https://github.com/pieceofcake2/acl_extras/blob/2.x/docs/ja/README.md)) - ACL テーブルを管理するコンソールツール
- [AclMenu Plugin](https://pieceofcake2.github.io/acl_menu/ja/) ([Markdown](https://github.com/pieceofcake2/acl_menu/blob/1.x/docs/ja/README.md)) - ACL で許可されたアクションのナビゲーションメニュー
- [Totp Plugin](https://pieceofcake2.github.io/totp/ja/) ([Markdown](https://github.com/pieceofcake2/totp/blob/1.x/docs/ja/README.md)) - ワンタイムパスワード (TOTP) による 2 要素認証

### ファイル

- [Upload Plugin](https://pieceofcake2.github.io/upload/ja/) ([Markdown](https://github.com/pieceofcake2/upload/blob/2.x/docs/ja/README.md)) - ファイルアップロードの処理
- [Filebinder Plugin](https://pieceofcake2.github.io/filebinder/ja/) ([Markdown](https://github.com/pieceofcake2/filebinder/blob/2.x/docs/ja/README.md)) - モデルのレコードへのファイルの添付。ローカルのディレクトリ、データベース、Amazon S3 に保存
- [Imagine Plugin](https://pieceofcake2.github.io/imagine/ja/) ([Markdown](https://github.com/pieceofcake2/imagine/blob/1.x/docs/ja/README.md)) - 画像処理ライブラリのラッパー
- [Image Plugin](https://pieceofcake2.github.io/image/ja/) ([Markdown](https://github.com/pieceofcake2/image/blob/1.x/docs/ja/README.md)) - アップロードされた画像の中身による検証 (JPEG、PNG、GIF、WebP、SVG)
- [CakePdf Plugin](https://pieceofcake2.github.io/cakepdf/ja/) ([Markdown](https://github.com/pieceofcake2/cakepdf/blob/1.x/docs/ja/README.md)) - PDF の生成

### SEO

- [Seo Plugin](https://pieceofcake2.github.io/seo/ja/) ([Markdown](https://github.com/pieceofcake2/seo/blob/6.x/docs/ja/README.md)) - 検索エンジン最適化
- [Sitemap Plugin](https://pieceofcake2.github.io/sitemap/ja/) ([Markdown](https://github.com/pieceofcake2/sitemap/blob/1.x/docs/ja/README.md)) - サイトマップの生成

### 国際化

- [Localized Plugin](https://pieceofcake2.github.io/localized/ja/) ([Markdown](https://github.com/pieceofcake2/localized/blob/2.x/docs/ja/README.md)) - ローカライズ用のユーティリティ

## プラグインのインストール

プラグインは Composer でインストールし、通常どおり `CakePlugin::load()` で読み込みます。
パッケージ名や設定方法は各プラグインのドキュメントサイトを参照してください。
[モダン構成](/ja/appendices/migrating-to-the-modern-layout) では、`installer-paths` の設定により
Composer がプラグインを `plugins/` にインストールします。

コンポーネントやヘルパーなど、設定に書いたプラグインのクラスは CakePHP が見つけます。
Composer が読み込まないプラグインのそのほかのクラスは、アプリケーションの classmap にプラグインのフォルダを追加してください。
[クラスの読み込み](/ja/development/autoloading#plugins) を参照してください。

## 各パッケージの由来 {#origins}

これらのパッケージの多くは、既存の CakePHP のプロジェクトを引き継いだものです。次の表は
各パッケージの由来をまとめたものです。「フォーク」は元のリポジトリの履歴を引き継いだもの、
「ベース」は元のプロジェクトから切り出したもの、「オリジナル」は pieceofcake2 が作成したものです。
各パッケージの README の Origin セクションにも同じ情報があり、LICENSE ファイルには
元の作者の著作権表示が残っています。

| パッケージ | 由来 | 元の作者 | 最後に取り込んだ上流のバージョン |
| --- | --- | --- | --- |
| [pieceofcake2/cakephp](https://github.com/pieceofcake2/cakephp) | [cakephp/cakephp](https://github.com/cakephp/cakephp) のフォーク。PHP 8 対応は [kamilwylegala/cakephp2-php8](https://github.com/kamilwylegala/cakephp2-php8) を取り込み | CakePHP コミュニティ (Cake Software Foundation、Inc.) | 2.10.24 |
| [pieceofcake2/app](https://github.com/pieceofcake2/app) | [cakephp/cakephp](https://github.com/cakephp/cakephp) がベース (`app/`) | CakePHP コミュニティ (Cake Software Foundation, Inc.) | 2.10.24 |
| [pieceofcake2/app-legacy](https://github.com/pieceofcake2/app-legacy) | [cakephp/cakephp](https://github.com/cakephp/cakephp) がベース (`app/`) | CakePHP コミュニティ (Cake Software Foundation, Inc.) | 2.10.24 |
| [pieceofcake2/plugin](https://github.com/pieceofcake2/plugin) | オリジナル | - | - |
| [pieceofcake2/bake](https://github.com/pieceofcake2/bake) | [cakephp/cakephp](https://github.com/cakephp/cakephp) がベース (BakeShell) | CakePHP コミュニティ (Cake Software Foundation, Inc.) | 2.10.24 |
| [pieceofcake2/upgrade](https://github.com/pieceofcake2/upgrade) | [cakephp/cakephp](https://github.com/cakephp/cakephp) がベース (UpgradeShell) | CakePHP コミュニティ (Cake Software Foundation, Inc.) | 2.10.24 |
| [pieceofcake2/phpstan-cakephp2](https://github.com/pieceofcake2/phpstan-cakephp2) | [sidz/phpstan-cakephp2](https://github.com/sidz/phpstan-cakephp2) のフォーク | Oleg Zhulnev | 0.2.0 |
| [pieceofcake2/phpcs-cakephp2](https://github.com/pieceofcake2/phpcs-cakephp2) | オリジナル | - | - |
| [pieceofcake2/panther](https://github.com/pieceofcake2/panther) | オリジナル | - | - |
| [pieceofcake2/debug_kit](https://github.com/pieceofcake2/debug_kit) | [cakephp/debug_kit](https://github.com/cakephp/debug_kit) のフォーク | Mark Story、CakePHP コミュニティ (Cake Software Foundation、Inc.) | 2.2.9 |
| [pieceofcake2/aws](https://github.com/pieceofcake2/aws) | オリジナル | - | - |
| [pieceofcake2/async-aws](https://github.com/pieceofcake2/async-aws) | オリジナル | - | - |
| [pieceofcake2/bref](https://github.com/pieceofcake2/bref) | オリジナル | - | - |
| [pieceofcake2/monolog](https://github.com/pieceofcake2/monolog) | [jadb/cakephp-monolog](https://github.com/jadb/cakephp-monolog) のフォーク | Jad Bitar | 2.0.0 |
| [pieceofcake2/sentry](https://github.com/pieceofcake2/sentry) | オリジナル | - | - |
| [pieceofcake2/twig-view](https://github.com/pieceofcake2/twig-view) | [predominant/TwigView](https://github.com/predominant/TwigView) のフォーク | Kjell Bublitz、Graham Weldon | v0.6 |
| [pieceofcake2/csv-view](https://github.com/pieceofcake2/csv-view) | [FriendsOfCake/cakephp-csvview](https://github.com/FriendsOfCake/cakephp-csvview) のフォーク | Jose Diaz-Gonzalez | 1.3.0 |
| [pieceofcake2/markdown](https://github.com/pieceofcake2/markdown) | [chronon/CakePHP-Markdown-Plugin](https://github.com/chronon/CakePHP-Markdown-Plugin) のフォーク | Gregory Gaskill | 2.1.0 |
| [pieceofcake2/menu_builder](https://github.com/pieceofcake2/menu_builder) | [torifat/cake-menu_builder](https://github.com/torifat/cake-menu_builder) のフォーク | Rifat Nabi | v2.0.3 |
| [pieceofcake2/vite](https://github.com/pieceofcake2/vite) | オリジナル | - | - |
| [pieceofcake2/webpack-encore](https://github.com/pieceofcake2/webpack-encore) | オリジナル | - | - |
| [pieceofcake2/xform](https://github.com/pieceofcake2/xform) | [ichikaway/xformHelper](https://github.com/ichikaway/xformHelper) のフォーク | Yasushi Ichikawa | 2.0.0 |
| [pieceofcake2/cakeplus](https://github.com/pieceofcake2/cakeplus) | [ichikaway/cakeplus](https://github.com/ichikaway/cakeplus) のフォーク | Yasushi Ichikawa | v2.0 |
| [pieceofcake2/captcha](https://github.com/pieceofcake2/captcha) | オリジナル | - | - |
| [pieceofcake2/datasources](https://github.com/pieceofcake2/datasources) | [cakephp/datasources](https://github.com/cakephp/datasources) のフォーク | CakePHP コミュニティ (Cake Software Foundation、Inc.) | コミット [`b741921`](https://github.com/cakephp/datasources/commit/b741921) (`master`) |
| [pieceofcake2/authenticate](https://github.com/pieceofcake2/authenticate) | [FriendsOfCake/Authenticate](https://github.com/FriendsOfCake/Authenticate) のフォーク | M. Ypes (Ceeram) | 1.0.2 |
| [pieceofcake2/acl_extras](https://github.com/pieceofcake2/acl_extras) | [markstory/acl_extras](https://github.com/markstory/acl_extras) のフォーク | Mark Story | 1.2.1 |
| [pieceofcake2/upload](https://github.com/pieceofcake2/upload) | [FriendsOfCake/cakephp-upload](https://github.com/FriendsOfCake/cakephp-upload) のフォーク | Jose Diaz-Gonzalez | 1.3.1 |
| [pieceofcake2/imagine](https://github.com/pieceofcake2/imagine) | [burzum/cakephp-imagine-plugin](https://github.com/burzum/cakephp-imagine-plugin) のフォーク | Florian Krämer | 1.0.6 |
| [pieceofcake2/cakepdf](https://github.com/pieceofcake2/cakepdf) | [FriendsOfCake/CakePdf](https://github.com/FriendsOfCake/CakePdf) のフォーク | Ceeram | 1.0.3 |
| [pieceofcake2/seo](https://github.com/pieceofcake2/seo) | [webtechnick/CakePHP-Seo-Plugin](https://github.com/webtechnick/CakePHP-Seo-Plugin) のフォーク | Nick Baker、Alan Blount、Matt Reishus | v6.1.0 |
| [pieceofcake2/sitemap](https://github.com/pieceofcake2/sitemap) | [loadsys/CakePHP-Sitemap](https://github.com/loadsys/CakePHP-Sitemap) のフォーク | Loadsys Web Strategies | 1.0.2 |
| [pieceofcake2/localized](https://github.com/pieceofcake2/localized) | [cakephp/localized](https://github.com/cakephp/localized) のフォーク | CakePHP コミュニティ (Cake Software Foundation、Inc.) | 2.7.0 |
| [pieceofcake2/docs](https://github.com/pieceofcake2/docs) | [cakephp/docs](https://github.com/cakephp/docs) のフォーク | CakePHP コミュニティ (Cake Software Foundation、Inc.) | コミット [`6b9d755`](https://github.com/cakephp/docs/commit/6b9d755) (`2.x`) |
| [pieceofcake2/docs-skeleton](https://github.com/pieceofcake2/docs-skeleton) | [cakephp/docs-skeleton](https://github.com/cakephp/docs-skeleton) のフォーク | CakePHP コミュニティ (Cake Software Foundation、Inc.) | コミット [`f256691`](https://github.com/cakephp/docs-skeleton/commit/f256691) (`main`) |

## コントリビューション

プロジェクトでは、バグ報告（特に PHP 8.x の互換性の問題）、セキュリティ脆弱性の報告、
プルリクエストを歓迎しています。
