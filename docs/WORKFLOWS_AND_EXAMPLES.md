# jcode workflows, use cases, and examples

This page is a **cookbook**: concrete commands and patterns you can copy. For install and first-time Windows setup, see [WINDOWS_GETTING_STARTED.md](WINDOWS_GETTING_STARTED.md). For machine-readable CLI details, see [WRAPPERS.md](WRAPPERS.md).

Unless noted, examples work on **Linux, macOS, and Windows**. On Windows, paths like `~/.jcode` mean `%USERPROFILE%\.jcode`.

---

## 1. Day-to-day coding in the TUI

**Goal:** Work inside one repo with the full terminal UI (chat, tools, side panel, session list).

```bash
cd /path/to/your/project
jcode
```

**Typical flow:**

1. Start `jcode` from the repo root so the agent’s cwd and git context match the project.
2. Ask for a change in natural language; the agent uses built-in tools (read, edit, shell, grep, and so on).
3. Use **Shift+Enter** to queue a message until the current agent turn finishes; plain submit interleaves when safe for the model cache (see [README.md](../README.md) “Misc”).

**Pick provider or model for this launch only:**

```bash
jcode --provider claude
jcode --provider openai --model gpt-5.4
```

**Discover what you can pass in:**

```bash
jcode provider list
jcode model list
jcode provider current
```

---

## 2. One-shot “run this and exit” (no TUI)

**Goal:** Script a single question, CI helper, or quick answer from the shell.

```bash
jcode run "Summarize the purpose of src/main.rs in three bullets."
```

**Explicit provider** (useful if your default hits a quota or the wrong subscription):

```bash
jcode --provider claude run "Reply with exactly: OK"
jcode --provider openai run "Reply with exactly: OK"
```

**Skip update check** (faster, quieter for scripts):

```bash
jcode --no-update run "Say hello"
```

---

## 3. Automation: JSON or streaming NDJSON

**Goal:** Integrate jcode with another tool, a CI job, or a local script that parses structured output.

Recommended defaults for wrappers (from [WRAPPERS.md](WRAPPERS.md)):

```bash
jcode --quiet --no-update --no-selfdev run --json "Reply with exactly OK"
```

**Streaming** (line-delimited events: tool life cycle, text deltas, usage, done):

```bash
jcode --quiet --no-update run --ndjson "Explain what this repo does in one sentence."
```

Event shapes and `done` payload are documented in [WRAPPERS.md](WRAPPERS.md).

---

## 4. Resume a previous session

**Goal:** Continue an old conversation by name or id (including sessions that started in other harnesses when supported).

```bash
jcode --resume fox
```

In the TUI, use the session picker and **`/resume`**-style flows as described in [README.md](../README.md) (session resume from Codex, Claude Code, OpenCode, pi where applicable).

---

## 5. Long-running server + multiple clients

**Goal:** One persistent jcode server in the background; attach from several terminals or workflows.

```bash
# Terminal A: start the server
jcode serve

# Terminal B (or another machine with matching setup): attach a client
jcode connect
```

Use cases: keep one warm session, attach from VS Code integrated terminal and a separate Alacritty window, or share a machine-local server across tools. Deeper IPC and layout notes: [SERVER_ARCHITECTURE.md](SERVER_ARCHITECTURE.md).

---

## 6. Auth checks before you rely on a provider

**Goal:** Confirm tokens load and refresh without opening the TUI.

```bash
jcode auth status
jcode auth status --json
jcode auth doctor -p claude
jcode auth-test -p openai --no-smoke --no-tool-smoke
jcode auth-test --all-configured --no-smoke --no-tool-smoke --json
```

Credential locations and “trust external file” behavior: [OAUTH.md](../OAUTH.md).

---

## 7. Log in once (interactive, headless, or scriptable)

**Goal:** Configure a provider from scratch or refresh OAuth.

**Interactive (opens browser where applicable):**

```bash
jcode login --provider claude
jcode login --provider openai
jcode login --provider gemini
jcode login --provider copilot
```

**SSH / no local browser:**

```bash
jcode login --provider claude --no-browser
```

**Two-step / scripted** (URLs, callbacks, device flow patterns): see [README.md](../README.md) “OAuth and Providers” (sections on `--print-auth-url`, `--callback-url`, `--auth-code`, Copilot `--complete`, etc.).

---

## 8. Named OpenAI-compatible endpoint (vLLM, gateway, self-hosted)

**Goal:** Point jcode at a custom base URL + model and store secrets safely.

```bash
printf '%s' "$MY_API_KEY" | jcode provider add my-api \
  --base-url https://llm.example.com/v1 \
  --model my-model-id \
  --api-key-stdin \
  --set-default \
  --json

jcode --provider-profile my-api auth-test --prompt 'Reply exactly JCODE_PROVIDER_SETUP_OK'
jcode --provider-profile my-api run 'hello'
```

