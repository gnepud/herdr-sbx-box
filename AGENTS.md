# AGENTS.md

Universal rules and development invariants for adding and maintaining agent kits in `herdr-sbx-box`.

## 1. Kit Architecture
- **Base Kit**: `herdr-sbx-kit` is the sole `kind: sandbox` (entrypoint: `[herdr]`).
- **Agent Mixins**: All AI coding harnesses must be standalone `kind: mixin` kits.

## 2. sbx Engine & Credential Rules
- **OAuth Restriction**: `oauth:` is only valid on `kind: sandbox`. Never declare `oauth:` in a mixin (causes HTTP 400). Host OAuth must be declared centrally in `herdr-sbx-kit`.
- **No Duplicate Services**: A `service:` cannot be declared in both base and mixin kits. Put shared host secrets in `herdr-sbx-kit` with `required: false`.

## 3. Startup Script Contracts
- **Command Schema**: `setup.startup[].command` must be a string array (`["bash", "-c", "..."]`), never a bare string.
- **PATH Export**: The startup dispatcher runs with a minimal system PATH. Always start scripts with:
  ```bash
  export PATH="/home/agent/.local/bin:/usr/local/share/npm-global/bin:$PATH"
  ```

## 4. Standard Workflow for Adding a New Agent
When adding a new agent mixin kit:
1. **Native YOLO Mode**: Configure zero-confirmation / unrestricted mode natively through the agent's config files or flags. Pre-seed onboarding and workspace trust flags so it runs headlessly without prompts.
2. **Safe Config Merging**: If the agent shares configuration files (e.g. `settings.json`), merge updates idempotently (e.g. via `jq`) to avoid wiping Herdr hooks.
3. **Herdr Auto-Integration**: Add auto-detection in `herdr-sbx-kit`'s startup script:
   ```bash
   if command -v <cli> >/dev/null 2>&1; then
     herdr integration install <target>
   fi
   ```
4. **Validation**: Ensure `sbx kit validate <kit-dir>` passes and verify multi-agent assembly via `sbx create`.
