# セキュリティチェックリスト

CakePHP 2 のアプリケーションを本番環境で運用する前に、次の点を確認してください。
各項目には、設定を説明しているページへのリンクを付けています。

<a id="debug"></a>

## デバッグモードを無効にする

本番環境では `debug` を `0` にしてください。デバッグレベルが高いと、エラーメッセージ、スタックトレース、
SQL クエリが訪問者に表示されます。`pieceofcake2/app` スケルトンから作成したアプリケーションは
`APP_DEBUG` 環境変数からデバッグレベルを読み込み、設定されていない場合はレベル 0 を使います。
本番サーバーでは `APP_DEBUG` を設定しないか、`APP_DEBUG=0` を設定してください。[core.php の更新](/ja/deployment#update-core-php)
を参照してください。

<a id="salt"></a>

## 独自のソルトとシード値を使う

`Security.salt` と `Security.cipherSeed` には、スケルトンに含まれる値ではなく、アプリケーション固有の
秘密の値を設定してください。ソルトは CSRF トークンとフォーム改ざん防止のトークンの署名に使われ、
`Cookie.key` が未設定か空のときは `CookieComponent` の鍵にもなります。スケルトンは `SECURITY_SALT` と
`SECURITY_CIPHER_SEED` 環境変数から値を読み込みます。既定のソルトについての CakePHP の通知は、デバッグが有効で
`AuthComponent` を読み込んでいるときだけ出ます。本番で通知が出ないことは確認になりません。本番の設定は
[pieceofcake2/security-sast](https://pieceofcake2.github.io/security-sast/ja/) で確かめてください。[CakePHP のコア設定](/ja/development/configuration#cakephp-のコア設定)
の `Security.salt` と `Security.cipherSeed`、および [環境変数](/ja/development/configuration#environment-variables)
を参照してください。

運用中のアプリケーションでソルトを変更すると、それで署名やハッシュをしたものが使えなくなります。既定の `Simple`
パスワードハッシャーのパスワード (誰もログインできなくなります)、`Security::encrypt()` の値、`Session.checkAgent`
(既定で有効) のセッション、ソルトを鍵にしたクッキー、開いているフォームです。先にパスワードを `Blowfish` ハッシャーに
移してください。手順は [cakephp2-config-salt](https://pieceofcake2.github.io/security-sast/ja/rules#cakephp2-config-salt)
をご覧ください。

<a id="dependencies"></a>

## 依存パッケージを最新に保つ

セキュリティ修正は最新のリリースにだけ提供されます ([サポートするバージョン](/ja/security/#supported-versions)
を参照してください)。`pieceofcake2/cakephp`、プラグイン、その他のパッケージを Composer で定期的に更新し、
本番環境では開発用の依存パッケージを除いてインストールしてください。

```bash
composer update
composer install --no-dev --optimize-autoloader
```

使っているパッケージに脆弱性が公開されたことを知るには、
[pieceofcake2/security-sca](https://pieceofcake2.github.io/security-sca/ja/) を毎週実行してください。
[セキュリティ診断](/ja/security/checks) を参照してください。

`composer/ca-bundle` も最新に保ってください。このパッケージは、`HttpSocket` の HTTPS 接続で使う
CA 証明書を提供します。[脆弱性への対応](/ja/security/fixes) と
[本番用の依存パッケージのインストール](/ja/deployment#install-dependencies-for-production) を参照してください。

<a id="security-component"></a>

## フォームと CSRF に SecurityComponent を使う

フォームを扱うコントローラーでは [SecurityComponent](/ja/core-libraries/components/security-component)
を読み込んでください。フォームの改ざんを検出し、CSRF 対策 (`$csrfCheck`、既定で有効) では
GET・HEAD・OPTIONS 以外のすべてのリクエストでトークンを検証します。コントローラー全体でチェックを
無効にせず、必要なアクションやフィールドだけを対象外にしてください。

ビュー・レイアウト・エレメントの名前や `Xml::build()` への入力を、確認せずにユーザーの入力から
組み立てないでください ([脆弱性への対応](/ja/security/fixes) を参照してください)。

<a id="validate-input"></a>

## 入力を検証する

リクエストから受け取るすべての項目を、モデルの検証ルールでサーバー側で検証し、ユーザーのデータは
検証を有効にしたまま保存してください。許可リストと長さの上限を使い、`fieldList` で保存する項目を限定し、
v2.12.6 以降では `utf8` ルールで不正な UTF-8 を拒否してください。
[入力の検証](/ja/security/escaping-and-sanitizing-html#validate-input) と
[データバリデーション](/ja/models/data-validation) を参照してください。

<a id="escape-output"></a>

## 出力をエスケープし、HTML を無害化する

出力するすべての値を `h()` でエスケープし (テンプレートは自動ではエスケープしません)、URL や
JavaScript に入れる値はそれに合わせてエンコードし、ユーザーからの HTML は Symfony HtmlSanitizer や
HTML Purifier のような、メンテナンスされている無害化ライブラリに通してください。二重の守りとして
Content Security Policy も使ってください。
[出力時のエスケープと HTML の無害化](/ja/security/escaping-and-sanitizing-html) を参照してください。

<a id="cookies-and-sessions"></a>

## クッキーとセッションを保護する

- CakePHP は、セッションのクッキーに `session.cookie_httponly` を設定し、HTTPS のリクエストでは
  `session.cookie_secure` も設定します。これらの ini の値を自分で設定している場合は、その値が使われます。
  `session.cookie_samesite` などその他のセッションの ini の設定は、`Session.ini` で設定できます。
  [セッション](/ja/development/sessions) を参照してください。
- `CookieComponent` は、既定では `secure` と `httpOnly` のフラグを設定しません。クッキーを HTTPS でだけ
  送るように `$this->Cookie->secure = true;` を設定し、JavaScript から読む必要がないクッキーには
  `$this->Cookie->httpOnly = true;` を設定してください。[Cookie](/ja/core-libraries/components/cookie) を参照してください。
- クッキーは、既定の `'cipher'` ではなく `'aes'` で暗号化してください。`config/core.php` で、`Cookie.type` を
  `'aes'`、`Cookie.key` を `COOKIE_KEY` 環境変数から読む 32 バイト以上のランダムな値、`Security.useOpenSsl` を `true`
  にします (v2.12.6 以降)。スケルトン v2.4.1 以降から作成したアプリケーションには、すでにこれらの行があります。
  切り替えると、既存の `'cipher'` のクッキーは一度失われます。[`'cipher'` 暗号化の弱さ](/ja/security/fixes#cipher)
  を参照してください。

<a id="file-permissions"></a>

## ファイルのパーミッションを制限する

ドキュメントルートを `webroot/` ディレクトリに設定し、アプリケーションのコード、設定、`vendor/`
に URL でアクセスできないようにしてください。ウェブサーバーが書き込む必要があるのは `tmp/` と `logs/`
だけです。[ドキュメントルートの指定](/ja/deployment#set-document-root)、
[パーミッション](/ja/installation#permissions)、[URLリライティング](/ja/installation/url-rewriting)
を参照してください。

<a id="https"></a>

## HTTPS を使う

アプリケーションは HTTPS だけで提供してください。CakePHP が HTTPS を検出すると (`HTTPS` サーバー変数)、
セッションのクッキーに secure 属性が付きます。HTTPS をプロキシやロードバランサーで終端していると CakePHP は検出
できないので、本番の `Session.ini` に `'session.cookie_secure' => 1` を設定してください。`SecurityComponent::requireSecure()` を使うと、
アクションを HTTPS に限定できます。[アクションを SSL 通信に限定](/ja/core-libraries/components/security-component#アクションを-ssl-通信に限定)
を参照してください。TLS とリダイレクトはウェブサーバーで設定してください。
[URLリライティング](/ja/installation/url-rewriting) と [デプロイ](/ja/deployment) を参照してください。

<a id="php-version"></a>

## サポートされている PHP を使う

`pieceofcake2/cakephp` は PHP 8.0 から 8.5 で動作します ([システム要件](/ja/installation#システム要件)
を参照してください)。PHP プロジェクトからセキュリティ修正が提供されているバージョンの PHP を使ってください。
php.net の [Supported Versions](https://www.php.net/supported-versions.php) を参照してください。
