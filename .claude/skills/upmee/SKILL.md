---
name: upmee
description: upmee OpenAPI 智能任务执行：用自然语言描述需求，自动调用对应接口完成账号查询、内容发布（含视频上传）、评论管理、私信、数据统计等任务
---

# upmee 任务执行

## 调用方式

```
/upmee <用自然语言描述你想做的事>
```

示例：
- `/upmee 帮我查询所有绑定的 TikTok 账号`
- `/upmee 把 /Users/me/video.mp4 发布到 TikTok，标题是"夏日风景"`
- `/upmee 查询最近7天的发布任务状态`
- `/upmee 查询 item_id 为 xxx 的评论列表`

## 执行原则

> 用户描述需求即表示已授权执行对应操作。**直接完成所有步骤（读配置、加载文档、发起 HTTP 请求），不要在中途询问"是否继续"或"确认执行"。仅在缺少必填参数或发生错误时停下来告知用户。**

---

## 第一步：加载配置

读取本地配置文件获取 API Key 和默认设置：

**macOS / Linux**：
```bash
cat ~/.upmee/config.json
```

**Windows（PowerShell）**：
```powershell
Get-Content "$env:USERPROFILE\.upmee\config.json" | ConvertFrom-Json
```

**Windows（cmd）**：
```cmd
type %USERPROFILE%\.upmee\config.json
```

若配置文件不存在，告知用户先执行 `/upmee_setup <api_key>` 完成初始化。

从配置中提取：
- `api_key`：后续所有请求的认证凭据
- `base_url`：默认 `https://www.upmee.cc`
- `team_id`：默认团队 ID（若用户在描述中指定了 team_id，优先使用用户指定的）
- `os`：操作系统类型，决定 HTTP 工具选择

---

## 第二步：理解用户意图，匹配接口

根据用户的自然语言描述，判断属于以下哪个操作类型：

| 用户描述关键词 | 对应操作 | 需要 team_id |
|---|---|---|
| 查询账号、账号列表、绑定的账号 | 账号管理 → `media.md` | ✅ |
| 删除账号、解绑账号 | 账号管理 → `media.md` | ✅ |
| 获取授权链接、授权、绑定新账号 | 账号管理 → `media.md` | ✅ |
| 热门音乐、背景音乐 | 账号管理 → `media.md` | ✅ |
| 发布视频、发布图文、发帖、创建任务 | 发布任务 → `task.md`（含上传） | ✅ |
| 查看任务、任务状态、发布记录 | 发布任务 → `task.md` | ✅ |
| B站分区、视频分区 | 发布任务 → `task.md` | ✅ |
| Pinterest画板 | 发布任务 → `task.md` | ✅ |
| 评论列表、查看评论 | 评论管理 → `comment.md` | ✅ |
| 回复评论 | 评论管理 → `comment.md` | ✅ |
| 删除评论 | 评论管理 → `comment.md` | ✅ |
| 统计数据、数据分析、播放量、点赞数 | 数据统计 → `stat.md` | ✅ |
| 私信、会话列表、消息 | 私信管理 → `message.md` | ✅ |
| 发送私信、回复私信 | 私信管理 → `message.md` | ✅ |
| 橱窗、TikTok Shop、商品 | 橱窗任务 → `shop.md` | ✅ |

若意图不明确，询问用户：
> "请问您想要执行哪类操作？例如：查询账号、发布内容、查看评论、查看私信等"

---

## 第三步：加载对应域文档

从配置中读取 `docs_path`，拼接域文件名，读取对应文档。

文档路径（按优先级依次尝试）：

1. **配置文件中的路径**（setup 下载的文档）：
   - macOS/Linux：`~/.upmee/docs/<domain>.md`
   - Windows：`%USERPROFILE%\.upmee\docs\<domain>.md`

2. **本地项目目录**（开发者环境）：`docs/open_api_guide/<domain>.md`

3. **以上均不存在**：告知用户先运行 `/upmee_setup` 下载文档。

读取文档的命令：

**macOS / Linux**：
```bash
cat ~/.upmee/docs/<domain>.md
```