Local server without auth:

```bash
jcode provider add local-vllm \
  --base-url http://localhost:8000/v1 \
  --model Qwen/Qwen3-Coder-30B-A3B-Instruct \
  --no-api-key \
  --set-default
```

More flags and `config.toml` layout: [README.md](../README.md) “Config-file setup for self-hosted endpoints and MCP”.

---

## 9. MCP servers (shared tools for the agent)

**Goal:** Give the agent filesystem, DB, or internal APIs via MCP.

**Config files:**

| Scope | Path |
|--------|------|
| Global | `~/.jcode/mcp.json` |
| Project | `.jcode/mcp.json` |
| Compatibility | `.claude/mcp.json` |

Minimal example:

```json
{
  "servers": {
    "filesystem": {
      "command": "/path/to/mcp-server",
      "args": ["--root", "/workspace"],
      "env": {},
      "shared": true
    }
  }
}
```

On first run, jcode may import MCP config from Claude Code / Codex locations if `~/.jcode/mcp.json` does not exist yet ([README.md](../README.md)).

---

## 10. Browser automation (Firefox)

**Goal:** Let the agent drive a real browser (open URL, click, type, screenshot).

```bash
jcode browser status
jcode browser setup
```

After setup, in a normal agent session, the model can use the built-in **`browser`** tool. Tool actions include `open`, `snapshot`, `click`, `type`, `screenshot`, and others listed in [README.md](../README.md) “Browser Automation”. Protocol details: [BROWSER_PROVIDER_PROTOCOL.md](BROWSER_PROVIDER_PROTOCOL.md).

---

## 11. Dictation and transcripts

**Goal:** Send voice-derived text into the focused jcode session or type into the frontmost app.

```bash
jcode dictate
```

Transcript injection (for external STT pipelines) is available via the `Transcript` subcommand; see `jcode transcript --help` for modes and targeting.

---

## 12. Memory, ambient, swarm (when you outgrow a single chat)

These are **larger features**; the CLI exposes entry points, while behavior is documented in dedicated docs.

| Feature | What it is for | Where to read more |
|---------|----------------|---------------------|
| **Memory** | Semantic recall, extraction, memory tools | [MEMORY_ARCHITECTURE.md](MEMORY_ARCHITECTURE.md), `jcode memory --help` |
| **Ambient** | Background agent loops and permissions | [AMBIENT_MODE.md](AMBIENT_MODE.md), `jcode ambient --help` |
| **Swarm** | Multiple agents in one repo with coordination | [SWARM_ARCHITECTURE.md](SWARM_ARCHITECTURE.md) |

---

## 13. Self-development (editing jcode’s own source)

**Goal:** Use jcode to work on the jcode repository with reload and canary flows.

```bash
jcode self-dev
jcode self-dev --build
```

Expect heavy model usage and real edits to the tree. Use a **strong frontier model**; weaker models can break the build. Overview: [README.md](../README.md) “Customizability / Self-Dev”.

---

## 14. Replay, video export, and review

**Goal:** Re-watch a session, export a timeline, or generate a demo video.

```bash
jcode replay <session-id-or-name>
jcode replay <session> --swarm
jcode replay <session> --export
jcode replay <session> --video
```

Use `jcode replay --help` for speed, timeline override, and video geometry flags.

---

## 15. Safety and permissions

**Goal:** Understand what requires explicit approval (email, external side effects, etc.).

See [SAFETY_SYSTEM.md](SAFETY_SYSTEM.md) and CLI helpers such as `jcode permissions` for pending ambient permission flows.

---

## 16. Version, usage limits, updates

```bash
jcode version
jcode version --json
jcode usage
jcode usage --json
jcode update
```

---

## Quick reference table

| I want to… | Command or entry point |
|------------|-------------------------|
| Full interactive UI | `jcode` |
| Single non-interactive answer | `jcode run "..."` |
| Structured output | `jcode --quiet run --json "..."` |
| Stream events | `jcode --quiet run --ndjson "..."` |
| Resume | `jcode --resume <name-or-id>` |
| Background server | `jcode serve` / `jcode connect` |
| Check auth | `jcode auth status`, `jcode auth-test …` |
| Add custom API profile | `jcode provider add …` |
| Wire MCP | `~/.jcode/mcp.json` or `.jcode/mcp.json` |
| Browser tool | `jcode browser setup` |
| List models / providers | `jcode model list`, `jcode provider list` |

For Windows install paths and PATH quirks, see [WINDOWS_GETTING_STARTED.md](WINDOWS_GETTING_STARTED.md) and [WINDOWS.md](WINDOWS.md).
