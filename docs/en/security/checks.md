# Security Checks

The pieceofcake2 project has three security checks for CakePHP 2 applications. Each is a package of its own
that writes an A4 PDF report and fails CI when it finds a risk that reaches its threshold, and
[pieceofcake2/security-check](https://pieceofcake2.github.io/security-check/) runs the three in one command.
This page says when to run them. How to set them up, what each finding means and how to fix it are in the
documentation of each package.

| Check | Package | Finds |
| --- | --- | --- |
| SCA | [pieceofcake2/security-sca](https://pieceofcake2.github.io/security-sca/) | Known vulnerabilities (CVE) of the Composer packages of `composer.lock`, and abandoned packages |
| SAST | [pieceofcake2/security-sast](https://pieceofcake2.github.io/security-sast/) | Unsafe code (SQL built from strings, views of the request, `unserialize()` of the request, variables printed without `h()`, ...) and unsafe settings of production (`debug`, the salt, the cookie keys) |
| DAST | [pieceofcake2/security-dast](https://pieceofcake2.github.io/security-dast/) | What the running application does, with OWASP ZAP: missing security headers, cookie flags, error pages; with the full scan, XSS and SQL injections found by attacks |

```bash
composer require --dev pieceofcake2/security-check
vendor/bin/security-check
```

The settings of the three are kept in `config/security.php`. The checks need Docker (for ZAP, and for Semgrep
when SAST uses it) and Chrome to print the reports; see the
[documentation of security-check](https://pieceofcake2.github.io/security-check/).

## When to Run Them {#when}

| When | What | Why |
| --- | --- | --- |
| Each pull request | `vendor/bin/security-check`: SCA, SAST and the baseline scan of DAST | A change that adds a vulnerable package, unsafe code or a missing header fails before it is merged |
| Every week | SCA, and the full scan of DAST on the End-to-End tests (the example workflow of security-dast runs it every night) | New advisories are published for packages that did not change, and the full scan attacks every form the tests send, which takes too long for each pull request |
| Before each release | All three, with the full scan of DAST | The last check of what is released, with the reports kept as the record of the release |
| After a deployment to staging | The baseline scan of DAST on the server: `vendor/bin/security-dast --url=https://staging.example.com` (security-check has no `--url`) | The headers and the cookies of the real web server, which the built-in server of CI does not have |

The full scan attacks the application: run it only on a test environment, e.g. the built-in web server and the
test database of a CI job, never on production. The weekly SCA matters most: a package that was safe last week
can have a published vulnerability today.

The workflows of GitHub Actions for each case are in the documentation:
[security-check](https://pieceofcake2.github.io/security-check/ci) for the pull requests and the weekly run,
[security-dast](https://pieceofcake2.github.io/security-dast/ci) for the full scan.

## Before a Release {#before-release}

1. Update the packages (`composer update`) and run the checks with the full scan of DAST, on the code of the
   release:

   ```bash
   SECURITY_DAST_SCAN=full SECURITY_DAST_END_TO_END="vendor/bin/phpunit --testsuite EndToEnd" vendor/bin/security-check
   ```

2. Fix what reaches the threshold (High by default). For a finding that does not apply, record why:
   [an ignored advisory](https://pieceofcake2.github.io/security-sca/#ignoring-an-advisory) with its reason in
   `composer.json`, [a suppressed finding](https://pieceofcake2.github.io/security-sast/#suppressing-a-finding)
   with a comment, [an ignored alert](https://pieceofcake2.github.io/security-dast/report#ignoring-an-alert) with
   its reason in the settings. The reports list them with their reasons, so that they are checked again at the
   next release.
3. Keep the PDF reports with the release, e.g. as assets of the GitHub Release or in the records of the project:
   they show what was checked, when, and with which versions of the rules.

## From the Reports to Fixes {#fixes}

For an application that has never been checked, the first reports can be long. The documentation of
security-check gives the order to go through them, from the quickest and most urgent fixes:
[From the Reports to a Secure Application](https://pieceofcake2.github.io/security-check/#from-the-reports-to-a-secure-application).
The fixes of the settings are also in the [Hardening Checklist](/security/hardening).
