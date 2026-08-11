# claude-code-1password

Connect **1Password** to **Claude Code** so Claude can read/write credentials in a dedicated **"Claude Code" vault** — 24/7, no desktop-app unlock required, no repeated approval prompts.

This repo is a **field guide**: not just *how* to set it up, but the **real problems we hit** while doing it (and what we'd do differently). No secrets anywhere — all tokens are placeholders like `<YOUR_SERVICE_ACCOUNT_TOKEN>`.

> ⚠️ **No secrets in this repo.** Everything here is a template.

---

## Table of Contents

1. [Why this exists](#why-this-exists)
2. [How it works](#how-it-works)
3. [Prerequisites](#prerequisites)
4. [Setup](#setup)
   - [Part A — Create the Service Account](#part-a--create-the-service-account)
   - [Part B — Install & verify `op`](#part-b--install--verify-the-op-cli)
   - [Part C — Configure Claude Code (global)](#part-c--configure-claude-code-global)
   - [Part D — Permission allowlist](#part-d--permission-allowlist)
   - [Part E — Verify end-to-end](#part-e--verify-end-to-end)
   - [Part F — SSH Agent (bonus)](#part-f--ssh-agent-bonus)
5. [Pitfalls we hit (the real lessons)](#pitfalls-we-hit-the-real-lessons)
6. [Pros & cons](#pros--cons)
7. [Security notes](#security-notes)
8. [Credential index template](#credential-index-template)

---

## Why this exists

We wanted Claude Code to **silently read/write our secrets** without:
- manual copy-paste of API keys into each session,
- desktop-app unlock prompts every few minutes,
- "I don't have the token" whiplash across different chats.

The **`1password-mcp` MCP server** looked like the answer but turned out to be the wrong tool. The **`op` CLI + Service Account** is the reliable path. This is the journey and the final setup.

---

## How it works

| Layer | What it does | Why it matters |
|---|---|---|
| **1Password desktop app** | Holds your vaults | Source of truth for secrets |
| **Service Account** | Machine identity, token scoped to ONE vault | Lets `op` work without the app being unlocked |
| **`op` CLI** | Command-line interface to 1Password | What Claude actually runs |
| **`OP_SERVICE_ACCOUNT_TOKEN`** env var | Injected into every Claude session | Authentication for `op` |
| **`permissions.allow`** | Claude Code allowlist | No approval prompts for `op` |
| **SSH Agent** (optional) | 1Password provides SSH keys to `ssh`/`git` | No local private-key files needed |

```
Claude Code session
   │  auto-injects OP_SERVICE_ACCOUNT_TOKEN
   ▼
op CLI ──Service Account token──▶ 1Password "Claude Code" vault
                                    (the ONLY vault this identity can see)
```

Because auth goes through the **Service Account token** (not the desktop app), the CLI works even when the app is locked or closed.

---

## Prerequisites

- macOS / Linux / Windows with a [1Password account](https://1password.com)
- [1Password desktop app](https://1password.com/downloads)
- [1Password CLI](https://developer.1password.com/docs/cli/get-started) (`op`)
- [Claude Code](https://code.claude.com)
- (optional) [GitHub CLI](https://cli.github.com) for the repo part

---

## Setup

### Part A — Create the Service Account

1. <https://my.1password.com> → **Service Accounts** → **Create Service Account**
2. Name it (e.g. `Claude AI Agent`)
3. Under **Vault access**, select **only** the vault Claude should use (e.g. `Claude Code`). This is the security boundary.
4. Copy the token (shown once, starts `ops_`).

Back it up inside the vault itself:

```bash
op item create --vault="Claude Code" \
  --category="API Credential" \
  --title="Service Account Auth Token: Claude AI Agent" \
  credential="<YOUR_SERVICE_ACCOUNT_TOKEN>" \
  notes="Recovery copy — rotate in my.1password.com if exposed."
```

> 💡 Keeping a token copy *inside the vault it grants* is a deliberate recovery pattern: if the env config is wiped, `op item get` can restore it.

### Part B — Install & verify `op`

```bash
brew install --cask 1password-cli
op --version
op account list   # should show your account
```

### Part C — Configure Claude Code (global)

`~/.claude/settings.json`:

```json
{
  "env": {
    "OP_SERVICE_ACCOUNT_TOKEN": "<YOUR_SERVICE_ACCOUNT_TOKEN>"
  }
}
```

> ⚠️ This file now holds a live token. Never commit/sync/share it.

### Part D — Permission allowlist

Same file, add `permissions` so `op` runs without prompts. All scoped to `Claude Code` vault:

```json
{
  "permissions": {
    "allow": [
      "Bash(op item get --vault=\"Claude Code\" *)",
      "Bash(op item get * --vault=\"Claude Code\" *)",
      "Bash(op item get --vault=\"Claude Code\")",
      "Bash(op item get * --vault=\"Claude Code\")",
      "Bash(op item list --vault=\"Claude Code\" *)",
      "Bash(op item list * --vault=\"Claude Code\" *)",
      "Bash(op item list --vault=\"Claude Code\")",
      "Bash(op item list * --vault=\"Claude Code\")",
      "Bash(op vault list)",
      "Bash(op vault list *)",
      "Bash(op item create --vault=\"Claude Code\" *)",
      "Bash(op item create * --vault=\"Claude Code\" *)",
      "Bash(op item create --vault=\"Claude Code\")",
      "Bash(op item create * --vault=\"Claude Code\")",
      "Bash(op item edit --vault=\"Claude Code\" *)",
      "Bash(op item edit * --vault=\"Claude Code\" *)",
      "Bash(op item edit --vault=\"Claude Code\")",
      "Bash(op item edit * --vault=\"Claude Code\")",
      "Bash(op account list)",
      "Bash(op whoami)"
    ]
  }
}
```

> **Order matters:** put `--vault="Claude Code"` *before* the item title, or the glob won't match.

### Part E — Verify end-to-end

```bash
op whoami                 # User Type: SERVICE_ACCOUNT
op item list --vault="Claude Code"
op item list --vault="Personal"   # should ERROR — scope is enforced
```

### Part F — SSH Agent (bonus)

Let 1Password provide SSH keys (so local `~/.ssh` private files can be removed):

1. 1Password App → **Settings → Developer → Use the SSH Agent** (must be ON)
2. Store keys as **SSH Key** item type (Ed25519/RSA) in the vault
3. SSH agent only reads **Personal/Private/Employee** vaults by default. For a custom vault like `Claude Code`, create `~/.config/1Password/ssh/agent.toml`:

```toml
[[ssh-keys]]
item = "SSH Key - My Pi"
vault = "Claude Code"
account = "<your>.1password.com"
```

4. `SSH_AUTH_SOCK` should point at the 1Password agent socket. Verify: `ssh-add -l`

---

## Pitfalls we hit (the real lessons)

These are the things that cost us time. Read before you start.

### 1. The MCP server looks right but is useless
`1password-mcp` **cannot return secret values** — by design. It only manages *Environments* (a developer feature) and writes. We configured it, it sat at "⏸ Pending approval", and it can never read a password back. **Use `op` CLI, not the MCP server.**

### 2. `op` uses `username=`/`password=`, not `--username`
Newer `op` rejects `--username`/`--password` flags (`unknown flag`). Use assignment syntax:
```bash
op item create --category=login ... username="u" password="p"
```

### 3. The desktop-app 12-hour limit is NOT configurable
Every new terminal window + 10-min inactivity + hard 12-hour cap → reauthorize with Touch ID. There is **no setting** to raise it (verified against official docs). Only a **Service Account token** bypasses this entirely.

### 4. Service Accounts can be silently deleted
Our first SA hit `403 Service Account Deleted` out of nowhere (revoked in the web console). The token in `.zshenv` became dead instantly. **Fix:** keep the recovery copy in the vault, and know how to recreate fast.

### 5. Another tool can wipe your `settings.json`
We run **CC Switch** (a model/provider switcher) that *rewrites* `~/.claude/settings.json`, silently dropping our `OP_SERVICE_ACCOUNT_TOKEN` and `permissions`. This was the cause of repeated "suddenly needs auth again". **Fix:** also put the token in `~/.zshenv` (shell env, outside settings.json) so it survives.

### 6. The allowlist glob is brittle
`Bash(op item get * --vault="Claude Code" *)` matches only if the literal `--vault="Claude Code"` segment appears exactly. Argument order/quoting changes break it. **Fix:** standardize `op item get --vault="Claude Code" "<title>"` (vault first).

### 7. SSH `IdentitiesOnly=yes` made agent auth "fail"
The 1Password SSH agent key was fine, but forcing `IdentitiesOnly=yes` caused `Permission denied`. Removing it, the agent key worked. Don't over-constrain SSH.

### 8. The SSH agent only sees default vaults
Keys in a custom vault (`Claude Code`) are invisible to the agent until you write `~/.config/1Password/ssh/agent.toml` with `[[ssh-keys]]` entries. (`[[vaults]]` is the wrong header — we learned that too.)

### 9. "Authorization timeout" when reading a token
`op item get` sometimes times out because the desktop-app approval never completed (app locked). Unlock the app, then retry.

### 10. Permissions vs approvals — two layers
Claude Code's allowlist stops *Claude* from asking. But **1Password itself** may still prompt (Touch ID / app unlock). Service Account removes the 1Password layer too.

---

## Pros & cons

**Pros**
- ✅ Zero prompts once configured (Service Account)
- ✅ Works even if the 1Password app is locked/closed
- ✅ One vault scope = small blast radius
- ✅ SSH Agent means no local private-key files
- ✅ Same setup serves many sessions/directories

**Cons**
- ⚠️ The Service Account is scoped to **one vault** — it can't reach your other vaults (a feature, but a limitation if you need more)
- ⚠️ The allowlist globs are brittle to argument order
- ⚠️ Config can be clobbered by other tooling (CC Switch) if you only use `settings.json`
- ⚠️ A leaked token = full read/write to that one vault — **rotate promptly on any exposure**
- ⚠️ If you fall back to desktop-app auth, the **12-hour/10-min Touch ID limits are unavoidable**

---

## Security notes

- **Never commit `settings.json`** with a live token.
- **Rotate on exposure**: any token that touched a chat log / screenshot / untrusted repo → revoke & recreate at <https://my.1password.com>.
- **`1password-credentials.json`** (account recovery file) can reset your *entire* account. Keep local, never share.
- Scope the Service Account to exactly one vault to contain blast radius.
- Prefer **read-only** patterns in automation; only `create`/`edit` when you genuinely store something.

---

## Credential index (template)

| Item | Category | Field | Example |
|---|---|---|---|
| Device login | `login` | `username` / `password` | Router / NAS |
| Cloud API token | `API Credential` | `credential` | Cloud provider |
| LLM key | `API Credential` | `credential` | DeepSeek / Qwen |
| SSH private key | `SSH Key` | `private_key` | SSH into a box |
| Service Account token | `API Credential` | `credential` | The `ops_...` token |

---

## License

MIT — documentation/template only. No 1Password code or secrets.
