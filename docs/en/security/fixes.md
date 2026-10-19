# Security Fixes

The CakePHP team stopped releasing security fixes for CakePHP 2.x when it
reached End of Life in June 2021. The pieceofcake2 fork continues to fix
vulnerabilities that affect CakePHP 2.x. This page lists the vulnerabilities
fixed in the fork and other security-related changes, and explains what you
need to do in your application.

To report a vulnerability, see
[Reporting a Vulnerability](/security/reporting). For the status of upstream
CakePHP advisories, see [Upstream Advisories](/security/upstream-advisories).

## Fixed Vulnerabilities

| ID | Summary | Fixed in |
|----|---------|----------|
| [CVE-2015-8379](https://nvd.nist.gov/vuln/detail/CVE-2015-8379) | CSRF protection bypass via the `_method` parameter and custom HTTP methods | 2.11.0 |
| [CVE-2020-15400](https://nvd.nist.gov/vuln/detail/CVE-2020-15400) | CSRF token fixation | 2.11.0 |
| [GHSA-wgww-24w3-wfqp](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-wgww-24w3-wfqp) | SSRF: `Xml::build()` fetched `https://` URLs even with `'readFile' => false` | 2.12.3 |
| [GHSA-4f9q-3c4x-52cv](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-4f9q-3c4x-52cv) | Email header injection in `SmtpTransport` and `MailTransport` (CakePHP 2 equivalent of CVE-2026-77634) | 2.12.5 |
| [GHSA-w2mh-x92p-5p95](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-w2mh-x92p-5p95) | Path traversal in view, layout and element names (CakePHP 2 equivalent of CVE-2026-48820) | 2.12.5 |
| [GHSA-x4p9-p4cx-gxwx](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-x4p9-p4cx-gxwx) | Open redirect after login via a protocol-relative `Auth.redirect` URL | 2.12.5 |
| [GHSA-65v5-h65v-8pgg](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-65v5-h65v-8pgg) | Weak `'cipher'` encryption and `Security.cipherSeed`: the default `CookieComponent` type is not secure, and most long seeds become seed 0 (deprecated with warnings; switch to `'aes'`) | 2.12.6 |
| [GHSA-wh43-gc93-wh4c](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-wh43-gc93-wh4c) | Form action and redirects to another host via a request URL that starts with `//` (`CakeRequest::here()` is protocol-relative) | 2.12.11 |

All of these vulnerabilities are present in the original CakePHP 2.10.24, and
the CakePHP team no longer releases fixes for it. If your application still
uses `cakephp/cakephp`, you can get the fixes by switching to
`pieceofcake2/cakephp`, which declares `replace` for `cakephp/cakephp`. It
requires PHP 8.0 or newer; see
[Migrating from CakePHP 2.10](/appendices/migrating-from-cakephp-2-10).

### CVE-2015-8379: CSRF bypass with the `_method` parameter

`SecurityComponent` only validated the CSRF token for POST, PUT, DELETE and
PATCH requests. A request that used another HTTP method, either directly or
by overriding the method with the `_method` parameter, skipped CSRF
validation.

The fork now validates the CSRF token for every request except GET, HEAD and
OPTIONS. The fix is a backport of an upstream CakePHP patch
([c0fb45e](https://github.com/pieceofcake2/cakephp/commit/c0fb45e79)), and tests
were added in [PR #6](https://github.com/pieceofcake2/cakephp/pull/6).

**What you need to do:** nothing, unless your application relies on custom
HTTP methods being exempt from CSRF validation. Such requests now need a valid
CSRF token, or the action must be listed in `SecurityComponent::$unlockedActions`.

### CVE-2020-15400: CSRF token fixation

CSRF tokens were random values stored in the session. An attacker who could
write to the session, for example through an XSS vulnerability, could fixate
a token and use it for CSRF attacks.

The fork generates HMAC-signed tokens, as CakePHP 3.x and 4.x do
([PR #5](https://github.com/pieceofcake2/cakephp/pull/5)). A token consists of
16 random bytes followed by an HMAC-SHA1 signature of those bytes computed with
`Security.salt`, encoded with base64. A token that was not signed with your
application's salt is rejected, and signatures are compared in constant time.

**What you need to do:** nothing in your code. Tokens issued before the
upgrade are still accepted until they expire. Make sure that each application
uses its own secret `Security.salt` in `config/core.php`, and not the value
shipped with the skeleton, because the signature depends on it.

### SSRF in `Xml::build()` with `'readFile' => false` {#xml-build-ssrf}

`Xml::build()` reads a local file or fetches a URL when its input is a file
path or a URL. The `'readFile' => false` option disables this so that user
input can be passed safely. Because of a missing pair of parentheses in the
condition, however, `https://` URLs were still fetched with
`'readFile' => false`. An attacker who controls the input could make the
server send requests to any `https://` URL (server-side request forgery).
`http://` URLs and local files were not affected.

The bug was introduced in CakePHP 2.6.6 together with the `readFile` option
([995d8d2](https://github.com/cakephp/cakephp/commit/995d8d22c6)) and is
present in every later 2.x release. It was fixed in 2.12.3
([PR #33](https://github.com/pieceofcake2/cakephp/pull/33)). With
`'readFile' => false`, neither `http://` nor `https://` URLs are fetched, and
an `XmlException` is thrown instead. See the security advisory [GHSA-wgww-24w3-wfqp](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-wgww-24w3-wfqp).

**What you need to do:** update to 2.12.3 or later. Pass `'readFile' => false`
whenever the input may come from users. If your code relied on fetching an
`https://` URL with `'readFile' => false`, remove the option (it defaults to
`true`) or fetch the document yourself, for example with `HttpSocket`, and
pass the XML string.

### Email header injection in `SmtpTransport` and `MailTransport` {#email-header-injection}

`SmtpTransport` wrote the message headers into the SMTP `DATA` command without
removing line breaks from header values. `MailTransport` removed `\r\n`
sequences from header values, the subject and the recipient, but not a bare
`\r` or `\n`, which many mail servers also treat as a line break. If an
application put user input into an email header, for example a custom header
set with `CakeEmail::setHeaders()` or `addHeaders()`, an attacker could add
headers such as `Bcc:` or start the message body.

This is the CakePHP 2 equivalent of CVE-2026-77634, which was fixed in CakePHP
4.x and 5.x. It was fixed in 2.12.5: `SmtpTransport` removes CR and LF bytes
from header names and values and keeps only folded header lines (a line break
followed by whitespace, as produced by the MIME encoding of long headers), and
`MailTransport` removes bare `\r` and `\n` as well. See the security advisory
[GHSA-4f9q-3c4x-52cv](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-4f9q-3c4x-52cv).

**What you need to do:** update to 2.12.5 or later. If your code put raw line
breaks into a header value to add more headers, pass one entry per header to
`CakeEmail::setHeaders()` or `addHeaders()` instead. Until you can update,
remove CR and LF from user input before you use it in header values, the
subject or addresses.

### Path traversal in view, layout and element names {#view-path-traversal}

`View` did not check that the file found for a view, layout or element name
was inside the view paths. When an application built such a name from user
input, for example `$this->element('widgets/' . $type)`, a name containing `..`
could resolve to another file on the server, which was then included and run
as a PHP template.

This is the CakePHP 2 equivalent of CVE-2026-48820, which was fixed in CakePHP
4.x and 5.x. It was fixed in 2.12.5: when a name contains `..`, the real path
of the file must be inside one of the view paths (`App::path('View')` and the
plugin and theme view paths). Files outside them are treated as missing:
`element()` behaves as for a missing element, `elementExists()` returns
`false`, and views and layouts throw `MissingViewException` and
`MissingLayoutException`. Names that stay inside the view paths keep working.
See the security advisory
[GHSA-w2mh-x92p-5p95](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-w2mh-x92p-5p95).

**What you need to do:** update to 2.12.5 or later. If you render templates
from outside the view paths with names such as `../../other/file`, move those
templates into a view path. Do not build view, layout or element names from
user input, or check the input against an allow list first.

### Open redirect after login in `AuthComponent` {#auth-redirect}

When an unauthenticated user requests a protected action, `AuthComponent`
stores the requested URL in the `Auth.redirect` session key, and
`AuthComponent::redirectUrl()` returns it after a successful login. Browsers
treat a URL that starts with `//`, or with a backslash variant such as `/\` or
`\\`, as a URL on another host. A crafted link whose path started with `//`
could therefore send the user to an attacker's site after logging in (open
redirect).

There is no upstream advisory for CakePHP 2; the issue is similar to
CVE-2026-55590 in the `cakephp/authentication` plugin. It was fixed in 2.12.5:
`redirectUrl()` collapses the leading slashes and backslashes of the stored URL
into a single `/`, so the redirect always stays on the same host. See the
security advisory
[GHSA-x4p9-p4cx-gxwx](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-x4p9-p4cx-gxwx).

**What you need to do:** update to 2.12.5 or later; no code changes are
needed. Until you can update, check the value of `Auth.redirect` before
redirecting, and reject values that start with `//`, `/\`, `\/` or `\\`.

### Weak `'cipher'` encryption and `Security.cipherSeed` {#cipher}

`Security::cipher()`, which the default `'cipher'` type of `CookieComponent`
uses, XORs the value with numbers from PHP's Mersenne Twister generator seeded
with `Security.cipherSeed`. This is not real encryption: Mersenne Twister is
not a cryptographic generator, and `mt_srand()` keeps only 32 bits of the
seed, so there are only about 4.3 billion keystreams. Worse, every seed with
27 or more digits and most 26-digit seeds, including the default
`76859309657453542496749683645` of the original `config/core.php`, become
seed `0`, so all applications that use such a seed share the same seed. An
attacker who can read or forge `'cipher'` cookies may be able to recover or
tamper with their values.

This affects `pieceofcake2/cakephp` 2.12.5 and earlier, and every
`cakephp/cakephp` 2.x release. See the security advisory
[GHSA-65v5-h65v-8pgg](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-65v5-h65v-8pgg).

`'cipher'` cannot be made secure without breaking every existing cookie, so
2.12.6 does not change how values are encrypted. Instead it:

- deprecates `'cipher'`: the first use of the `'cipher'` type or of
  `Security::cipher()` in a process writes a `warning` to the log, also with
  debug off, and with debug on also triggers `E_USER_DEPRECATED`;
- warns about seeds outside 1 to 4294967295: the first `Security::cipher()`
  call in a process logs a `warning` (the seed is never logged), and with
  debug on `Debugger::checkSecurityKeys()` triggers an `E_USER_NOTICE`;
- sets the in-range default seed `'2659758734'` and `Security.useOpenSsl` in
  the fallback `config/core.php` of the core;
- adds the `Cookie.type` and `Cookie.key` settings, which set the default type
  and key of `CookieComponent` for the whole application.

**What you need to do:** `'cipher'` remains insecure on 2.12.6 with any seed;
an in-range seed only avoids the collapse to `0`. Update to 2.12.6 or later and
switch to `'aes'`: set `Security.useOpenSsl` to `true`, `Cookie.type` to
`'aes'`, and `Cookie.key` to a random value of at least 32 bytes from an
environment variable, in `config/core.php`. Existing `'cipher'` cookies are
lost once when you switch. Replace your own calls to `Security::cipher()` with
`Security::encrypt()` and `Security::decrypt()`. On 2.12.5 and earlier, set the
type and key in the controller instead. See
[cipherSeed and the 'cipher' Cookie Type](/development/configuration#cipher-seed)
for the details and the migration steps.

### Protocol-relative `here` for request URLs that start with `//` {#request-here}

`CakeRequest` removed only the first slash of the request URL. A request to
`//example.com/posts` was routed like `/example.com/posts`, but
`CakeRequest::$here` and `here()` were the protocol-relative URL
`//example.com/posts`, which browsers treat as a URL on the host
`example.com`. As a result:

- `FormHelper::create()` without a `url` printed
  `<form action="//example.com/posts">`, so a form with personal data or file
  uploads was submitted to another host;
- `$this->redirect($this->request->here(false))` redirected to another host
  (open redirect);
- with a base path, `here(false)` of `/base//example.com/posts` had the same
  problem, and `/\example.com` worked like `//example.com`.

This affects `pieceofcake2/cakephp` 2.12.10 and earlier, every
`cakephp/cakephp` 2.x release and 3.0 to 3.3. CakePHP 3.4 and later are not
affected, because zend/laminas-diactoros collapses the leading slashes of the
path. The fix ([GHSA-wh43-gc93-wh4c](https://github.com/pieceofcake2/cakephp/security/advisories/GHSA-wh43-gc93-wh4c), [PR #90](https://github.com/pieceofcake2/cakephp/pull/90)) removes all the leading slashes and backslashes of
the request URL, so `CakeRequest::$url` never starts with `/` or `\` and
`here` starts with a single slash after the base path. `here()` also collapses
the leading slashes and backslashes of its result, so a `$here` changed by the
application can not become protocol-relative either. Routing is unchanged.

**What you need to do:** update to the release with the fix; no code changes
are needed. Until you can update, pass an explicit `url` to
`FormHelper::create()`, and do not redirect to `here()` without collapsing its
leading slashes, for example in `AppController::beforeFilter()`:

```php
if (preg_match('#^[/\\\\]{2}#', $this->request->here(false))) {
    return $this->redirect('/' . ltrim($this->request->url, '/\\'));
}
```

## Other Security Changes

### SSL/TLS certificate validation

The original CakePHP 2.x validated HTTPS certificates with a bundled
`lib/Cake/Config/cacert.pem` that was last updated in 2016. The fork removed
that file and uses [composer/ca-bundle](https://github.com/composer/ca-bundle)
([PR #15](https://github.com/pieceofcake2/cakephp/pull/15), fixed in 2.11.0):

- The system CA bundle is used when OpenSSL provides one (`openssl.cafile` or
  `openssl.capath`).
- Otherwise, the Mozilla CA bundle shipped with composer/ca-bundle is used. It
  is updated through Composer.

This applies to `CakeSocket` and `HttpSocket`. If you set
`context.ssl.cafile` or `context.ssl.capath` yourself, your setting is used.

**What you need to do:** update code that refers to the removed `cacert.pem`
file, and keep `composer/ca-bundle` up to date.

### Encryption on PHP 8 {#encryption-on-php-8}

The mcrypt extension is not bundled with PHP 7.2 and later; it is only
available from [PECL](https://pecl.php.net/package/mcrypt). Without it, `Security::encrypt()`,
`Security::decrypt()` and the `'aes'` type of `CookieComponent` only work when
OpenSSL is enabled:

```php
// config/core.php
Configure::write('Security.useOpenSsl', true);
```

Applications created from the [pieceofcake2/app](https://pieceofcake2.github.io/app/)
skeleton (v2.3.1 and later) already have this line. Add it to existing
applications.

`Security::rijndael()` and the `'rijndael'` cookie type always require mcrypt.
See the [Upgrade Guide](/appendices/migrating-from-cakephp-2-10#enable-openssl).

### CookieComponent `cipher` type {#cookiecomponent-cipher-type}

The default encryption type of `CookieComponent` is `'cipher'`, which is not
secure (see [Weak `'cipher'` encryption](#cipher) above). On v2.12.6 and later,
set `Cookie.type` and `Cookie.key` in `config/core.php` as described there. On
v2.12.5 and earlier, set `'aes'` with the `'type'` setting in `$components`:

```php
public $components = [
    'Cookie' => ['type' => 'aes'],
];
```

You can also call `$this->Cookie->type('aes');` in `beforeFilter()`.

::: pieceofcake2
The `'type'` setting in `$components` is honored since v2.12.3. In v2.12.2 and
earlier it was ignored and cookies were encrypted with `'cipher'`.

:::

::: warning
Changing the encryption type makes the existing cookies unreadable: they are
lost once, e.g. a "remember me" login. Switch with a release.
:::

### XML external entities

`Xml::build()` does not load `<!ENTITY` definitions unless you pass
`'loadEntities' => true`. Keep the default when you parse untrusted XML.

`Xml::build()` also reads a local file or fetches a URL when the input is a
path or URL. When you pass user input, set `'readFile' => false`
(see [above](#xml-build-ssrf)) or check that it is an XML string, for example
that it starts with `<`.
