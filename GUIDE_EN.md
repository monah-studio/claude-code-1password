# claude-code-1password — English Guide

Give your AI coding agent (Claude Code) secure, **silent** access to your secrets via 1Password — no approval prompts, no desktop-app unlock, 24/7.

> ⚠️ No secrets in this repo. Replace `<YOUR_SERVICE_ACCOUNT_TOKEN>` with your own.

---

## Quickstart

### 1. Create a Service Account
Go to `https://my.1password.com` → **Service Accounts** → **Create Service Account**.
Grant access to **only** the vault the agent should use (e.g. `Claude Code`).
Copy the `ops_...` token (shown only once).

### 2. Put the token in TWO places

**A. Shell env** (survives tools that rewrite config files):
```bash
echo 'export OP_SERVICE_ACCOUNT_TOKEN="<YOUR_SERVICE_ACCOUNT_TOKEN>"' >> ~/.zshenv
echo 'export OP_VAULT="Claude Code"' >> ~/.zshenv
```

**B. Claude Code global settings** — edit `~/.claude/settings.json`:
```json
{
  "env": {
    "OP_SERVICE_ACCOUNT_TOKEN": "<YOUR_SERVICE_ACCOUNT_TOKEN>"
  }
}
```

### 3. Allow `op` without prompts
Add to `~/.claude/settings.json` `permissions.allow` (or the project-level `.claude/settings.local.json`, which survives tool rewrites):
```json
"permissions": {
  "allow": [
    "Bash(op item get --vault=\"Claude Code\" *)",
    "Bash(op item get * --vault=\"Claude Code\" *)",
    "Bash(op item list --vault=\"Claude Code\" *)",
    "Bash(op item list * --vault=\"Claude Code\" *)",
    "Bash(op vault list)",
    "Bash(op item create --vault=\"Claude Code\" *)",
    "Bash(op item edit --vault=\"Claude Code\" *)",
    "Bash(op account list)",
    "Bash(op whoami)"
  ]
}
```

### 4. Verify
```bash
op whoami                  # → User Type: SERVICE_ACCOUNT
op item list --vault="Claude Code"
```

---

## Pitfalls we hit (read this)

1. **The MCP server can't read values.** `1password-mcp` only writes Environments and never returns secrets. Use the `op` CLI.
2. **`op` uses assignment syntax.** `op item create ... username="u" password="p"` — NOT `--username`/`--password` flags.
3. **Desktop-app 12-hour limit is hard-coded.** New terminal + 10-min idle + 12-h hard cap all force Touch ID re-auth. Only a Service Account bypasses it.
4. **Service Accounts can be silently deleted** (403 Service Account Deleted). Keep a recovery copy of the token inside the vault.
5. **Other tools can wipe `settings.json`.** A model switcher (e.g. CC Switch) rewrote ours, dropping the token + permissions. Putting the token in `~/.zshenv` makes it survive.
6. **The allowlist glob is brittle.** Argument order/quoting must match exactly. Always write `op item get --vault="Claude Code" "<title>"` (vault first).
7. **SSH `IdentitiesOnly=yes` breaks the 1Password agent.** Remove it and let the agent negotiate.
8. **The SSH agent ignores custom vaults.** Write `~/.config/1Password/ssh/agent.toml` with `[[ssh-keys]]` entries (not `[[vaults]]`).
9. **Service Accounts get deleted repeatedly if you create many.** Keep only ONE active; verify `op whoami` right after creating.

---

## Pros & Cons

**Pros**
- ✅ Zero prompts once configured
- ✅ Works while the 1Password app is locked/closed
- ✅ One-vault scope = small blast radius
- ✅ SSH Agent removes local private-key files

**Cons**
- ⚠️ Scoped to ONE vault (a limitation if you need more)
- ⚠️ Token leak = full read/write to that one vault — rotate promptly
- ⚠️ Config can be clobbered by other tooling (use `~/.zshenv`)
- ⚠️ Desktop-app fallback has the 12-h Touch ID limits

---

## Agent prompt (copy-paste)

Put this in your agent's custom instructions / CLAUDE.md / system prompt:

```
You have secure access to 1Password. Use the `op` CLI — NEVER the MCP server (it cannot read values).

RULES:
1. When the user gives you a new key/password/token, ALWAYS save it to 1Password automatically:
   - Website/device login  -> op item create --vault="Claude Code" --category=login --title="<name>" username="<u>" password="<p>"
   - API key/token         -> op item create --vault="Claude Code" --category="API Credential" --title="<name>" credential="<value>"
2. When you need a secret, read it yourself (do NOT ask the user):
   - op item get --vault="Claude Code" "<title>" --fields password --reveal
   - op item list --vault="Claude Code"
3. Put --vault="Claude Code" BEFORE the item title.
4. Use assignment syntax (username="x", password="y", credential="z").
5. Only touch the "Claude Code" vault.
6. Never write real secret values into memory files or notes — only the item title.
7. If `op` errors with "No accounts configured" or "Service Account Deleted", tell the user to check OP_SERVICE_ACCOUNT_TOKEN and recreate the Service Account at my.1password.com.
```

---

## Security

- Never commit `settings.json` with a live token.
- Rotate any token that touched a chat log / screenshot / untrusted repo.
- `1password-credentials.json` (account recovery) can reset your whole account — keep it local.

---

## Official references / 官方参考

**1password.com (product / console)**
- 1Password home: <https://1password.com>
- 1Password web console (create Service Accounts, manage policies): <https://my.1password.com>
- 1Password Service Accounts docs: <https://developer.1password.com/docs/service-accounts/>

**1password.dev (developer docs)**
- CLI get started (install, `op`, integrate with desktop app): <https://www.1password.dev/cli/get-started/>
- **App integration security — the 10-minute idle / 12-hour hard limit**: <https://www.1password.dev/cli/app-integration-security>
  > "Authorization expires after 10 minutes of inactivity in the terminal session. There's a **hard limit of 12 hours**, after which you must reauthorize."
  > This is why the desktop-app (Touch ID) path keeps asking: the 12-hour cap is **not configurable**. Only a Service Account token bypasses it.
- SSH & Git (1Password SSH Agent overview): <https://www.1password.dev/ssh>
- SSH Agent setup: <https://www.1password.dev/ssh/agent/>
- SSH Agent config file (`agent.toml`, custom vaults): <https://www.1password.dev/ssh/agent/config/>
- MCP server (1Password Environments — writes only, cannot read values): <https://www.1password.dev/environments/mcp-server>
- Agentic Autofill (Browserbase pairing, separate feature): <https://www.1password.dev/agentic-autofill>

---

## License
MIT — documentation/template only. No secrets.
