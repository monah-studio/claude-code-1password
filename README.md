# claude-code-1password

**Connect 1Password to Claude Code so agents can read/write secrets silently — no prompts, no desktop unlock.**
**把 1Password 接入 Claude Code,让 AI 代理静默读写密钥 —— 无弹窗、无需桌面解锁。**
**1Password を Claude Code に接続し、AI エージェントがシームレスに秘密情報を読み書きできるようにする — プロンプトなし、デスクトップロック解除不要。**
**1Password을 Claude Code에 연결하여 AI 에이전트가 비밀 정보를 원활하게 읽고 쓸 수 있게 합니다 — 프롬프트 없음, 데스크톱 잠금 해제 불필요.**

> ⚠️ **No secrets in this repo / 本仓库不含任何密钥 / このリポジトリに秘密情報は含まれません / 이 리포지토리에는 비밀이 없습니다.** All tokens are placeholders `<YOUR_SERVICE_ACCOUNT_TOKEN>`.

---

## 📖 What is this? / 这是什么? / これは何? / 이게 뭐예요?

This is a **field guide** for giving an AI coding agent (Claude Code) secure, silent access to your secrets via 1Password. It covers the working setup plus the **real pitfalls** we hit.

这是一份**实战指南**:通过 1Password 让 AI 编码代理(Claude Code)安全、静默地访问你的密钥。包含完整配置 + 我们真实踩过的坑。

これは、AI コーディングエージェント(Claude Code)に 1Password 経由で秘密情報を安全かつシームレスにアクセスさせるための**実践ガイド**です。動作する設定と、私たちが実際に遭遇した**落とし穴**を網羅しています。

이것은 AI 코딩 에이전트(Claude Code)가 1Password를 통해 비밀 정보를 안전하고 조용히 접근하도록 하는 **실전 가이드**입니다. 작동하는 설정과 실제로 겪은 **문제점들**을 다룹니다.

---

## 🚀 Quickstart / 快速开始 / クイックスタート / 빠른 시작

### 1. Create a Service Account / 创建服务账号 / サービスアカウント作成 / 서비스 계정 생성

Go to `https://my.1password.com` → **Service Accounts** → **Create Service Account**. Grant access to **only** the vault you want the agent to use (e.g. `Claude Code`).

前往 `https://my.1password.com` → **服务账号** → **创建服务账号**。仅授予 agent 需要使用的**那一个**保险库(如 `Claude Code`)。

`https://my.1password.com` → **サービスアカウント** → **サービスアカウント作成**。エージェントに使わせたい**その1つの**ボールトのみにアクセスを許可します(例 `Claude Code`)。

`https://my.1password.com` → **서비스 계정** → **서비스 계정 만들기**. 에이전트가 사용할 **하나의** 볼트에만 액세스를 부여합니다(예: `Claude Code`).

Copy the `ops_...` token (shown once).

复制 `ops_...` token(只显示一次)。

`ops_...` トークンをコピーします(一度だけ表示)。

`ops_...` 토큰을 복사합니다(한 번만 표시됨).

### 2. Put the token in TWO places / 把 token 放到两个地方 / トークンを2箇所に / 토큰을 두 곳에

```bash
# A. Shell env (survives tool rewrites) / Shell 环境变量(不怕工具覆盖)
echo 'export OP_SERVICE_ACCOUNT_TOKEN="<YOUR_SERVICE_ACCOUNT_TOKEN>"' >> ~/.zshenv
echo 'export OP_VAULT="Claude Code"' >> ~/.zshenv

# B. Claude Code global settings / 全局设置
# Edit ~/.claude/settings.json, add to "env":
```

```json
{
  "env": {
    "OP_SERVICE_ACCOUNT_TOKEN": "<YOUR_SERVICE_ACCOUNT_TOKEN>"
  }
}
```

### 3. Allow `op` without prompts / 让 op 免批准 / op をプロンプトなしに / op 프롬프트 없이

Add to `~/.claude/settings.json` `permissions.allow` (or the project-level `.claude/settings.local.json` to survive tool rewrites):

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

### 4. Verify / 验证 / 確認 / 확인

```bash
op whoami                  # → User Type: SERVICE_ACCOUNT
op item list --vault="Claude Code"
```

---

## 🕳️ Pitfalls we hit (READ THIS) / 我们踩过的坑(必读) / 遭遇した落とし穴(必読) / 겪은 문제점(꼭 읽으세요)

