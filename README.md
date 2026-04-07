# Kia Token Generator

Generate a refresh token for the **Kia Connect API (Europe)** to use with the [kia_uvo](https://github.com/Hyundai-Kia-Connect/kia_uvo) Home Assistant integration.

> **⚠️ This tool works ONLY for European Kia accounts.**  
> For Hyundai EU, see: [hyundai_kia_connect_api#925](https://github.com/Hyundai-Kia-Connect/hyundai_kia_connect_api/issues/925)

## What changed in v2.5.2

- **Fixed: macOS token not showing (main fix)** — after login, the script was opening a **new tab** for the OAuth redirect instead of navigating in the existing tab. On macOS (and sometimes Linux), the new tab did not carry the session cookies, so the redirect failed silently and the authorization code was never captured. Now it navigates in the existing tab using `Page.navigate` via CDP, preserving the session
- **Faster login detection** — added detection of landing on `www.kia.com` after login as an additional signal (previously only `java.util.NoSuchElementException` in page body was checked)
- **Chrome crash detection** — if the user closes the Chrome window during login or OAuth redirect, the script now exits immediately with a clear error instead of spinning for up to 5 minutes
- **Unexpected token format warning** — if the authorization code doesn't match the expected UUID.UUID.UUID pattern, a `[WARN]` message is printed before falling back to generic extraction
- **Zombie process fix** — `chrome.wait()` is now called after `chrome.kill()` to prevent orphaned Chrome processes on macOS/Linux

## What changed in v2.5.1

- **Fixed: "Mismatched token redirect uri" (HTTP 400)** — the script was incorrectly capturing the authorization code from the login redirect URL (`kia.com`) instead of the OAuth redirect URL (`prd.eu-ccapi.kia.com`). Now it strictly waits for the correct redirect and ignores codes from wrong URLs
- **Automatic login detection** — detects `java.util.NoSuchElementException` in the browser via CDP and proceeds automatically (no more manual "Y" confirmation)
- **Browser closes automatically** — as soon as the authorization code is captured, Chrome closes so it doesn't cover the terminal output
- **Window stays open** — after finishing, the script waits for Enter so the CMD window doesn't disappear (useful for `.exe` users)
- **Redirect timeout increased** from 30s to 60s — the OAuth redirect to `prd.eu-ccapi.kia.com:8080` can be slow on some networks
- **EXE support** — can be built as a standalone `.exe` with PyInstaller (no Python needed to run)

### Earlier changes (v2.0)

- **No more Selenium / ChromeDriver dependency** — uses Chrome's native DevTools Protocol (CDP) over WebSocket
- **Automatic locale detection** — login page language matches your system (override with `--locale pl`)
- **Dynamic state parameter** — no more hardcoded timestamps that could expire
- **Auto-install dependencies** — creates a local `.venv` and installs `requests` + `websocket-client` automatically
- **Retry with backoff** on token exchange errors (429, timeouts)
- **Works on Windows, macOS, and Linux** without extra setup beyond Chrome + Python

## Requirements

| Component | Version |
|-----------|---------|
| Python    | 3.8+    |
| Chrome    | Any recent version (Chromium also works) |

That's it. No ChromeDriver, no Selenium, no manual pip installs.

If you use the `.exe` version — you only need Chrome. Python is not required.

## Quick start

### Windows (EXE — no Python needed)

Download `KIA_TOKEN.exe` from the [Releases](https://github.com/smalarz/kia-token-generator/releases) page and double-click it. No Python installation required.

> **Note:** Windows Defender / SmartScreen may warn about an unknown publisher. This is normal for unsigned executables. Click **"More info" → "Run anyway"**. You can verify the source code in this repository.

### Windows (Python)

```
python KIA_TOKEN.py
```

### macOS / Linux

```
python3 KIA_TOKEN.py
```

### What happens when you run it

1. First run only: creates a `.venv` and installs dependencies automatically
2. Chrome opens with the Kia login page
3. You log in and complete the CAPTCHA
4. The script **automatically detects** that login succeeded (it watches for `java.util.NoSuchElementException` in the browser — this is a normal Kia backend message)
5. Chrome closes automatically
6. The script exchanges the authorization code for a refresh token
7. Token is displayed in the terminal — copy it
8. Press Enter to close the window

> **💡 TL;DR:** Run the script → log in in Chrome → the script does the rest → copy the token.
>
> You can also press **Enter** in the terminal at any time to skip waiting for auto-detection.

### Command-line options

```
python KIA_TOKEN.py --locale de    # force German login page
python KIA_TOKEN.py --locale pl    # force Polish login page
python KIA_TOKEN.py --port 9333    # use different debugging port
python KIA_TOKEN.py --help
```

Supported locales: `de`, `en`, `fr`, `it`, `es`, `nl`, `pl`, `pt`, `cs`, `sk`, `hu`, `ro`, `bg`, `hr`, `sl`, `sv`, `no`, `da`, `fi`

## Using the token with Home Assistant

After getting your refresh token:

1. Go to **Settings → Devices & Services → + Add Integration**
2. Search for **Kia Uvo / Hyundai Bluelink**
3. Enter:
   - **Username**: your Kia Connect email address
   - **Password**: the **refresh token** from this script (NOT your Kia password)
   - **PIN**: leave **empty**
4. Submit — your car should appear

### Important notes for Home Assistant

- The refresh token is long (500–1000+ characters) — this is normal
- If the integration shows "unexpected error", generate a **fresh** token and try again
- After a Home Assistant **update**, the integration may need a new token
- The token expires after some time — run the script again when needed

## HAOS / Docker users

If you run Home Assistant OS (HAOS) or Home Assistant in Docker, you **do not** run this script inside HA. Run it on your **desktop/laptop** (Windows, macOS, or Linux with a browser), then copy the token into HA's integration setup.

### Step by step for HAOS users

1. On your **PC/Mac** (not on HA), download `KIA_TOKEN.exe` or `KIA_TOKEN.py`
2. Run it (double-click the `.exe`, or `python KIA_TOKEN.py` in terminal)
3. Log in to Kia in the Chrome window that opens
4. The script automatically detects login, closes the browser, and shows the token
5. Copy the refresh token from the terminal output
6. In HA: **Settings → Devices & Services → Add Integration → Kia Uvo**
7. Paste the refresh token as the password

You do **not** need to modify any files inside HA containers or replace `KiaUvoApiEU.py` if you are using the latest version of the kia_uvo integration from HACS.

## Security

```
┌─────────────────────────────────────────────────────────────┐
│  NEVER publish your refresh token or access token!          │
│  Anyone with your token has FULL ACCESS to your Kia         │
│  account and can control your car (lock/unlock, location,   │
│  climate, etc.)                                             │
│                                                             │
│  If you accidentally leak your token:                       │
│  1. Change your Kia Connect password immediately            │
│  2. Generate a new token                                    │
│  3. Update Home Assistant with the new token                │
└─────────────────────────────────────────────────────────────┘
```

## Building your own EXE (for nerds 🤓)

Don't want to install Python? Want a single `.exe` you can double-click and share with friends? You can build it yourself. Here's how — step by step, even if you've never done this before.

### What you need to install first

**1. Python** (only needed to build the `.exe` — not needed to run it afterwards)

- Go to [python.org/downloads](https://www.python.org/downloads/) and download the latest Python 3
- Run the installer
- ⚠️ **IMPORTANT: Check the box that says "Add Python to PATH"** at the bottom of the first screen — without this, nothing will work
- Click "Install Now" and wait for it to finish
- To verify: open **Command Prompt** (press `Win+R`, type `cmd`, press Enter) and type: `python --version`. You should see something like `Python 3.12.x`

**2. The script files** from this repository

- On this GitHub page, click the green **Code** button → **Download ZIP**
- Unzip the downloaded file to any folder (e.g. `C:\Users\YourName\Downloads\kia-token-generator`)

### Option A: Double-click to build (easiest)

1. Open the unzipped folder — you should see `KIA_TOKEN.py` and `build_exe.bat`
2. **Double-click `build_exe.bat`**
3. A black command window will open and start doing stuff — this is normal, don't close it
4. Wait about 1–2 minutes
5. When you see "Build complete!", you're done
6. Open the `dist` subfolder — there's your `KIA_TOKEN.exe` 🎉
7. You can copy this `.exe` anywhere — it's completely standalone, no Python needed to run it

### Option B: Type it yourself

If `build_exe.bat` doesn't work, or you like typing:

1. Open **Command Prompt** (`Win+R` → type `cmd` → press Enter)
2. Navigate to the folder with `KIA_TOKEN.py`:
   ```
   cd C:\Users\YourName\Downloads\kia-token-generator
   ```
   (replace with your actual path)
3. Run these commands one by one:
   ```
   python -m venv .build_venv
   .build_venv\Scripts\activate
   pip install pyinstaller requests websocket-client
   pyinstaller --onefile --name KIA_TOKEN --clean --noconfirm KIA_TOKEN.py
   ```
4. When it finishes, your `.exe` is in the `dist` folder

### Option C: Let GitHub build it for you

If you fork this repository on GitHub, the included GitHub Actions workflow (`.github/workflows/build-exe.yml`) builds the `.exe` automatically:

1. Fork this repo
2. Push a tag: `git tag v2.0.0 && git push --tags`
3. Go to **Actions** tab — wait for the build to finish
4. The `.exe` appears in the **Releases** section

### Build troubleshooting

**"python is not recognized as an internal or external command"**

You didn't check "Add Python to PATH" during installation. The easiest fix: reinstall Python from [python.org/downloads](https://www.python.org/downloads/) and this time **check the PATH checkbox**.

**SmartScreen / Windows Defender warning when running the built .exe**

This happens because the `.exe` is not digitally signed (signing costs money). Click **"More info"** → **"Run anyway"**. The source code is right here in this repository — you built it yourself.

**Antivirus flags the .exe as suspicious**

PyInstaller executables are sometimes falsely flagged by antivirus software. This is a well-known issue with PyInstaller, not a real threat. You can add an exception in your antivirus settings, or skip the `.exe` and just run `python KIA_TOKEN.py` directly.

## Troubleshooting / FAQ

### Browser shows `java.util.NoSuchElementException` after login

**This is normal and expected.** It means your login was successful. The error comes from Kia's Java backend — it has nothing to do with your script or your setup. The script detects this automatically and proceeds to fetch the token. If auto-detection doesn't trigger, press **Enter** in the terminal to continue manually.

### "Chrome not found"

The script auto-detects Chrome in standard locations. If you installed Chrome in a non-standard path, add it to your system PATH or create a symlink.

### Error: `code=` not found / "Authorization code not found"

**Most common cause:** the user-agent was not applied correctly, or the login was not fully completed.

The script sets the user-agent automatically via Chrome launch flags. It also prints `[DEBUG]` lines showing which URLs it finds and whether they match the expected redirect prefix (`prd.eu-ccapi.kia.com:8080`). If it still fails:

- Make sure you **fully complete** the login (CAPTCHA + credentials) and wait until `java.util.NoSuchElementException` appears
- Try a different network (VPN/firewall may block `prd.eu-ccapi.kia.com:8080`)
- Close **all** other Chrome windows before running the script (they may interfere with CDP)

### Error: "Existing Chrome debug session on port 9222"

Close all Chrome instances or use a different port:

```
python KIA_TOKEN.py --port 9333
```

### Error: "Token exchange failed" / HTTP 400

- The authorization code may have expired — run the script again
- Ensure you're using a **European** Kia account

### macOS: "Chrome cannot be opened because it is from an unidentified developer"

For Chromium installed via `brew`:

```
xattr -d com.apple.quarantine /Applications/Chromium.app
```

### Linux: "No graphical display detected"

This script needs a desktop environment. It cannot run on a headless server (e.g., inside a HA container or SSH without X forwarding). Run it on a machine with a monitor/display.

### Python version error on macOS

If you get `unsupported operand type(s) for |: 'type' and 'NoneType'`, your Python is older than 3.10. This script (v2.0) requires only 3.8+ and does not use `str | None` syntax, so this error should not occur. If it does, update Python:

```
brew install python
```

### Token expires — how often do I need to regenerate?

The token validity varies but is typically several hours to days. The HA integration should auto-refresh it. If the integration stops working, generate a new token.

### Does this work for Hyundai?

**No.** This script is for Kia EU only. For Hyundai EU, use one of these:

- [hyundai_kia_connect_api#925](https://github.com/Hyundai-Kia-Connect/hyundai_kia_connect_api/issues/925) — community solution with batch files
- [hyundai_kia_connect_api#959](https://github.com/Hyundai-Kia-Connect/hyundai_kia_connect_api/issues/959) — multiplatform CDP script (no Selenium)
- [Wiki: Hyundai Europe Login Flow](https://github.com/Hyundai-Kia-Connect/hyundai_kia_connect_api/wiki) (if available)

### I had `KiaFetchApiTokens.py` before — what's different?

The original `KiaFetchApiTokens.py` (by marvinwankersteen) requires manual steps: changing user-agent in Chrome DevTools, copying URLs between terminal and browser. This script automates those steps.

The Selenium-based `KiaFetchApiTokensSelenium.py` requires matching ChromeDriver versions, which caused most user issues. This script (v2.0) eliminates that entirely.

## How it works

1. Launches Chrome with `--remote-debugging-port` and a mobile user-agent (required by Kia's API)
2. Opens the Kia login page — you log in manually (CAPTCHA cannot be automated)
3. Monitors the browser via CDP — when it detects `java.util.NoSuchElementException` in the page content (= login complete), it proceeds automatically
4. Navigates the existing browser tab to the OAuth authorize endpoint via CDP `Page.navigate` (preserves session cookies)
5. Scans all open tabs for a redirect URL containing the authorization `code`, prioritizing the expected Kia API endpoint (`prd.eu-ccapi.kia.com:8080`)
6. Closes the browser (no longer needed)
7. Exchanges the code for `access_token` + `refresh_token` via HTTP POST

## License

MIT — use at your own risk. This tool is for personal use only.

## Related

- [kia_uvo](https://github.com/Hyundai-Kia-Connect/kia_uvo) — Home Assistant integration
- [hyundai_kia_connect_api](https://github.com/Hyundai-Kia-Connect/hyundai_kia_connect_api) — Python API library
- [Wiki: Kia Europe Login Flow](https://github.com/Hyundai-Kia-Connect/hyundai_kia_connect_api/wiki/Kia-Europe-Login-Flow) — official workaround documentation
