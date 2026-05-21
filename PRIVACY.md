# Privacy Notice — DocExtractify

_Last reviewed: 2026-05-18_

## Summary

DocExtractify is designed to run **entirely on your local
machine**. By default, no document content, file metadata, or telemetry is
transmitted off the device.

> **Always Offline. Always Private.**

## What data is processed

| Category | Where it lives | How long |
|---|---|---|
| Files you upload / drag-and-drop | A per-request temporary directory (`%TEMP%\docconv_*`) | Deleted automatically when the HTTP response completes |
| Conversion outputs (`.md` / `.txt` / `.docx`) | The output folder you pick (defaults to the input file's folder) | Until you delete them |
| WinUI session history | In-memory only (`HistoryStore.Entries`) | Cleared when the app is closed |
| App settings (theme, port) | In-memory only | Cleared when the app is closed |
| `crash.log` (WinUI) | `%LOCALAPPDATA%\DocExtractify\` | Until you delete it. Contains stack traces — treat as sensitive |
| Python server logs | `stdout` of the Python process | Until the process exits |

## What data is **not** collected

* No telemetry, analytics, crash reporting, or remote logging is sent.
* No accounts, identifiers, or device fingerprints are created.
* No cookies, session tokens, or persistent browser state are set.

## Network connections

The service binds to `127.0.0.1` (loopback) by default. The only outbound
network calls happen if you:

1. **Provide an `api_url`** to the optional LLM-assisted image / Word
   converter. In that case, the file content (or extracted image bytes) is
   POSTed to the URL **you supplied**. The URL is validated to block
   private / loopback / cloud-metadata hosts as a basic SSRF safeguard,
   but you remain responsible for the data-handling practices of the LLM
   endpoint you choose.
2. **First-run model downloads** by `paddleocr`. These
   fetch OCR model weights from `paddleocr.bj.bcebos.com`.
   To run fully offline after first launch, pre-download
   the weights and copy them to `~\.paddleocr\`.

## Third-party dependencies that may phone home

| Package | Behaviour | How to disable |
|---|---|---|
| `paddleocr` | Downloads OCR weights on first use | Pre-cache weights, then run offline |

For maximum privacy, set in your shell before launching `api.py`:

```powershell
$env:HF_HUB_OFFLINE = "1"
$env:TRANSFORMERS_OFFLINE = "1"
```

## Your rights

Because no data leaves the device, there is no central record to access,
rectify, port, or erase. To remove all local data created by the app:

1. Delete the output folder(s) you used.
2. Delete `%TEMP%\docconv_*` (cleaned up automatically on success, but kept
   on hard crashes).
3. Delete `%LOCALAPPDATA%\DocExtractify\crash.log`.
4. (Optional) Delete `~\.paddleocr\` to remove cached OCR weights.

## Compliance posture

The default configuration is suitable for processing personal data on your
own device under most general-purpose privacy frameworks (GDPR data-subject
self-processing, CCPA personal use). For regulated workloads (PHI, PCI,
classified) you must:

* Run on a properly configured / accredited host.
* Disable the optional `api_url` LLM integration unless the endpoint is
  itself in scope of your accreditation.
* Encrypt the disk where temporary and output folders live.
* Keep `DOCCONV_DEBUG=0`.

## Contact

Questions about this notice: **DocExtractify@outlook.com**
