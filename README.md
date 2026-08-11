# claude-code-1password

> ⚡ **让 AI 代理静默读写 1Password —— 3 秒配置,复制即用。**
> **Give your AI agent silent 1Password access — copy, paste, done.**
> **AIエージェントに 1Password への静かなアクセスを — コピーして貼るだけ。**
> **AI 에이전트에 1Password 조용한 액세스를 — 복사해서 붙이기만 하세요.**

---

## 🚀 第一步:复制这段给 agent(3 秒搞定)

> 📋 **直接把下面的代码块整个复制**,粘到你的 agent 的自定义指令 / `CLAUDE.md` / 系统提示词。
> **Copy this whole block** into your agent's instructions / `CLAUDE.md` / system prompt.

```text
⚡ 1PASSWORD RULE (agent must follow):
You have secure access to 1Password via the `op` CLI (NOT the MCP server — it can't read values).

1. User gives you a NEW key/password/token → AUTO-SAVE it, don't ask:
   op item create --vault="Claude Code" --category=login --title="<name>" username="<u>" password="<p>
   op item create --vault="Claude Code" --category="API Credential" --title="<name>" credential="<value>"
2. You NEED a secret → READ IT yourself, don't ask:
   op item get --vault="Claude Code" "<title>" --fields password --reveal
   op item list --vault="Claude Code"
3. Put --vault="Claude Code" BEFORE the title.
4. Use assignment syntax (username="x", password="y", credential="z") — NOT --username flags.
5. ONLY touch the "Claude Code" vault.
6. Never write real secrets into memory/notes — only the item title.
7. If op fails ("No accounts configured"/"Service Account Deleted") → tell user to check
   OP_SERVICE_ACCOUNT_TOKEN and recreate the Service Account at my.1password.com.
```

> ✅ **配合左侧那个 `ops_...` token(服务账号)使用** —— 详见下方各语言指南的「配置」。
> ✅ **Works with the Service Account token** — full setup in each language guide below.

---

## 🎯 它做什么 / What it does / 効果 / 효과

| 你说话 | Agent 自动做 |
|---|---|
| "这是我的新 API key: sk-xxx" | 自动存进 1Password Claude Code 库 |
| "用 Orange Pi 的密码 SSH" | 自己读 1Password,不问你要 |
| "GitHub token 呢?" | 自己查,不用你贴 |

| You say | Agent auto-does |
|---|---|
| "Here's my new API key: sk-xxx" | Saves it to the Claude Code vault |
| "SSH with the Orange Pi password" | Reads it from 1Password, no asking |
| "Where's the GitHub token?" | Looks it up itself |

---

## 📚 完整指南 / Full guides / 完全ガイド / 전체 가이드

| 语言 | 文件 | 内容 |
|---|---|---|
| 🇺🇸 **English** | [GUIDE_EN.md](GUIDE_EN.md) | Setup · pitfalls · pros/cons · references |
| 🇨🇳 **中文** | [GUIDE_ZH.md](GUIDE_ZH.md) | 配置 · 踩坑 · 优缺点 · 参考链接 |
| 🇯🇵 **日本語** | [GUIDE_JA.md](GUIDE_JA.md) | セットアップ · 落とし穴 · 長所短所 · リファレンス |
| 🇰🇷 **한국어** | [GUIDE_KO.md](GUIDE_KO.md) | 설정 · 문제점 · 장단점 · 참조 |

每个指南都**独立完整**:架构图 + 快速开始 + 9 个真实踩坑 + 优缺点 + 该语言的 agent prompt + 官方参考链接(含 12 小时限制出处)。

---

> ⚠️ **No secrets in this repo / 本仓库不含任何密钥 / このリポジトリに秘密情報は含まれません / 이 리포지토리에는 비밀이 없습니다.**

## 📄 License & Credits / 许可证与致谢 / ライセンスとクレジット / 라이선스 및 크레딧

- **License:** [MIT](LICENSE)
- **Credits & disclaimer:** [CREDITS.md](CREDITS.md) (unofficial community guide, not affiliated with 1Password/Anthropic)
- **Author:** [Monah Studio](https://monah.ai) · <https://github.com/monah-studio>
