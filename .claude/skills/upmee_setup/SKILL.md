---
name: upmee_setup
description: upmee OpenAPI 一次性初始化：保存 API Key 到本地配置文件，验证连通性，完成后即可使用 /upmee 执行任何接口任务
---

# upmee 初始化配置

## 调用方式

```
/upmee_setup <你的API-Key> [team_id]
```

- `API-Key`：必填，在 upmee 平台「开发者设置」中获取
- `team_id`：可选，默认团队 ID，可后续修改

---

## 执行原则

> 用户运行 `/upmee_setup` 即表示已授权完成全部初始化操作。**所有步骤直接执行，不要在中途询问"是否继续"或"是否保存"，不要逐步确认，出现错误时再停下来告知用户。**

## 执行步骤

### 第一步：检测操作系统和工具

运行以下命令检测当前环境：

```bash
uname -s 2>/dev/null || echo "Windows"
```

- 输出包含 `Darwin` → macOS
- 输出包含 `Linux` → Linux  
- 命令失败或输出 `Windows` → Windows

同时检测 curl 是否可用：

```bash
curl --version 2>&1 | head -1
```

- **macOS / Linux**：直接使用系统 `curl`
- **Windows**：Windows 10+ 内置 `curl.exe`，在 PowerShell 和 cmd 中均可使用。若不可用，使用 PowerShell 的 `Invoke-RestMethod`

记录检测结果，供后续步骤使用。

---

### 第二步：创建配置目录和文件

**macOS / Linux**：
```bash
mkdir -p ~/.upmee
```

**Windows（PowerShell）**：
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.upmee"
```

将以下内容写入配置文件：

- 配置文件路径（macOS/Linux）：`~/.upmee/config.json`
- 配置文件路径（Windows）：`%USERPROFILE%\.upmee\config.json`

```json
{
  "api_key": "<用户提供的API-Key>",
  "team_id": <用户提供的team_id或null>,
  "base_url": "https://www.upmee.cc",
  "os": "<检测到的操作系统>",
  "setup_time": "<当前时间>"
}
```

**写入命令（macOS/Linux）**：
```bash
cat > ~/.upmee/config.json << 'EOF'
{内容}
EOF
```

**写入命令（Windows PowerShell）**：
```powershell
'{内容}' | Out-File -FilePath "$env:USERPROFILE\.upmee\config.json" -Encoding UTF8
```

---

### 第三步：验证 API Key 可用性

发送一个简单的账号列表请求，page_size=1，验证 key 是否有效。

**macOS / Linux**：
```bash
curl -s -w "\n%{http_code}" \
  -X POST https://www.upmee.cc/open_api/media/list \
  -H "API-Key: <api_key>" \
  -H "Content-Type: application/json" \
  -d '{"page":1,"page_size":1,"team_id":0}'
```

**Windows（curl.exe）**：
```cmd
curl.exe -s -w "\n%{http_code}" -X POST https://www.upmee.cc/open_api/media/list -H "API-Key: <api_key>" -H "Content-Type: application/json" -d "{\"page\":1,\"page_size\":1,\"team_id\":0}"
```

**Windows（PowerShell，curl 不可用时）**：
```powershell
$resp = Invoke-RestMethod -Method POST -Uri "https://www.upmee.cc/open_api/media/list" `
  -Headers @{"API-Key"="<api_key>"} `
  -Body '{"page":1,"page_size":1,"team_id":0}' `
  -ContentType "application/json"
$resp | ConvertTo-Json
```

**解读结果**：
- HTTP 200 且 `code=0` → API Key 有效，初始化成功
- HTTP 200 且 `code!=0`（如团队不存在）→ API Key 本身有效，team_id 需要后续确认
- HTTP 401 → API Key 无效，请检查是否复制正确
- 连接失败 → 网络问题，请检查网络连接

---

### 第四步：下载 API 文档到本地

文档托管地址（每个文件约 5–30KB，共 8 个文件）：

```
https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/index.md
https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/media.md
https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/task.md
https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/comment.md
https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/stat.md
https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/message.md
https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/shop.md
https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/upload.md
```

创建文档目录并逐一下载：

**macOS / Linux**：
```bash
mkdir -p ~/.upmee/docs
docs=(index media task comment stat message shop upload)
total=${#docs[@]}
i=0
for doc in "${docs[@]}"; do
  i=$((i+1))
  printf "[%d/%d] 下载 %s.md ... " "$i" "$total" "$doc"
  curl -sL "https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/${doc}.md" \
    -o ~/.upmee/docs/${doc}.md
  size=$(wc -c < ~/.upmee/docs/${doc}.md | tr -d ' ')
  echo "✓ ${size}B"
done
echo ""
echo "✅ 全部下载完成（${total} 个文件）"
```

**Windows（PowerShell）**：
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.upmee\docs" | Out-Null
$docs = @("index","media","task","comment","stat","message","shop","upload")
$total = $docs.Count
for ($i = 0; $i -lt $total; $i++) {
  $doc = $docs[$i]
  Write-Progress -Activity "下载 upmee API 文档" `
    -Status "[$($i+1)/$total] ${doc}.md" `
    -PercentComplete ([int](($i / $total) * 100))
  $url = "https://cdn.upmee.cc/dreamerainc/docs/open_api_guide/$doc.md"
  $dest = "$env:USERPROFILE\.upmee\docs\$doc.md"
  Invoke-WebRequest -Uri $url -OutFile $dest
  $size = (Get-Item $dest).Length
  Write-Host "  ✓ [$($i+1)/$total] ${doc}.md (${size}B)"
}
Write-Progress -Activity "下载 upmee API 文档" -Completed
Write-Host ""
Write-Host "✅ 全部下载完成（$total 个文件）"
```

若下载失败（网络原因），告知用户：
> "文档下载失败，请检查网络连接后重新运行 `/upmee_setup`。API Key 已保存，无需重新填写。"

下载成功后，将文档路径追加到配置文件：
- macOS/Linux 路径：`~/.upmee/docs`
- Windows 路径：`%USERPROFILE%\.upmee\docs`

在 `config.json` 中补充字段：
```json
"docs_path": "<实际文档目录的绝对路径>"
```

---

### 第五步：输出配置完成提示

向用户展示以下信息：

```
✅ upmee 配置完成！

配置信息：
  API Key：<前8位>****（已安全保存）
  Base URL：https://www.upmee.cc
  操作系统：<检测到的OS>
  配置文件：<配置文件路径>
  API 文档：<docs_path>（8 个文件已下载）

现在可以直接使用 /upmee 执行任务，例如：
  "帮我查询所有绑定的账号"
  "帮我发布一个视频到 TikTok"
  "查看最近的发布任务状态"

如果接口文档有更新，重新运行 /upmee_setup 即可刷新本地文档。
```

如果 `team_id` 未提供，提示用户：
```
⚠️  未设置默认 team_id。执行需要 team_id 的接口时，请在描述中说明，例如：
  "用 team_id 12345 帮我查询账号列表"
```

---

## 注意事项

- API Key 以明文存储在本地配置文件中，请勿分享给他人
- 若需更换 API Key，重新执行 `/upmee_setup <新的key>` 即可覆盖
- 配置文件仅存储在本机，不会上传到任何服务器