**Windows（PowerShell）**：
```powershell
Get-Content "$env:USERPROFILE\.upmee\docs\<domain>.md"
```

域名与文件名对应：`media` → `media.md`，`task` → `task.md`，以此类推。

---

## 第四步：收集必要参数

根据域文档中的接口参数要求，检查用户描述中是否已提供所需参数：

**已在用户描述中 → 直接使用**

**缺少必填参数 → 主动询问用户**，示例：
- 缺少 team_id：`请问您的 team_id 是多少？`
- 缺少 media_ids：`请问要发布到哪些账号？我可以先帮您查询已绑定的账号列表`
- 缺少文件路径：`请提供视频文件的完整路径，例如：/Users/me/video.mp4`

**有默认值的可选参数 → 使用默认值，不打扰用户**

---

## 第五步：执行 HTTP 请求

### 普通 JSON 请求（绝大多数接口）

**macOS / Linux**：
```bash
curl -s -X POST "<base_url><path>" \
  -H "API-Key: <api_key>" \
  -H "Content-Type: application/json" \
  -d '<JSON请求体>'
```

**Windows（curl.exe）**：
```cmd
curl.exe -s -X POST "<base_url><path>" -H "API-Key: <api_key>" -H "Content-Type: application/json" -d "<JSON请求体（双引号需转义）>"
```

**Windows（PowerShell，当 curl 不可用时）**：
```powershell
$body = '<JSON请求体>'
$resp = Invoke-RestMethod -Method POST -Uri "<base_url><path>" `
  -Headers @{"API-Key"="<api_key>"} `
  -Body $body -ContentType "application/json"
$resp | ConvertTo-Json -Depth 10
```

解析响应 JSON：
- `code=0`：成功，提取 `data` 字段展示给用户
- `code!=0`：失败，向用户展示 `msg` 字段的错误原因

---

## 第六步（上传发布流程）：文件上传

**仅当用户需要发布视频或图片时执行此步骤**

### 6.1 获取文件信息

**macOS / Linux**：
```bash
# 获取文件大小（字节）
stat -c%s "<文件路径>"           # Linux
stat -f%z "<文件路径>"           # macOS

# 获取文件大小（macOS/Linux 通用）
wc -c < "<文件路径>"
```

**Windows（PowerShell）**：
```powershell
(Get-Item "<文件路径>").Length
```

### 6.2 计算分片数量

建议每片 10MB（10485760 字节）。

```
分片数 = ceil(文件大小 / 10485760)
若文件大小 ≤ 50MB，可使用单文件直传
若文件大小 > 50MB，必须使用分片上传
```

### 6.3a 分片上传流程（文件 > 50MB 或推荐方式）

**步骤一：初始化（JSON 接口，AI 可直接调用）**

```bash
curl -s -X POST "<base_url>/open_api/tool/s3/upload/v2/multipart/init" \
  -H "API-Key: <api_key>" \
  -H "Content-Type: application/json" \
  -d '{"total_parts": <分片数>}'
```

保存返回的 `upload_token`。

**步骤二：逐片上传（multipart/form-data，需用户配合运行）**

> ⚠️ 分片上传涉及二进制文件分割，需要在用户本机终端运行命令。向用户展示以下命令，请用户逐个运行。

**macOS / Linux（每片）**：
```bash
# 上传第 N 片（N 从 1 开始）
dd if="<文件路径>" bs=10485760 skip=$((N-1)) count=1 2>/dev/null | \
curl -s -X POST "<base_url>/open_api/tool/s3/upload/v2/multipart" \
  -H "API-Key: <api_key>" \
  -F "upload_token=<upload_token>" \
  -F "part_number=<N>" \
  -F "file=@-;type=application/octet-stream"
```