| # | Pitfall / 坑 / 落とし穴 / 문제점 | Lesson / 教训 / 教訓 / 교훈 |
|---|---|---|
| 1 | **The MCP server can't read values** / MCP 服务器只能写不能读 / MCPサーバーは値を読み取れない / MCP 서버는 값을 읽을 수 없음 | Use `op` CLI, not `1password-mcp` / 用 op CLI,别用 MCP / op CLIを使う / op CLI 사용 |
| 2 | **`op` uses `username=` not `--username`** / op 用赋值语法 / opは代入構文 / op는 할당 구문 | `op item create ... username="u" password="p"` |
| 3 | **Desktop 12-hour limit is hard-coded** / 桌面 12 小时上限不可改 / デスクトップ12時間制限は変更不可 / 데스크톱 12시간 제한 변경 불가 | Only a Service Account bypasses it / 只有 SA 能绕过 / SAのみ回避可能 / SA만 우회 가능 |
| 4 | **Service Accounts can be silently deleted** / SA 会被静默删除 / SAは静かに削除される / SA가 조용히 삭제됨 | Keep a recovery copy in the vault / 在保险库留备份 / ボールトに復旧用コピー / 볼트에 복구용 사본 |
| 5 | **Another tool can wipe settings.json** / 其它工具会清空 settings.json / 別ツールがsettings.jsonを消す / 다른 도구가 settings.json을 지움 | Put token in `~/.zshenv` too / token 也放 .zshenv / トークンも.zshenvへ / 토큰도 .zshenv에 |
| 6 | **Allowlist glob is brittle** / 权限规则脆弱 / 許可リストのglobは脆い / 허용 목록 glob 취약 | Always `--vault="Claude Code"` before the title / 参数顺序固定 / 引数順を固定 / 인자 순서 고정 |
| 7 | **SSH `IdentitiesOnly=yes` breaks agent** / SSH 强约束反而失败 / SSHのIdentitiesOnlyが失敗を招く / SSH IdentitiesOnly로 실패 | Remove it; let the agent negotiate / 去掉它 / 外す / 제거 |
| 8 | **SSH agent ignores custom vaults** / SSH agent 看不到自定义保险库 / SSHエージェントはカスタムボールトを見ない / SSH 에이전트는 커스텀 볼트를 안 봄 | Write `~/.config/1Password/ssh/agent.toml` with `[[ssh-keys]]` / 写 agent.toml |
| 9 | **Service Accounts get deleted repeatedly** / SA 反复被删 / SAが繰り返し削除される / SA가 반복 삭제됨 | Keep only ONE SA; verify `op whoami` immediately / 只留一个 SA,立即验证 / SAを1つだけに / SA 하나만 유지 |

---

## ⚖️ Pros & Cons / 优缺点 / 長所と短所 / 장단점

**Pros / 优点 / 長所 / 장점**
- ✅ Zero prompts once configured / 配置后零弹窗 / 設定後プロンプトなし / 설정 후 프롬프트 없음
- ✅ Works with app locked / App 锁定也能用 / アプリロック中でも動作 / 앱 잠겨 있어도 동작
- ✅ One-vault scope = small blast radius / 单库作用域风险小 / 単一ボールトでリスク小 / 단일 볼트로 리스크 최소
- ✅ SSH Agent removes local private-key files / SSH Agent 免本地私钥文件 / ローカル鍵ファイル不要 / 로컬 키 파일 불필요

**Cons / 缺点 / 短所 / 단점**
- ⚠️ Scoped to ONE vault / 只限单库 / 1ボールト限定 / 하나의 볼트로 제한
- ⚠️ Token leak = full access to that vault / token 泄露=单库全权限 / トークン漏洩=そのボールト全アクセス / 토큰 유출 = 볼트 전체 액세스
- ⚠️ Settings can be clobbered by other tools / 配置可能被覆盖 / 設定が他ツールに上書きされる / 설정이 다른 도구로 덮어씀
- ⚠️ Desktop-app fallback has 12-h Touch ID limits / 桌面模式有 12 小时限制 / デスクトップ方式は12時間制限 / 데스크톱 방식은 12시간 제한

---

## 🤖 Give this prompt to YOUR agent / 把这段提示词给你的 agent / エージェントにこのプロンプトを / 에이전트에게 이 프롬프트를

See **[AGENT_PROMPT.md](AGENT_PROMPT.md)** — copy-paste ready, four languages, tells the agent exactly how to store/retrieve secrets without asking you.

见 **[AGENT_PROMPT.md](AGENT_PROMPT.md)** —— 可直接复制,四语,告诉 agent 如何静默存取密钥、不问你。

**[AGENT_PROMPT.md](AGENT_PROMPT.md)** 参照 — コピペ可能、4言語対応、エージェントに秘密情報の扱い方を指示。

**[AGENT_PROMPT.md](AGENT_PROMPT.md)** 참조 — 복사하여 사용 가능, 4개 언어, 에이전트가 묻지 않고 비밀 정보를 다루도록 지시.

---

## 🔒 Security / 安全 / セキュリティ / 보안

- Never commit `settings.json` with a live token / 绝不提交含 token 的 settings.json / 生のトークンを含むsettings.jsonをコミットしない / 라이브 토큰이 있는 settings.json 커밋 금지
- Rotate any token that touched a chat log / 泄露过的 token 一律轮换 / チャットに晒したトークンは必ずローテーション / 채팅에 노출된 토큰은 반드시 교체
- `1password-credentials.json` can reset your whole account — keep local / 账号恢复文件绝不外传 / アカウント復旧ファイルはローカルのみ / 계정 복구 파일은 로컬에서만

---

## 📄 License

MIT — documentation/template only. No secrets.
