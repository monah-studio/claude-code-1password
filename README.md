# claude-code-1password

Connect **1Password** to **Claude Code** (and Claude Cowork) so that Claude can read and write credentials in a dedicated **"Claude Code" vault** — 24/7, no desktop-app unlock required, no repeated approval prompts.

> ⚠️ **No secrets in this repo.** Everything here is a template. All real tokens are placeholders like `<YOUR_SERVICE_ACCOUNT_TOKEN>`.

---

## Table of Contents

1. [How it works](#how-it-works)
2. [Prerequisites](#prerequisites)
3. [Part A — Create the Service Account](#part-a--create-the-service-account)
4. [Part B — Install & verify the `op` CLI](#part-b--install--verify-the-op-cli)
5. [Part C — Configure Claude Code (global)](#part-c--configure-claude-code-global)
6. [Part D — Permission allowlist](#part-d--permission-allowlist)
7. [Part E — Verify end-to-end](#part-e--verify-end-to-end)
8. [Part F — Claude Cowork (optional)](#part-f--claude-cowork-optional)
9. [Daily usage — a skill that "just works"](#daily-usage--a-skill-that-just-works)
10. [Troubleshooting](#troubleshooting)
11. [Security notes](#security-notes)
12. [Credential index](#credential-index-template)

---

## How it works

| Layer | What it does | Why it matters |
|---|---|---|
| **1Password desktop app** | Holds your vaults, unlocked by you | The source of truth for secrets |
| **Service Account** | A machine identity with a token, scoped to ONE vault | Lets `op` read/write without the app being unlocked |
| **`op` CLI** | Command-line interface to 1Password | What Claude actually runs |
| **`OP_SERVICE_ACCOUNT_TOKEN`** env var | Global env in Claude Code settings | Injected into every session automatically |
| **`permissions.allow`** | Claude Code permission allowlist | Lets `op` run without approval prompts |

```
Claude Code session
   │  auto-injects OP_SERVICE_ACCOUNT_TOKEN
   ▼
op CLI ──Service Account token──▶ 1Password "Claude Code" vault
                                    (the ONLY vault this identity can see)
```

Because authentication goes through the **Service Account token** (not the desktop app), the CLI works even when the 1Password app is locked or closed. That is what makes it "24/7".

> ❌ **Do NOT rely on the `1password-mcp` MCP server.** It can only *write* Environments (a developer feature), cannot *read* secret values back, and requires desktop-app approval that often hangs on "Pending approval". The `op` CLI is the reliable path.

---

## Prerequisites

- macOS / Linux / Windows with a [1Password account](https://1password.com)
- [1Password desktop app](https://1password.com/downloads) installed (for creating the vault & Service Account, and optional browser autofill)
- [1Password CLI](https://developer.1password.com/docs/cli/get-started) (`op`) installed
- [Claude Code](https://code.claude.com) installed and authenticated
- [GitHub CLI](https://cli.github.com) (`gh`) — only needed for [Part F / repo upload]

---

## Part A — Create the Service Account

1. Go to <https://my.1password.com> → **Service Accounts**.
2. **Create Service Account**, name it e.g. `Claude AI Agent`.
3. Under **Vault access**, select **only** the vault you want Claude to use — e.g. `Claude Code`. (This is the security boundary: the token can *never* see your other vaults.)
4. Create it, and **copy the token** (shown once, starts with `ops_`).

Store the token safely (it is your authentication credential):

```bash
op item create --vault="Claude Code" \
  --category="API Credential" \
  --title="Service Account Auth Token: Claude AI Agent" \
  credential="<YOUR_SERVICE_ACCOUNT_TOKEN>" \
  notes="1Password Service Account token for Claude. Rotate in my.1password.com."
```

> 💡 Keeping a copy of the token *inside* the vault it grants access to is a deliberate recovery pattern: if the env config is ever wiped, you can recover it with `op item get`.

---

## Part B — Install & verify the `op` CLI

```bash
# macOS (Homebrew)
brew install --cask 1password-cli

# verify
op --version
op account list          # should list your account, e.g. <your>.1password.com
```

> Note: with a Service Account token set, `op` authenticates via the token and does **not** need the desktop app unlocked.

---

## Part C — Configure Claude Code (global)

Add the token to **global** Claude Code settings so every session in every directory inherits it.

File: `~/.claude/settings.json`

```json
{
  "env": {
    "OP_SERVICE_ACCOUNT_TOKEN": "<YOUR_SERVICE_ACCOUNT_TOKEN>"
  }
}
```

(Keep your existing `env` keys; just add this one.)

> ⚠️ This file now contains a live token. Do **not** commit it, sync it to cloud drives, or share it. It is a *local runtime* requirement.

---

## Part D — Permission allowlist

Still in `~/.claude/settings.json`, add a `permissions` block so `op` read/write commands run **without approval prompts**. All rules are scoped to the `Claude Code` vault.

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

> **Order matters for `op` commands** — put `--vault="Claude Code"` *before* the item title so the allowlist glob matches. See [Troubleshooting → allowlist](#allowlist-rules-not-matching).

> 🔒 Write commands (`create`/`edit`) are included here deliberately, scoped only to the `Claude Code` vault. Because the Service Account cannot see any other vault, this does not expose your personal vaults.

---

## Part E — Verify end-to-end

```bash
# 1. Confirm the token is picked up
op whoami
#   URL: https://<your>.1password.com
#   User Type: SERVICE_ACCOUNT

# 2. Confirm the vault is reachable
op vault list

# 3. Confirm only the intended vault is visible
op item list --vault="Claude Code"

# 4. (Negative test) confirm OTHER vaults are NOT visible
op item list --vault="Personal"        # → should error: not a vault in this account
```

Expected result: `op` works without any desktop-app unlock, and can only see the `Claude Code` vault.

---

## Part F — Claude Cowork (optional)

Claude Cowork is a separate agentic environment. Two things to know:

- **It does not automatically read `~/.claude/settings.json`.** If Cowork needs 1Password access, the token must be injected into *its* environment separately (its own env config / launch environment).
- If you don't need Cowork to read secrets, tell it plainly: *"Do not use 1Password / op; I'll provide credentials."*

> For fully automated browser *login* flows inside a headless agent, see 1Password's separate **Agentic Autofill** (Browserbase pairing) — that is a different feature from CLI vault access and not covered here.

---

## Daily usage — a skill that "just works"

Install a global Claude skill so any session triggers on "1password" / "取密码" / "save a key":

```bash
mkdir -p ~/.claude/skills/1password
```

`~/.claude/skills/1password/SKILL.md`:

```markdown
---
name: 1password
description: Read/write credentials in the 1Password "Claude Code" vault via the op CLI.
  Triggers on "1password", "取密码", "存 key", "credential", "secret".
---

# 1Password — credential access

Use `op` CLI (NOT the MCP server). Put `--vault="Claude Code"` BEFORE the item title
so the allowlist matches.

Read:   op item get --vault="Claude Code" "<TITLE>" --fields <field> --reveal
List:   op item list --vault="Claude Code"
Create: op item create --vault="Claude Code" --category="API Credential" \
          --title="<TITLE>" credential="<VALUE>" notes="<NOTE>"
Edit:   op item edit --vault="Claude Code" "<TITLE>" field=value

Syntax: use assignment (`username=x`, `password=y`, `credential=z`), NOT `--username` flags.
Vault:  "Claude Code" only. Service Account scopes access to this vault.
```

After creating the skill, **restart the session** so it loads.

---

## Troubleshooting

### `op` says "No accounts configured" / "account is not signed in"
- The `OP_SERVICE_ACCOUNT_TOKEN` is not injected into *this* session.
- Check: `env | grep OP_SERVICE_ACCOUNT_TOKEN`
- The token lives in `~/.claude/settings.json` env — new sessions load it. If missing, re-add it (or restore from your vault copy).

### Allowlist rules not matching
- Argument order matters. Use `op item get --vault="Claude Code" "<TITLE>"` (vault first), not `<TITLE> --vault=...`.
- The allowlist globs are exact-ish; if a command still prompts, standardize the argument order and add a matching rule.

### "Only the Claude Code vault, always"
- The Service Account is scoped at creation time in `my.1password.com`. It physically cannot list other vaults, even with a broad allowlist.

### Permission rules take effect immediately
- `settings.json` is watched and reloaded live (no restart needed for permission/env changes) — though a new session is safest.

---

## Security notes

- **Never commit `settings.json`** with a live token. Use env vars / secret managers in CI instead.
- **Rotate on exposure**: if a token appears in a chat log, screen recording, or untrusted repo, rotate it at <https://my.1password.com> (Service Accounts → revoke → recreate).
- **`1password-credentials.json`** is the account-recovery file. It can reset your whole account. Keep it local, never share.
- The allowlist above grants `op` write access to the `Claude Code` vault *by design* so Claude can store newly-issued keys. Scope the Service Account to that one vault to contain blast radius.
- Prefer **read-only** patterns in automation: only `create`/`edit` when the workflow genuinely needs to store something.

---

## Credential index (template)

Use this in your README or team wiki to map what lives where. Replace values — do **not** put real secrets in a repo.

| Item | Category | Field to use | Example purpose |
|---|---|---|---|
| `<device>` login | `login` | `username` / `password` | Router / NAS / server SSH login |
| `<cloud>` API token | `API Credential` | `credential` | Cloud provider API key |
| `<LLM>` key | `API Credential` | `credential` | DeepSeek / Qwen / Kimi / etc. |
| `<SSH>` private key | `API Credential` | `credential` (full PEM) | SSH into a box |
| Service Account token | `API Credential` | `credential` | The `ops_...` token above |

---

## License

MIT — this is a documentation/template repo. No 1Password code or secrets included.
