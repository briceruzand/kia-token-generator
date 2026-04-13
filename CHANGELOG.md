# Changelog

All notable changes to this project will be documented in this file.

## [v2.5.2] - 2026-04-07

### Fixed
- **macOS token not showing (main fix)** — after login, the script was opening a new tab for the OAuth redirect instead of navigating in the existing tab. On macOS (and sometimes Linux), this caused the authorization code to land in a tab the script wasn't monitoring. Now uses CDP `Page.navigate` to navigate the existing tab, preserving session cookies and keeping everything in one place.
- **Zombie process fix** — `chrome.wait()` is now called after `chrome.kill()` to prevent orphaned Chrome processes on macOS/Linux

### Added
- **Faster login detection** — added detection of landing on `www.kia.com` after login as an additional signal (previously only `java.util.NoSuchElementException` in page body was checked)
- **Chrome crash detection** — if the user closes the Chrome window during login or OAuth redirect, the script now exits immediately with a clear error instead of spinning for up to 5 minutes
- **Unexpected token format warning** — if the authorization code doesn't match the expected UUID.UUID.UUID pattern, a `[WARN]` message is printed before falling back to generic extraction

## [v2.5.1] - 2026-02-13

### Fixed
- **"Mismatched token redirect uri" (HTTP 400)** — the script was incorrectly capturing the authorization code from the login redirect URL (`kia.com`) instead of the OAuth redirect URL (`prd.eu-ccapi.kia.com:8080`)

### Added
- **Automatic login detection** — detects `java.util.NoSuchElementException` in the browser via CDP and proceeds automatically (no more manual "Y" confirmation)
- **Browser closes automatically** — as soon as the authorization code is captured, Chrome closes so it doesn't cover the terminal output
- **Window stays open** — after finishing, the script waits for Enter so the CMD window doesn't disappear (useful for `.exe` users)
- **EXE support** — can be built as a standalone `.exe` with PyInstaller (no Python needed to run)

### Changed
- **Redirect timeout increased** from 30s to 60s — the OAuth redirect to `prd.eu-ccapi.kia.com:8080` can be slow on some networks

## [v2.5.0] - 2026-02-13

### Changed
- Minor improvements and stability fixes

## [v2.1.0] - 2026-02-13

### Changed
- Minor improvements

## [v2.0.0] - 2026-02-13

### Changed
- **No more Selenium / ChromeDriver dependency** — uses Chrome's native DevTools Protocol (CDP) over WebSocket
- **Automatic locale detection** — login page language matches your system (override with `--locale pl`)
- **Dynamic state parameter** — no more hardcoded timestamps that could expire
- **Auto-install dependencies** — creates a local `.venv` and installs `requests` + `websocket-client` automatically
- **Retry with backoff** on token exchange errors (429, timeouts)
- **Works on Windows, macOS, and Linux** without extra setup beyond Chrome + Python
