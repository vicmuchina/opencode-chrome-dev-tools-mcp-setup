---
name: chrome-wsl-bridge
description: Use Windows Chrome from WSL via Chrome DevTools Protocol with a stable CDP profile.
---
# Chrome DevTools via Windows Chrome (WSL)

## What this is
Reliable workflow to control **Windows Chrome** from **WSL** using CDP. This uses a **dedicated CDP profile** so Chrome actually opens the debug port consistently.

## Known-good workflow (streamlined, agent-safe)

### 0) One-time setup (only if profile does not exist)
Run from WSL. This creates the dedicated CDP profile directory once.
```
cmd.exe /c if not exist "%LOCALAPPDATA%\ChromeCDPProfile" mkdir "%LOCALAPPDATA%\ChromeCDPProfile"
```

### 1) Launch Chrome with CDP (from WSL or Linux shell)
Run these from WSL. Use `cmd.exe` only (no PowerShell) to avoid quoting/env issues.
```
if curl -s --connect-timeout 3 http://127.0.0.1:9333/json/version >/dev/null; then
  echo "CDP already running on 9333. Do NOT kill Chrome."
else
  cmd.exe /c taskkill /F /IM chrome.exe
  cmd.exe /c start "" "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9333 --remote-debugging-address=127.0.0.1 --user-data-dir="%LOCALAPPDATA%\ChromeCDPProfile" --profile-directory=Default --no-first-run --no-default-browser-check about:blank
fi
```

### 2) Verify in Windows (must return JSON)
```
curl.exe http://127.0.0.1:9333/json/version
```

### 3) WSL/OpenCode config
Set MCP Chrome DevTools endpoint to:
```
http://127.0.0.1:9333
```

### 4) Verify from WSL
```
curl -s --connect-timeout 5 http://127.0.0.1:9333/json/version
```

## Port fallback rule
- Stay on port `9333` by default.
- Do **not** auto-switch to `9334` under any circumstance.
- If `9333` fails twice, **ask the user** before changing ports or killing Chrome.

## Notes
- This profile is **persistent** at `C:\Users\<you>\AppData\Local\ChromeCDPProfile`.
- Sign in once; login persists for this profile.
- Do **not** run PowerShell as Administrator (can cause “Failed to Create Data Directory”).
- Do **not** delete the CDP profile on each run; keep it stable so sign-in persists.
- If the profile is new (first run), **pause and ask the user to sign in** before continuing automated steps.
