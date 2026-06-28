# upmee-open-agent

通过自然语言直接调用 upmee API，零代码、零基础即可使用。

本仓库内置了两条 Claude Code 技能命令：

- `/upmee_setup`：一次性初始化，保存 API Key 并下载接口文档
- `/upmee`：用自然语言描述你的需求，AI 自动完成接口调用

---

## 你需要准备什么

| 所需项目 | 获取方式 |
|---|---|
| [Claude Code](https://claude.ai/code) | 安装并登录 |
| upmee API Key | 登录 upmee 平台 → 开发者设置 → API Key 管理 → 创建 |
| team_id | 登录 upmee 平台 → 团队设置中查看 |

---

## 快速开始

### 第一步：克隆仓库

```bash
git clone https://github.com/dreamerainc/upmee-open-agent.git
cd upmee-open-agent
```

> 后续每次使用都需要在这个目录下启动 Claude Code，技能命令才会生效。

### 第二步：启动 Claude Code

```bash
claude
```

### 第三步：初始化（只需一次）

在 Claude Code 对话中运行：

```
/upmee_setup 你的API-Key
```

如果已知团队 ID，可以一并传入：

```
/upmee_setup 你的API-Key 12345
```

看到 `✅ upmee 配置完成！` 即表示初始化成功，API Key 永久保存在本机，无需重复配置。

### 第四步：开始使用

```
/upmee 帮我查询所有绑定的账号，team_id 是 12345
```

---

## 使用示例

### 查询账号

```
/upmee 查询 team_id 为 12345 的所有绑定账号
```

### 发布视频

```
/upmee 把 /Users/me/Desktop/video.mp4 发布到 TikTok，标题是"夏日风景"，team_id 是 12345
```

### 查看发布任务状态

```
/upmee 查看最近7天的发布任务，team_id 是 12345
```

### 查询并回复评论

```
/upmee 查询 item_id 为 7xxx 的视频评论，team_id 是 12345
/upmee 回复 item_id 7xxx、comment_id xxx 的评论，内容是"感谢支持！"，team_id 是 12345
```

### 查询私信

```
/upmee 查询 media_id 为 888 的账号最新会话列表，team_id 是 12345
```

---

## 日常使用

每次打开终端后，进入本仓库目录再启动 Claude Code：

```bash
cd upmee-open-agent
claude
```

然后直接输入需求即可，无需重新初始化。

---

## 常见问题

**Q：`/upmee` 命令不被识别？**

必须在本仓库目录下启动 Claude Code 才能识别技能命令。确认方法：

```bash
# macOS / Linux
ls .claude/skills/upmee/SKILL.md

# Windows PowerShell
Test-Path ".\.claude\skills\upmee\SKILL.md"
```

若文件存在但命令仍不识别，退出 Claude Code 后重新在该目录运行 `claude`。

---

**Q：`/upmee_setup` 提示 API Key 无效？**

1. 确认复制的 Key 完整，没有多余空格
2. 登录 upmee 平台确认该 Key 处于启用状态
3. 重新运行 `/upmee_setup 你的新API-Key`

---

**Q：每次打开都要重新运行 `/upmee_setup` 吗？**

不需要。初始化只需运行一次，配置永久保存在本机（`~/.upmee/config.json`）。

---

**Q：发布视频时 AI 要求手动运行命令，正常吗？**

正常。视频上传涉及二进制文件传输，AI 会生成完整命令，你复制到终端运行后，把输出结果粘贴回对话，AI 继续完成后续步骤。整个过程只需这一次手动操作。

---

**Q：upmee 接口文档更新了怎么办？**

重新运行 `/upmee_setup 你的API-Key` 即可刷新本地文档，无需重新克隆仓库。

---

## 在 Cursor / Windsurf 中使用

Cursor 和 Windsurf 不支持 `/upmee` 斜杠命令，需通过 Rules 机制引入技能文档。

**Cursor**：`Cursor Settings → Rules → User Rules`

**Windsurf**：项目根目录创建 `.windsurfrules` 文件，或通过 Cascade Rules 设置

在 Rules 中添加以下内容：

```
你是一个 upmee API 助手。
当用户说"初始化 upmee"时，读取当前目录下 .claude/skills/upmee_setup/SKILL.md 并按步骤执行。
当用户提出 upmee 相关操作需求时，读取当前目录下 .claude/skills/upmee/SKILL.md 并按步骤执行。
所有 HTTP 请求通过 IDE 内置终端的 curl 命令执行。
```

然后在 AI 对话框中输入自然语言即可，无需 `/upmee` 前缀：

```
初始化 upmee，我的 API Key 是 sk-xxxxxxxx，team_id 是 12345
帮我查询所有绑定的账号
```

---

## 目录结构

```
upmee-open-agent/
└── .claude/
    └── skills/
        ├── upmee_setup/
        │   └── SKILL.md   # 初始化技能
        └── upmee/
            └── SKILL.md   # 任务执行技能
```

---

如有使用问题，请联系 upmee 官方支持。
