# OpenCode + Chrome DevTools MCP (WSL -> Windows Chrome)

This repo documents a **reliable** workflow to control **Windows Chrome** from **WSL** using the Chrome DevTools Protocol (CDP). It includes:

- `Chrome_workflow_Setup.txt`: PowerShell + WSL launch steps
- `skills/chrome-wsl-bridge/SKILL.md`: streamlined skill instructions for agents

## Why this exists
Automation from WSL to Windows Chrome is brittle if you use a live Chrome profile or PowerShell quoting goes wrong. We hit repeated failures:

- CDP endpoint not reachable because Chrome did **not** launch with debug flags
- Profile locks when using the main Chrome profile (Profile 3)
- "Failed to Create Data Directory" when running PowerShell as Administrator
- Quoting and environment variable expansion breaking when called from bash

The fix was:

- Use a **dedicated CDP profile folder** (persistent)
- Launch Chrome with **cmd.exe** from WSL to avoid PowerShell quoting issues
- Keep CDP on **port 9333** and only switch if it actually fails

## Quick Start (PowerShell)
Run in **Windows PowerShell (non-admin)**:

```
1) taskkill /F /IM chrome.exe

2) $cdpDir = "$env:LOCALAPPDATA\ChromeCDPProfile"
   if (-not (Test-Path $cdpDir)) { New-Item -ItemType Directory -Force -Path $cdpDir | Out-Null }
   icacls "$cdpDir" /inheritance:e /grant "$($env:USERNAME):(OI)(CI)F" | Out-Null

3) Start-Process -FilePath "C:\Program Files\Google\Chrome\Application\chrome.exe" -ArgumentList "--remote-debugging-port=9333","--remote-debugging-address=127.0.0.1","--user-data-dir=$cdpDir","--profile-directory=Default","--no-first-run","--no-default-browser-check","about:blank"

4) curl.exe http://127.0.0.1:9333/json/version
```

If step 4 returns JSON, CDP is live.

## WSL Launch (agent-safe)
Run from WSL:

```
cmd.exe /c taskkill /F /IM chrome.exe
cmd.exe /c start "" "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9333 --remote-debugging-address=127.0.0.1 --user-data-dir="%LOCALAPPDATA%\ChromeCDPProfile" --profile-directory=Default --no-first-run --no-default-browser-check about:blank
curl -s --connect-timeout 5 http://127.0.0.1:9333/json/version
```

## OpenCode MCP config
In `~/.config/opencode/opencode.json` set the Chrome DevTools MCP to **port 9333**:

```
"chrome-devtools": {
  "type": "local",
  "enabled": true,
  "timeout": 150000,
  "command": [
    "node",
    "/home/vic/.local/share/chrome-devtools-mcp/node_modules/chrome-devtools-mcp/build/src/index.js",
    "--browserUrl",
    "http://127.0.0.1:9333",
    "--viewport=1280x720"
  ]
}
```

### Port fallback rule
- **Stay on 9333** by default.
- Only switch to **9334** if 9333 fails to return JSON after a retry.

## Common issues and fixes
- **Failed to Create Data Directory**
  - Use **non-admin PowerShell** and the dedicated CDP profile.
- **CDP not reachable**
  - Confirm Chrome launched with `--remote-debugging-port=9333`.
- **Profile not staying signed in**
  - Do NOT delete the CDP profile folder between runs.

## Files in this repo
- `Chrome_workflow_Setup.txt`
- `skills/chrome-wsl-bridge/SKILL.md`

## Security note
Do not commit API keys or tokens from `opencode.json`. Redact secrets before sharing configs.
