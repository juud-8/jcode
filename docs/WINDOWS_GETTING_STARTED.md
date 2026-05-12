# Getting started with jcode on Windows

This guide assumes you already installed jcode (for example with the official PowerShell installer). If you have not installed yet, start with step 0.

## 0. Install (if you have not already)

Open **PowerShell 5.1+** and run:

```powershell
irm https://raw.githubusercontent.com/1jehuang/jcode/master/scripts/install.ps1 | iex
```

Optional flags (see [scripts/install.ps1](../scripts/install.ps1)):

- `-SkipAlacrittySetup` — do not try to install Alacritty via winget.
- `-SkipHotkeySetup` — do not configure Alt+`;` to launch jcode in Alacritty.

After install, **close and reopen your terminal** (or sign out and back in) so your **user** `PATH` picks up the launcher directory.

## 1. Confirm jcode is on PATH

The installer adds this directory to your user `PATH`:

```text
%LOCALAPPDATA%\jcode\bin
```

In a **new** PowerShell or Command Prompt window:

```powershell
jcode --version
```

You should see a version line (for example `jcode v0.12.1 (...)`). If `jcode` is not found, your session may still be using an old `PATH`; open a new terminal or check **Settings → System → About → Advanced system settings → Environment Variables** and confirm `jcode\bin` is listed under your user **Path**.

## 2. Understand what you are running

jcode is a **standalone** terminal coding agent. It is not a plugin inside the Claude Code desktop app or Codex. You use it by running `jcode` in a terminal (TUI) or with subcommands like `jcode run`.

## 3. Connect accounts you already use (Codex / Claude Code)

jcode can reuse credentials that other tools store on disk **without copying or rewriting** those files, after you approve which paths it may read.

### Paths on Windows

| Tool | Typical credential file |
|------|---------------------------|
| Codex CLI | `%USERPROFILE%\.codex\auth.json` |
| Claude Code CLI | `%USERPROFILE%\.claude\.credentials.json` |

Details and more providers: [OAUTH.md](../OAUTH.md).

### Option A — Interactive approval (recommended for most people)

1. Run `jcode` in a normal terminal (not piped or fully non-interactive).
2. When jcode finds existing Codex or Claude Code logins, it will ask you to **trust** reading those files in place.
3. Approve the sources you want. jcode remembers **path-bound** trust in `%USERPROFILE%\.jcode\config.toml`.

### Option B — Edit `config.toml` yourself (automation or headless setups)

If you need trust recorded without a prompt, add **canonical** Windows paths under `[auth]` in:

```text
%USERPROFILE%\.jcode\config.toml
```

The trust entry format is `source_id|full_path` (lowercase path segment after `|` is fine). jcode resolves paths with the same rules as the rest of the app; on Windows the stored canonical path may look like `\\?\c:\users\...`.

**Avoid** setting `JCODE_TRUSTED_EXTERNAL_AUTH_SOURCES` to short paths that do not match jcode’s canonical resolution — that environment variable **overrides** file-based trust and wrong values can make jcode behave as if nothing were trusted. If you use it at all, prefer exact canonical paths or rely on `config.toml` and interactive approval instead.

### Optional Codex-only escape hatch

For legacy `~/.codex/auth.json` reads without path trust, jcode also honors `JCODE_ALLOW_CODEX_LEGACY_AUTH=1` (values like `1`, `true`, `yes`). Prefer explicit trust in `config.toml` or the interactive flow when possible.

## 4. Log in directly with jcode (if you have no existing CLI logins)

```powershell
jcode login --provider claude
jcode login --provider openai
```

Use the provider you want as your primary subscription. See the README “OAuth and Providers” section for others.

## 5. Verify authentication

Quick checks (non-interactive friendly):

```powershell
# All providers jcode thinks are configured (no live model smoke unless you omit flags)
jcode auth-test --all-configured --no-smoke --no-tool-smoke

# One provider at a time
jcode auth-test -p claude --no-smoke --no-tool-smoke
jcode auth-test -p openai --no-smoke --no-tool-smoke
```

If credential probes fail, run `jcode auth doctor -p <provider>` and follow the suggested next steps.

## 6. Run a one-shot prompt

```powershell
jcode run "Say hello in one short sentence."
```

If the default provider hits a subscription or usage error, try an explicit provider:

```powershell
jcode --provider claude run "Say hello in one short sentence."
jcode --provider openai run "Say hello in one short sentence."
```

## 7. Start the full TUI

```powershell
cd path\to\your\repo
jcode
```

From there you can start sessions, switch models, and use commands documented in the main [README.md](../README.md).

## 8. Optional: Alacritty and Alt+`;` hotkey

If you skipped those during install, run the installer again **without** `-SkipAlacrittySetup` and `-SkipHotkeySetup` if you want:

- Alacritty installed via winget (when winget is available).
- **Alt+`;** opening jcode inside Alacritty (requires Alacritty).

## 9. Logs and config (when something goes wrong)

| What | Where |
|------|--------|
| Daily logs | `%USERPROFILE%\.jcode\logs\` (files like `jcode-YYYY-MM-DD.log`) |
| Main config | `%USERPROFILE%\.jcode\config.toml` |
| jcode-owned Claude OAuth | `%USERPROFILE%\.jcode\auth.json` |
| jcode-owned OpenAI OAuth | `%USERPROFILE%\.jcode\openai-auth.json` |

Repository note for contributors: [docs/WINDOWS.md](WINDOWS.md) describes Windows transport and install layout under `%LOCALAPPDATA%\jcode\`.

## 10. Telemetry (optional)

Release builds may show a short notice about anonymous usage statistics. To opt out:

```powershell
setx JCODE_NO_TELEMETRY 1
```

Open a new terminal afterward. Details: [TELEMETRY.md](../TELEMETRY.md).

## Further reading

- [README.md](../README.md) — features, providers, detailed install copy-paste block.
- [OAUTH.md](../OAUTH.md) — where credentials live and how jcode uses them.
- [AGENTS.md](../AGENTS.md) — contributor workflow and path conventions.
