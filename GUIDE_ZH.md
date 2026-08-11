# claude-code-1password — 中文指南

让 AI 编码代理(Claude Code)通过 1Password **安全、静默**地访问你的密钥 —— 无批准弹窗、无需桌面解锁、24/7 可用。

> ⚠️ 本仓库不含任何密钥。把 `<YOUR_SERVICE_ACCOUNT_TOKEN>` 换成你自己的。

---

## 快速开始

### 1. 创建服务账号
前往 `https://my.1password.com` → **服务账号** → **创建服务账号**。
仅授予 agent 需要使用的**那一个**保险库(如 `Claude Code`)。
复制 `ops_...` token(只显示一次)。

### 2. 把 token 放到两个地方

**A. Shell 环境变量**(不怕工具覆盖配置):
```bash
echo 'export OP_SERVICE_ACCOUNT_TOKEN="<YOUR_SERVICE_ACCOUNT_TOKEN>"' >> ~/.zshenv
echo 'export OP_VAULT="Claude Code"' >> ~/.zshenv
```

**B. Claude Code 全局设置** —— 编辑 `~/.claude/settings.json`:
```json
{
  "env": {
    "OP_SERVICE_ACCOUNT_TOKEN": "<YOUR_SERVICE_ACCOUNT_TOKEN>"
  }
}
```

### 3. 让 `op` 免批准
把以下加到 `~/.claude/settings.json` 的 `permissions.allow`(或项目级 `.claude/settings.local.json`,它不怕工具覆盖):
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

### 4. 验证
```bash
op whoami                  # → User Type: SERVICE_ACCOUNT
op item list --vault="Claude Code"
```

---

## 我们踩过的坑(必读)

1. **MCP 服务器读不了值。** `1password-mcp` 只能写 Environments,永远不返回密钥。用 `op` CLI。
2. **`op` 用赋值语法。** `op item create ... username="u" password="p"` —— 不是 `--username`/`--password` 标志。
3. **桌面 12 小时上限不可改。** 新终端 + 空闲 10 分钟 + 12 小时硬上限都会强制 Touch ID 重授权。只有 Service Account 能绕过。
4. **服务账号会被静默删除**(403 Service Account Deleted)。在保险库里留一份 token 备份。
5. **其它工具会清空 `settings.json`。** 一个模型切换器(如 CC Switch)重写了我们的配置,丢了 token 和权限。把 token 放 `~/.zshenv` 能保住。
6. **权限规则 glob 脆弱。** 参数顺序/引号必须完全一致。始终写 `op item get --vault="Claude Code" "<标题>"`(vault 在前)。
7. **SSH `IdentitiesOnly=yes` 会破坏 1Password agent。** 去掉它,让 agent 自己协商。
8. **SSH agent 看不到自定义保险库。** 写 `~/.config/1Password/ssh/agent.toml`,用 `[[ssh-keys]]` 条目(不是 `[[vaults]]`)。
9. **建太多服务账号会被反复删。** 只保留一个活跃的;创建后立刻 `op whoami` 验证。

---

## 优缺点

**优点**
- ✅ 配置后零弹窗
- ✅ App 锁定/关闭时也能用
- ✅ 单库作用域,风险小
- ✅ SSH Agent 免本地私钥文件

**缺点**
- ⚠️ 只限单库(需要更多库时是限制)
- ⚠️ token 泄露 = 该库全读写权限 —— 立即轮换
- ⚠️ 配置可能被其它工具覆盖(用 `~/.zshenv`)
- ⚠️ 桌面模式有 12 小时 Touch ID 限制

---

## Agent 提示词(一键复制)

把下面这段放进你的 agent 的自定义指令 / CLAUDE.md / 系统提示词:

```
你拥有 1Password 的安全访问权。使用 `op` CLI —— 绝不用 MCP 服务器(它读不了值)。

规则:
1. 用户给你新的 key/密码/token 时,一律自动存进 1Password:
   - 网站/设备登录  -> op item create --vault="Claude Code" --category=login --title="<名称>" username="<用户名>" password="<密码>"
   - API key/token   -> op item create --vault="Claude Code" --category="API Credential" --title="<名称>" credential="<值>"
2. 需要密钥时自己读,不要问用户:
   - op item get --vault="Claude Code" "<标题>" --fields password --reveal
   - op item list --vault="Claude Code"
3. --vault="Claude Code" 必须放在条目名之前。
4. 用赋值语法(username="x", password="y", credential="z")。
5. 只操作 "Claude Code" 保险库。
6. 绝不把真实密钥值写进记忆文件或笔记 —— 只写条目名。
7. 若 op 报 "No accounts configured" 或 "Service Account Deleted",提示用户检查 OP_SERVICE_ACCOUNT_TOKEN,并在 my.1password.com 重建服务账号。
```

---

## 安全须知

- 绝不提交含真实 token 的 `settings.json`。
- 任何出现在聊天/截图/不可信仓库里的 token,一律轮换。
- `1password-credentials.json`(账号恢复文件)能重置整个账号 —— 只留在本地。

---

## 官方参考链接

**1password.com(产品 / 控制台)**
- 1Password 官网: <https://1password.com>
- 1Password 网页控制台(创建服务账号、管理策略): <https://my.1password.com>
- 1Password 服务账号文档: <https://developer.1password.com/docs/service-accounts/>

**1password.dev(开发者文档)**
- CLI 快速开始(安装、`op`、桌面集成): <https://www.1password.dev/cli/get-started/>
- **桌面集成安全 —— 10 分钟空闲 / 12 小时硬上限的官方出处**: <https://www.1password.dev/cli/app-integration-security>
  > "Authorization expires after 10 minutes of inactivity in the terminal session. There's a **hard limit of 12 hours**, after which you must reauthorize."
  > (授权在终端空闲 10 分钟后过期;有 **12 小时硬上限**,之后必须重新授权。)
  > 这就是为什么桌面(Touch ID)模式老是要你批准:12 小时上限**不可配置**。只有 Service Account token 能绕过。
- SSH & Git(1Password SSH Agent 概览): <https://www.1password.dev/ssh>
- SSH Agent 设置: <https://www.1password.dev/ssh/agent/>
- SSH Agent 配置文件(`agent.toml`,自定义保险库): <https://www.1password.dev/ssh/agent/config/>
- MCP 服务器(1Password Environments —— 只能写、不能读值): <https://www.1password.dev/environments/mcp-server>
- Agentic Autofill(Browserbase 配对,独立功能): <https://www.1password.dev/agentic-autofill>

---

## 许可证
MIT —— 仅文档/模板,不含密钥。