**Windows（PowerShell，每片）**：
```powershell
# 上传第 N 片（N 从 1 开始）
$chunkSize = 10485760
$offset = ($N - 1) * $chunkSize
$bytes = [System.IO.File]::ReadAllBytes("<文件路径>")
$chunk = $bytes[$offset..([Math]::Min($offset + $chunkSize - 1, $bytes.Length - 1))]
$tmpFile = "$env:TEMP\upmee_chunk_$N.bin"
[System.IO.File]::WriteAllBytes($tmpFile, $chunk)
curl.exe -s -X POST "<base_url>/open_api/tool/s3/upload/v2/multipart" `
  -H "API-Key: <api_key>" `
  -F "upload_token=<upload_token>" `
  -F "part_number=<N>" `
  -F "file=@$tmpFile"
Remove-Item $tmpFile
```

若分片数量较多（>5片），可生成一个完整的脚本文件，让用户一次运行完成所有分片。

**步骤三：完成合并（JSON 接口，AI 可直接调用）**

```bash
curl -s -X POST "<base_url>/open_api/tool/s3/upload/v2/multipart/complete" \
  -H "API-Key: <api_key>" \
  -H "Content-Type: application/json" \
  -d '{"upload_token": "<upload_token>"}'
```

保存返回的 `object_id`。

### 6.3b 单文件直传（文件 ≤ 50MB）

> ⚠️ 也是二进制上传，向用户展示命令请其运行。

**macOS / Linux**：
```bash
curl -s -X POST "<base_url>/open_api/tool/s3/upload/v2/single" \
  -H "API-Key: <api_key>" \
  -F "type=<video|cover|image>" \
  -F "file=@<文件路径>"
```

**Windows（curl.exe）**：
```cmd
curl.exe -s -X POST "<base_url>/open_api/tool/s3/upload/v2/single" -H "API-Key: <api_key>" -F "type=<video|cover|image>" -F "file=@<文件路径>"
```

保存返回的 `object_id`。

### 6.4 用 object_id 创建发布任务

上传完成后，用获得的 `object_id` 作为 `video_id`（或 `picture_ids`）调用发布接口：

```bash
curl -s -X POST "<base_url>/open_api/media/task/create/v2" \
  -H "API-Key: <api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "team_id": <team_id>,
    "media_ids": [<media_id1>, <media_id2>],
    "video_id": "<object_id>",
    "pub_now": true,
    "title": "<标题>",
    "common_data": {"desc": "<描述>"}
  }'
```

---

## 第七步：展示结果

将 API 返回结果转化为用户友好的中文描述：

**成功示例**：
```
✅ 操作成功！

查询到 5 个已绑定账号：
  1. TikTok - @username1（粉丝 12.3万）
  2. Instagram - @username2（粉丝 8.6万）
  3. YouTube - username3（粉丝 3.2万）
  ...

如需查看某账号详情或执行其他操作，请继续描述。
```

**发布任务成功**：
```
✅ 发布任务创建成功！

任务 ID：123456
发布到：TikTok @username1
发布方式：立即发布
标题："夏日风景"

您可以说"查询任务 123456 的状态"来跟踪发布进度。
```

**失败示例**：
```
❌ 操作失败

原因：team_id 不存在或无权限访问
建议：请确认 team_id 是否正确，或联系 upmee 客服。
```

**分页数据**：
- 若结果有多页，展示当前页数据并询问："结果共 X 页，当前第 1 页，是否继续查看下一页？"

---

## 特殊处理规则

**team_id 不在配置中时**：
1. 优先从用户描述中提取（如"用团队123"、"team_id 456"）
2. 若用户未提供，提示："请提供 team_id，您可以登录 upmee 平台在团队设置中找到"
3. 任务成功后询问是否保存为默认 team_id

**多账号发布**：
- 若用户未指定账号，先调用账号列表接口，展示账号供用户选择
- 确认账号后再执行发布

**二进制上传提示**：
- 分片上传和单文件直传需要用户在本机运行命令
- AI 负责生成完整命令，用户只需复制粘贴到终端运行，再把输出结果粘贴回来
- 若 AI 有 Bash 工具且文件路径可访问，可直接执行，无需用户手动操作

**Windows 路径兼容**：
- Windows 路径含反斜杠（`\`），在 PowerShell 中通常可直接使用
- 在 cmd 中传给 curl 时，路径中的空格需用双引号包裹
- 建议用户使用正斜杠（`/`）或将路径用引号括起来