# herdr-sbx-box

> **Modular Docker Sandbox (`sbx`) kits powered by Herdr — running Claude Code, OpenAI Codex, Grok, Pi, and Antigravity in full YOLO mode.**

[![Docker Sandboxes](https://img.shields.io/badge/Docker-Sandboxes%20(sbx)-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/ai/sandboxes/)
[![Herdr](https://img.shields.io/badge/Workspace-Herdr%20v0.8-blueviolet)](https://herdr.dev)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

---

## Overview

**`herdr-sbx-box`** is a production-ready, modular kit suite for [Docker Sandboxes (`sbx`)](https://docs.docker.com/ai/sandboxes/).

It turns an isolated Docker Sandbox into a unified, multiplexed multi-agent terminal workspace orchestrated by [**Herdr**](https://herdr.dev). Instead of locking yourself into a single monolithic AI container, `herdr-sbx-box` decouples your tools:
- **`herdr-sbx-kit`** acts as the base Sandbox Agent Kit (providing the Herdr terminal multiplexer, shell completions, and central credential proxy).
- Individual AI harnesses (**Claude Code**, **OpenAI Codex**, **Grok**, **Pi**, and **Antigravity**) are packaged as independent **`kind: mixin`** kits that can be dynamically stacked in any combination.

All agents are pre-configured to run out of the box in **YOLO mode** (zero confirmation prompts, full capability) within the safety of the Docker sandbox boundary.

---

## Key Features

1. **Modular Mixin Architecture**:
   - Mix and match agents at launch time. Need only Claude and Grok? Stack only what you need.
   - Adding a new AI harness in the future requires only creating a new mixin kit.
2. **Zero-Clone Remote Execution**:
   - Run kits directly from GitHub via `git+https://` without cloning the repository locally.
3. **Automated Herdr Integrations**:
   - The startup hook dispatcher automatically detects installed agent CLIs (`claude`, `codex`, `grok`, `pi`, `agy`) and runs `herdr integration install <agent>` so pane titles, status, and agent events sync with Herdr in real time.
4. **True Out-of-the-box YOLO Mode**:
   - **Claude**: `defaultMode: "bypassPermissions"`, onboarding bypass, and pre-accepted project trust dialogs in `~/.claude.json`.
   - **Codex**: `approval_policy = "never"`, `sandbox_mode = "danger-full-access"`, and automatic MCP gateway registration in `~/.codex/config.toml`.
   - **Grok**: Native `[ui] permission_mode = "always-approve"` in `~/.grok/config.toml` with `GROK_LOGIN_DEVICE_FLOW=1`.
   - **Antigravity**: Pre-seeded permissive policies in `settings.json` with remote headless OAuth support.
5. **Centralized & Proxy-Managed Credentials**:
   - Host OAuth flows and API key injections are centralized in `herdr-sbx-kit` to avoid schema collisions and comply with Docker Sandbox credential isolation rules.

---

## Quick Start

### Prerequisites
- Docker Desktop with **Docker Sandboxes (`sbx`)** v0.42+ installed and enabled (`sbx version`).
- **Trust Remote Kit Sources**: By default, `sbx` only permits kits from Docker Hub (`docker.io/`). To run kits directly from GitHub (Option A), authorize the repository namespace in your host settings:
  ```bash
  sbx settings set kit.allowedSources '["docker.io/","github.com/gnepud/"]'
  ```

### Option A: Direct Remote Run (Zero Clone)

You can launch the sandbox directly from GitHub without cloning the repo:

```bash
# Launch Herdr + Claude + Codex + Grok + Pi + Antigravity
sbx run \
  --name my-herdr-box \
  --kit "git+https://github.com/gnepud/herdr-sbx-box.git#dir=claude-mixin-kit" \
  --kit "git+https://github.com/gnepud/herdr-sbx-box.git#dir=codex-mixin-kit" \
  --kit "git+https://github.com/gnepud/herdr-sbx-box.git#dir=grok-mixin-kit" \
  --kit "git+https://github.com/gnepud/herdr-sbx-box.git#dir=pi-mixin-kit" \
  --kit "git+https://github.com/gnepud/herdr-sbx-box.git#dir=agy-mixin-kit" \
  "git+https://github.com/gnepud/herdr-sbx-box.git#dir=herdr-sbx-kit" \
  .
```

### Option B: Local Clone

```bash
# 1. Clone the repository
git clone git@github.com:gnepud/herdr-sbx-box.git
cd herdr-sbx-box

# 2. Launch with local kits
sbx run \
  --name my-herdr-box \
  --kit ./claude-mixin-kit \
  --kit ./codex-mixin-kit \
  --kit ./grok-mixin-kit \
  --kit ./pi-mixin-kit \
  --kit ./agy-mixin-kit \
  ./herdr-sbx-kit \
  .
```

### Custom Combinations (Mix & Match)

Stack only the agents you want. For example, Herdr + Claude + Grok:

```bash
sbx run \
  --name claude-grok-box \
  --kit "git+https://github.com/gnepud/herdr-sbx-box.git#dir=claude-mixin-kit" \
  --kit "git+https://github.com/gnepud/herdr-sbx-box.git#dir=grok-mixin-kit" \
  "git+https://github.com/gnepud/herdr-sbx-box.git#dir=herdr-sbx-kit" \
  .
```

---

## Credential Configuration

Credentials are managed on the host via the `sbx secret` CLI and safely injected into the sandbox via proxy sentinels.

### Anthropic (Claude Code)
```bash
# Store host API key
echo "$ANTHROPIC_API_KEY" | sbx secret set anthropic

# Or interactive prompt
sbx secret set anthropic
```

### OpenAI (Codex)
```bash
# Store host API key
echo "$OPENAI_API_KEY" | sbx secret set openai

# Or start the browser OAuth flow
sbx secret set openai --oauth
```

### xAI (Grok Build)
Supports both API Key and OAuth (browser or device code):
```bash
# Option 1: Store host API key
echo "$XAI_API_KEY" | sbx secret set xai

# Option 2: Inside the sandbox terminal, authenticate via device code flow
grok login --device-auth
```

### Google (Antigravity)
When running `agy` for the first time in the sandbox, it detects the remote container environment and automatically outputs an authentication URL. Simply open the URL in your host browser and paste the callback back into the terminal.

---

## Verification & Diagnostics

Inside the sandbox or via `sbx exec`, verify agent versions and Herdr integration status:

```bash
# Check installed CLI versions
herdr --version
claude --version
codex --version
grok --version
pi --version
agy --version

# Check Herdr agent state integration
herdr integration status
```

Expected `herdr integration status` output:
```text
claude: current (v9) (/home/agent/.claude/hooks/herdr-agent-state.sh)
codex: current (v8) (/home/agent/.codex/herdr-agent-state.sh)
grok: current (v1) (/home/agent/.grok/hooks/herdr-agent-state.sh)
pi: current (v8) (/home/agent/.pi/agent/extensions/herdr-agent-state.ts)
antigravity-cli: current (v3) (/home/agent/.gemini/config/hooks/herdr-agent-state.sh)
```

---

## Acknowledgements

Special thanks to [@shelajev](https://github.com/shelajev) for:
- [agy-sbx-kit](https://github.com/shelajev/agy-sbx-kit): Reference implementation for the Antigravity sandbox kit.
- [sbx-skill](https://github.com/shelajev/sbx-skill): The agent skill that assisted in designing, building, and operating Docker Sandboxes.

---

## License

Apache License 2.0. See [LICENSE](LICENSE) for details.
