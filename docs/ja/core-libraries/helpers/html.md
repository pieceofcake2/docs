# HtmlHelper

`class` **HtmlHelper**(View $view, array $settings = [])

CakePHP における HtmlHelper の役割は、 HTML に関連するオプションを
より簡単、高速に作成し、より弾力的なものに変えることです。
このヘルパーを使うことで、アプリケーションの足どりはより軽くなり、
そしてドメインのルートが置かれている場所に関して、よりフレキシブル
なものになるでしょう。

HtmlHelper にある多くのメソッドは `$options` という
引数を持っています。これにより、いかなる追加属性もタグに
付け加えることができます。これは `$options` を使う
方法についての簡単な例です。

```html
付けられる属性: <tag class="someClass" />
配列での指定: array('class' => 'someClass')

付けられる属性: <tag name="foo" value="bar" />
配列での指定:  array('name' => 'foo', 'value' => 'bar')
```

> [!NOTE]
> HtmlHelpler は既定ではすべてのビューで使うことができます。
> このヘルパーが存在しないという旨のエラーが発生したとき、
> たいていの原因はコントローラーで変数 `$helpers` を手動で
> 設定した際、名前を書き忘れたことです。

## Well-Formatted な要素の挿入

HtmlHelper の果たすもっとも重要なタスクは、適切に定義された
マークアップの生成です。 CakePHP はレンダリングと送信にかかる
CPU のサイクルを減らすために、ビューをキャッシュすることが
できます。この節では、いくつかの HtmlHelper のメソッドと、
その使用方法について説明します。

`method` HtmlHelper::**charset**($charset=null)

`method` HtmlHelper::**css**(mixed $path, array $options = [])

`method` HtmlHelper::**meta**(string $type, string $url = null, array $options = [])

`method` HtmlHelper::**docType**(string $type = 'xhtml-strict')

`method` HtmlHelper::**style**(array $data, boolean $oneline = true)

`method` HtmlHelper::**image**(string $path, array $options = [])

`method` HtmlHelper::**link**(string $title, mixed $url = null, array $options = [])

`method` HtmlHelper::**media**(string|array $path, array $options)

`method` HtmlHelper::**tag**(string $tag, string $text, array $options)

`method` HtmlHelper::**div**(string $class, string $text, array $options)

`method` HtmlHelper::** para(string $class, string $text, array $options)**()

`method` HtmlHelper::**script**(mixed $url, mixed $options)

`method` HtmlHelper::** scriptBlock($code, $options = [])**()

`method` HtmlHelper::**scriptStart**($options = [])

`method` HtmlHelper::**scriptEnd**()

`method` HtmlHelper::**nestedList**(array $list, array $options = [], array $itemOptions = [], string $tag = 'ul')

`method` HtmlHelper::**tableHeaders**(array $names, array $trOptions = null, array $thOptions = null)

`method` HtmlHelper::**tableCells**(array $data, array $oddTrOptions = null, array $evenTrOptions = null, $useCount = false, $continueOddEven = true)

`method` HtmlHelper::**url**(mixed $url = NULL, boolean $full = false)

`method` HtmlHelper::**useTag**(string $tag)

<a id="html5"></a>

## HTML5 のタグ

v2.12.11 から、HtmlHelper は HTML5 の script、link、style のタグを出力します。`type` 属性も CDATA も
付けません。

| 呼び出し | 出力 |
| --- | --- |
| `$this->Html->script('app')` | `<script src="/js/app.js"></script>` |
| `$this->Html->css('style')` | `<link rel="stylesheet" href="/css/style.css"/>` |
| `$this->Html->css('style', ['rel' => 'import'])` | `<style>@import url(/css/style.css);</style>` |
| `$this->Html->scriptBlock('init();')` | `<script>init();</script>` |

`application/xhtml+xml` で配信するページなど、XHTML の出力 (`type="text/javascript"`、`type="text/css"`、
CDATA で囲んだスクリプトブロック) にするには、ヘルパーの設定 `html5` を `false` にします。

```php
public $helpers = ['Html' => ['html5' => false]];
```

`scriptBlock()` と `scriptStart()` の `safe` オプションは、`true` のときスクリプトを CDATA で囲みます。デフォルトは
`false` で、`html5` が `false` のときは `true` です。`JsHelper` も同じ設定に従います。

## HtmlHelper が出力するタグの変更

`method` HtmlHelper::**loadConfig**(mixed $configFile, string $path = null)

## HtmlHelper を使ったパンくずリストの作成

`method` HtmlHelper::**getCrumbs**(string $separator = '&raquo;', string|array|bool $startText = false)

`method` HtmlHelper::**addCrumb**(string $name, string $link = null, mixed $options = null)

`method` HtmlHelper::**getCrumbList**(array $options = [], mixed $startText)
