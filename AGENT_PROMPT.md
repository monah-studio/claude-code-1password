# Agent Prompt — 1Password for AI Agents

Copy one of the blocks below into your agent's custom instructions / CLAUDE.md / system prompt.
The agent will then read and store secrets in 1Password **silently** — no asking you.

把下面任一段粘贴到你的 agent 的自定义指令 / CLAUDE.md / 系统提示词。
之后 agent 会**静默**地在 1Password 中读取和存储密钥 —— 不用问你。

以下のいずれかのブロックをエージェントのカスタム指示 / CLAUDE.md / システムプロンプトに貼り付けます。
以後エージェントは 1Password で秘密情報を**静かに**読み書きします — あなたに尋ねません。

아래 블록 중 하나를 에이전트의 사용자 지정 지시 / CLAUDE.md / 시스템 프롬프트에 붙여넣습니다.
이후 에이전트는 1Password에서 비밀 정보를 **조용히** 읽고 씁니다 — 여러분에게 묻지 않습니다.

---

## English (EN)

```
You have secure access to 1Password. Use the `op` CLI — NEVER the MCP server (it cannot read values).

RULES:
1. When the user gives you a new key/password/token, ALWAYS save it to 1Password automatically:
   - Website/device login  -> op item create --vault="Claude Code" --category=login --title="<name>" username="<u>" password="<p>"
   - API key/token         -> op item create --vault="Claude Code" --category="API Credential" --title="<name>" credential="<value>"
2. When you need a secret, read it yourself (do NOT ask the user):
   - op item get --vault="Claude Code" "<title>" --fields password --reveal
   - op item list --vault="Claude Code"   (to see what's stored)
3. Command order matters: put --vault="Claude Code" BEFORE the item title.
4. Use assignment syntax (username="x", password="y", credential="z") — NOT --username/--password flags.
5. Only ever touch the "Claude Code" vault. Other vaults are off-limits.
6. Never write real secret values into memory files or notes — only the 1Password item title.
7. If `op` errors with "No accounts configured" or "Service Account Deleted", tell the user to check OP_SERVICE_ACCOUNT_TOKEN and recreate the Service Account in my.1password.com.
```

## 中文 (ZH-CN)

```
你拥有 1Password 的安全访问权。使用 `op` CLI —— 绝不用 MCP 服务器(它读不了值)。

规则:
1. 用户给你新的 key/密码/token 时,一律自动存进 1Password:
   - 网站/设备登录  -> op item create --vault="Claude Code" --category=login --title="<名称>" username="<用户名>" password="<密码>"
   - API key/token   -> op item create --vault="Claude Code" --category="API Credential" --title="<名称>" credential="<值>"
2. 需要密钥时自己读,不要问用户:
   - op item get --vault="Claude Code" "<标题>" --fields password --reveal
   - op item list --vault="Claude Code"  (查看已存内容)
3. 参数顺序重要:--vault="Claude Code" 必须放在条目名之前。
4. 用赋值语法(username="x", password="y", credential="z"),不要用 --username/--password 标志。
5. 只操作 "Claude Code" 保险库。其它库一律不碰。
6. 绝不把真实密钥值写进记忆文件或笔记 —— 只写 1Password 条目名。
7. 若 op 报 "No accounts configured" 或 "Service Account Deleted",提示用户检查 OP_SERVICE_ACCOUNT_TOKEN,并在 my.1password.com 重建 Service Account。
```

## 日本語 (JA)

```
あなたは 1Password への安全なアクセス権を持っています。`op` CLI を使用してください — MCP サーバーは絶対に使わない(値を読めません)。

ルール:
1. ユーザーが新しい key/パスワード/token を渡したら、必ず自動で 1Password に保存する:
   - サイト/デバイスログイン -> op item create --vault="Claude Code" --category=login --title="<名前>" username="<ユーザー名>" password="<パスワード>"
   - API key/token          -> op item create --vault="Claude Code" --category="API Credential" --title="<名前>" credential="<値>"
2. 秘密情報が必要なときは自分で読む(ユーザーに聞かない):
   - op item get --vault="Claude Code" "<タイトル>" --fields password --reveal
   - op item list --vault="Claude Code"  (保存内容を確認)
3. 引数の順序が重要:--vault="Claude Code" はタイトルの前。
4. 代入構文を使う(username="x", password="y", credential="z")。--username/--password フラグは使わない。
5. "Claude Code" ボールトのみ操作。他のボールトには触れない。
6. 実際の秘密情報の値をメモリファイルやノートに書かない — 1Password の項目名だけ。
7. op が "No accounts configured" や "Service Account Deleted" とエラーしたら、ユーザーに OP_SERVICE_ACCOUNT_TOKEN を確認してもらい、my.1password.com でサービスアカウントを再作成するよう伝える。
```

## 한국어 (KO)

```
당신은 1Password에 대한 안전한 액세스 권한을 가집니다. `op` CLI를 사용하세요 — MCP 서버는 절대 사용하지 마세요(값을 읽을 수 없음).

규칙:
1. 사용자가 새 key/비밀번호/token을 주면, 항상 자동으로 1Password에 저장하세요:
   - 웹사이트/기기 로그인 -> op item create --vault="Claude Code" --category=login --title="<이름>" username="<사용자명>" password="<비밀번호>"
   - API key/token      -> op item create --vault="Claude Code" --category="API Credential" --title="<이름>" credential="<값>"
2. 비밀 정보가 필요하면 스스로 읽으세요(사용자에게 묻지 마세요):
   - op item get --vault="Claude Code" "<제목>" --fields password --reveal
   - op item list --vault="Claude Code"  (저장 내용 확인)
3. 인자 순서가 중요합니다: --vault="Claude Code"를 제목 앞에 두세요.
4. 할당 구문을 사용하세요(username="x", password="y", credential="z"). --username/--password 플래그를 쓰지 마세요.
5. "Claude Code" 볼트만 조작하세요. 다른 볼트는 접근 금지.
6. 실제 비밀 정보 값을 메모리 파일이나 노트에 쓰지 마세요 — 1Password 항목 이름만.
7. op가 "No accounts configured" 또는 "Service Account Deleted" 오류를 내면, 사용자에게 OP_SERVICE_ACCOUNT_TOKEN을 확인하고 my.1password.com에서 서비스 계정을 다시 만들도록 안내하세요.
```

---

## Tips / 小贴士 / ヒント / 팁

- **Where to put it** / 放哪里 / どこに置く / 어디에 넣나:
  - Global: `~/.claude/CLAUDE.md` (all projects) / 全局(所有项目)
  - Project: `./CLAUDE.md` (this repo only) / 项目级
  - Claude Code skill: `~/.claude/skills/1password/SKILL.md`
- **Verify** / 验证: run `op whoami` — should show `User Type: SERVICE_ACCOUNT`.
- **Troubleshoot** / 排查: if `op` fails, check `env | grep OP_SERVICE_ACCOUNT_TOKEN`; recreate the Service Account at my.1password.com if deleted.
