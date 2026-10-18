# セキュリティ診断

pieceofcake2 プロジェクトには、CakePHP 2 のアプリケーションのためのセキュリティ診断が 3 つあります。それぞれが独立した
パッケージで、A4 の PDF の報告書を書き、しきい値に達するリスクを見つけると CI を失敗させます。
[pieceofcake2/security-check](https://pieceofcake2.github.io/security-check/ja/) は 3 つを 1 つのコマンドで実行します。
このページでは、いつ実行するかを説明します。設定の方法、それぞれの指摘の意味と修正の方法は、各パッケージの
ドキュメントにあります。

| 診断 | パッケージ | 見つけるもの |
| --- | --- | --- |
| SCA | [pieceofcake2/security-sca](https://pieceofcake2.github.io/security-sca/ja/) | `composer.lock` の Composer のパッケージの既知の脆弱性 (CVE) と、開発が終了したパッケージ |
| SAST | [pieceofcake2/security-sast](https://pieceofcake2.github.io/security-sast/ja/) | 危ないコード (文字列で組み立てた SQL、リクエストで選ぶビュー、リクエストの `unserialize()`、`h()` を通さない変数の出力など) と、本番の危ない設定 (`debug`、salt、Cookie の鍵) |
| DAST | [pieceofcake2/security-dast](https://pieceofcake2.github.io/security-dast/ja/) | 動いているアプリケーションの振る舞い (OWASP ZAP)。セキュリティヘッダーや Cookie のフラグの不足、エラーページ。フルスキャンでは、攻撃で見つかる XSS や SQL インジェクション |

```bash
composer require --dev pieceofcake2/security-check
vendor/bin/security-check
```

3 つの設定は `config/security.php` に置きます。診断には、Docker (ZAP と、SAST で Semgrep を使うとき) と、報告書を
印刷する Chrome が必要です。[security-check のドキュメント](https://pieceofcake2.github.io/security-check/ja/) を
ご覧ください。

<a id="when"></a>

## いつ実行するか

| いつ | 何を | なぜ |
| --- | --- | --- |
| pull request のたび | `vendor/bin/security-check`。SCA、SAST、DAST のベースラインスキャン | 脆弱なパッケージ、危ないコード、ヘッダーの不足を足す変更を、merge の前に失敗させます |
| 毎週 | SCA と、End-to-End テストでの DAST のフルスキャン (security-dast の例の workflow は毎晩実行します) | 変えていないパッケージにも新しいアドバイザリが公開されます。フルスキャンはテストが送るすべてのフォームを攻撃するので、pull request のたびには時間がかかりすぎます |
| リリースの前 | 3 つすべてを、DAST はフルスキャンで | リリースするものの最後の確認です。報告書はリリースの記録として残します |
| ステージングへのデプロイのあと | サーバーでの DAST のベースラインスキャン。`vendor/bin/security-dast --url=https://staging.example.com` (security-check には `--url` がありません) | CI の組み込みサーバーにはない、実際の Web サーバーのヘッダーと Cookie を確かめます |

フルスキャンはアプリケーションを攻撃します。CI のジョブの組み込みの Web サーバーとテストのデータベースなど、テストの
環境だけで実行し、本番では実行しないでください。最も大事なのは毎週の SCA です。先週は安全だったパッケージにも、今日は
脆弱性が公開されていることがあります。

それぞれの GitHub Actions の workflow はドキュメントにあります。pull request と毎週の実行は
[security-check](https://pieceofcake2.github.io/security-check/ja/ci)、フルスキャンは
[security-dast](https://pieceofcake2.github.io/security-dast/ja/ci) をご覧ください。

<a id="before-release"></a>

## リリースの前に

1. パッケージを更新し (`composer update`)、リリースするコードで、DAST はフルスキャンにして診断を実行します。

   ```bash
   SECURITY_DAST_SCAN=full SECURITY_DAST_END_TO_END="vendor/bin/phpunit --testsuite EndToEnd" vendor/bin/security-check
   ```

2. しきい値 (デフォルトは High) に達するものを修正します。当てはまらない指摘は、理由を残してください。`composer.json`
   で理由と一緒に[除外したアドバイザリ](https://pieceofcake2.github.io/security-sca/ja/#アドバイザリを除外する)、
   コメントで[抑制した指摘](https://pieceofcake2.github.io/security-sast/ja/#指摘を除外する)、設定で理由と一緒に
   [除外したアラート](https://pieceofcake2.github.io/security-dast/ja/report#アラートを除外する)です。報告書に理由と
   一緒に載るので、次のリリースでもう一度確かめられます。
3. PDF の報告書をリリースと一緒に残します。GitHub の Release のアセットや、プロジェクトの記録などです。何を、いつ、
   どのバージョンのルールで確かめたかがわかります。

<a id="fixes"></a>

## 報告書から修正へ

一度も診断したことのないアプリケーションでは、最初の報告書が長くなることがあります。security-check のドキュメントに、
早く修正できて急ぐものからの進め方があります。
[報告書から安全なアプリケーションへ](https://pieceofcake2.github.io/security-check/ja/#報告書から安全なアプリケーションへ)
をご覧ください。設定の修正は [セキュリティチェックリスト](/ja/security/hardening) にもあります。
