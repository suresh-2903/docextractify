# Security Policy — DocExtractify

## Supported Versions

| Version | Supported |
|---------|-----------|
| `main`  | ✅ |
| older   | ❌ |

## Reporting a Vulnerability

**Please do not file public GitHub issues for security problems.**

Email the maintainers privately at **security@example.invalid** (replace with
the actual contact for your fork) with:

1. A description of the issue and the impact.
2. Steps to reproduce, ideally with a minimal proof-of-concept.
3. Affected component (`api.py`, `convert_*.py`, WinUI app, etc.) and version
   / commit.
4. Your name / handle for credit (optional).

You will receive an acknowledgement within **5 business days** and a remediation
plan within **15 business days**. Critical issues will be patched on a priority
basis. Coordinated disclosure is preferred — please give us **90 days** before
publication.

## Threat model

The project ships **three** runtime surfaces. Their threat models differ:

### 1. Local CLI (`convert_all.py`)
* Trust boundary: the user. The CLI processes files the user already trusts
  on the local filesystem. No network listener.
* Risks: maliciously crafted input documents (PDF / DOCX / XLSX) could
  trigger parser bugs in the underlying libraries (pdfplumber, python-docx,
  openpyxl, etc.). Keep dependencies up to date.

### 2. Local web service (`api.py`)
* Default bind: `127.0.0.1:8080` (loopback only). **Do not expose this
  service to the public internet without putting it behind an authenticated
  reverse proxy.**
* Built-in controls (see `api.py`):
  - `TrustedHostMiddleware` (`DOCCONV_ALLOWED_HOSTS`)
  - Optional CORS allow-list (`DOCCONV_ALLOWED_ORIGINS`, empty = same-origin)
  - Per-file size cap (`DOCCONV_MAX_UPLOAD_BYTES`, default 100 MB)
  - Per-request size cap (`DOCCONV_MAX_REQUEST_BYTES`, default 500 MB)
  - Per-request file-count cap (`DOCCONV_MAX_FILES_PER_REQ`, default 50)
  - Filename sanitisation (path-traversal, NUL-byte, control-char stripping)
  - Extension allow-list enforced server-side
  - SSRF guard on the optional `api_url` field (rejects loopback /
    RFC1918 / link-local / cloud-metadata hosts)
  - Hardening response headers (`X-Content-Type-Options`, `X-Frame-Options`,
    `Referrer-Policy`, `Cache-Control: no-store`, `Content-Security-Policy`)
  - Internal exception details suppressed unless `DOCCONV_DEBUG=1`
  - Per-request temporary directory cleaned up after the response is sent
* Out of scope:
  - Authentication / authorisation (single-user local app).
  - TLS termination — provide via reverse proxy if exposed beyond loopback.

### 3. WinUI 3 desktop app
* Trust boundary: the interactive Windows user.
* Talks only to `127.0.0.1` on the configured port.
* No telemetry, no analytics, no remote code execution paths.

## Operational hardening checklist

* [ ] Run on an OS with security updates current.
* [ ] Run inside a non-privileged user account.
* [ ] Keep Python dependencies pinned; refresh with `uv sync --upgrade` and
      audit with `uv pip list` + `pip-licenses`.
* [ ] Run `pip-audit` (or `uv pip audit` when stable) in CI:
      `pip-audit -r requirements.txt`.
* [ ] If exposing the service beyond localhost:
  * Front it with TLS (Caddy, Nginx, or Azure Application Gateway).
  * Add authentication (mTLS, API key, or AAD) at the proxy.
  * Lower the size caps via the `DOCCONV_MAX_*` env vars.
  * Set `DOCCONV_ALLOWED_HOSTS` and `DOCCONV_ALLOWED_ORIGINS` explicitly.
* [ ] Never set `DOCCONV_DEBUG=1` in production — it leaks exception text.
* [ ] Treat uploaded files as untrusted; do not open them outside this app
      without your own AV scanning.

## Known caveats

* All PDF dependencies (`pdfplumber`, `pypdfium2`) are permissively licensed
  (MIT / Apache-2.0). See [THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md).
* `crash.log` written by the WinUI app may contain stack traces; treat it
  as confidential and rotate / delete after triage.
