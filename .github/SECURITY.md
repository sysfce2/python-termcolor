# Security policy

## Reporting a vulnerability

To report a security issue, either report it privately
[on GitHub](https://github.com/termcolor/termcolor/security) (preferred) or through the
[Tidelift security contact](https://tidelift.com/security). Do not open a public issue.

## Supported versions

Security fixes are only applied to the latest release.

## Incident response plan

### 1. Triage

- Acknowledge the report to the reporter.
- Reproduce and assess severity.
- Open a private GitHub Security Advisory if confirmed.
- If necessary, yank the affected versions on PyPI (`pip install termcolor` will skip
  yanked versions, but explicit pins still work).
- If the release contains malicious code (for example, supply chain compromise), delete
  it from PyPI entirely so it cannot be installed at all.

### 2. Fix

- Develop a fix in a private fork via the Security Advisory.
- Include a regression test.
- Request a CVE through the GitHub Advisory if applicable.

### 3. Release and disclosure

- Merge the fix and publish a patch release to PyPI.
- Publish the GitHub Security Advisory to notify users.
- Credit the reporter (unless they prefer anonymity).
- Include details in the release notes.

## Threat model

termcolor is a pure-Python library with no dependencies and no network or file I/O. The
primary risk surface is the supply chain, not the library code itself.

**Supply chain & release integrity:**

- [Trusted Publishing](https://docs.pypi.org/trusted-publishers/) (OIDC) is used for
  PyPI uploads — no long-lived API tokens exist.
- Only the `deploy.yml` workflow triggered by a GitHub Release can publish to PyPI.
- All third-party GitHub Actions are pinned to commit SHAs, not mutable tags.
- Workflows use `permissions: {}` by default, granting only `id-token: write` where
  needed.
- Checkout steps use `persist-credentials: false`.
- Published packages include [digital attestations](https://docs.pypi.org/attestations/)
  that can be verified with
  [`pypi-attestations verify`](https://pypi.org/project/pypi-attestations/), for
  example:
  ```bash
  pypi-attestations verify pypi --repository https://github.com/termcolor/termcolor pypi:termcolor-3.3.0-py3-none-any.whl
  ```

**Account security:**

- 2FA is required for all maintainers on both GitHub and PyPI.
- Trusted Publishing means PyPI account access alone cannot publish a release.

**GitHub Actions hardening:**

- Workflows do not interpolate PR-controlled values into `run:` steps.
- CI runs with no write permissions.
- The deploy job only triggers on `published` release events.
- Workflows are audited with [zizmor](https://github.com/woodruffw/zizmor).

**ANSI escape injection:**

- If user-controlled input is passed to `colored()`/`cprint()`, embedded escape
  sequences could manipulate terminal output. This is the caller's responsibility to
  sanitise.
