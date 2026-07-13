---
name: workflow-installer
description: "一键安装 workflow 全流程开发工作流——说「装工作流」自动搞定"
---

# workflow Workflow 一键安装器

当用户说「装工作流」、`安装 workflow 工作流`、`/agents-workflow` 时，按以下步骤执行。

**如果用户说「升级工作流」**，跳转到最后的「一键升级」部分。

## 一句话安装流程

### 步骤1：克隆仓库

```powershell
# 克隆到临时目录
git clone --depth 1 https://github.com/TonyQ-AI/agents-workflow.git $env:TEMP/agents-workflow

# 如果 git clone 失败（网络问题），尝试下载 ZIP：
# Invoke-WebRequest -Uri "https://github.com/TonyQ-AI/agents-workflow/archive/refs/heads/main.zip" -OutFile $env:TEMP/workflow.zip
# Expand-Archive $env:TEMP/workflow.zip $env:TEMP/agents-workflow
```

### 步骤2：复制技能

```powershell
# 复制所有 40 个技能到项目
New-Item -Path ".workflow/skills" -ItemType Directory -Force | Out-Null
Copy-Item -Recurse "$env:TEMP/agents-workflow/skills/*" ".workflow/skills/" -Force
```

### 步骤3：配置 AGENTS.md（进度检查规则）

```powershell
$agentsRule = @"

## 进度检查规则
在任何操作开始之前，检查 docs/superpowers/plans/task_plan.md 是否存在。
如果存在，读取它以了解当前执行进度。
这一规则适用于所有对话，不论是否使用 wf-orchestrator 编排器。
"@
if (Test-Path "AGENTS.md") {
    Add-Content -Path "AGENTS.md" -Value $agentsRule
} else {
    Set-Content -Path "AGENTS.md" -Value $agentsRule
}
```

### 步骤4：配置 MCP 服务器（MiMo 多模态）

在 `workflow.toml` 或对应配置文件中添加：

```toml
[[plugins]]
name    = "mimo-multimodal"
command = "npx"
args    = ["-y", "tonyq-mimo-mcp-server"]
env     = {
  MIMO_API_URL  = "https://api.xiaomimimo.com/v1/chat/completions",
  MIMO_API_KEY = "${MIMO_API_KEY}"
}
call_timeout_seconds = 600
```


### 自动检测 MiMo 端点

用户提供 MIMO_API_KEY 后，自动测试两个端点：

1. 先用 key 请求 `https://api.xiaomimimo.com/v1/chat/completions`
   - 返回 200 → 标准 API，写入 `MIMO_API_URL = "https://api.xiaomimimo.com/v1/chat/completions"`
2. 如果返回 401 → 再试 `https://token-plan-cn.xiaomimimo.com/v1/chat/completions`
   - 返回 200 → Token Plan，写入对应的 URL
3. 两个都 401 → 提示用户「API key 无效，请检查」

无需用户手动选择，全自动判断。


### 步骤5：配置 MiMo API key（必须执行，不可跳过）

⚠️ **此步骤是强制交互。未完成前，禁止进入步骤6。**

**第一步 — 检测本地是否已有 MiMo key：**
检查当前工具的配置文件（如工具自己的 config、.env 等）中是否已存在 `MIMO_API_KEY`。

**第二步 — 根据检测结果交互：**

▸ 如果检测到已有 MiMo key：
> 🔍 检测到你已配置 MiMo API Key，是否直接使用？
> A) 直接使用 → 读取已有 key，自动检测端点，写入 MCP 配置
> B) 重新输入 → 按下方「另行提供」流程
> C) 跳过 → 不配置多模态，告知后果

▸ 如果未检测到：
> 请提供 MiMo API Key（图片/音频/视频理解需要，没有可跳过但多模态功能不可用）：
> 格式：sk-xxx***xxxx
> - 用户提供了 → 自动检测端点（见上方"自动检测 MiMo 端点"），写入 .env 和 MCP 配置
> - 用户明确说跳过 → 跳过 MCP 配置，**必须告知**："⚠️ 已跳过多模态配置，图片/音频/视频理解功能不可用"

**第三步 — 确认完成：**
> ✅ MiMo API Key 配置完成。是否继续验证安装？


### 步骤6：验证安装

确认以下关键文件已就位：

```
- [ ] .workflow/skills/wf-orchestrator/SKILL.md
- [ ] .workflow/skills/wf-orchestrator-engine/SKILL.md
- [ ] .workflow/skills/wf-architect/SKILL.md
- [ ] .workflow/skills/wf-developer/SKILL.md
- [ ] .workflow/skills/wf-tester/SKILL.md
- [ ] .workflow/skills/wf-reviewer/SKILL.md
- [ ] .workflow/skills/wf-deployer/SKILL.md
- [ ] AGENTS.md 包含进度检查规则
```

### 步骤7：报告结果

```
✅ workflow 全流程开发工作流安装完成！

已安装 40 个技能：
  📋 核心工作流  — 8 个（编排器+引擎+6子Agent）
  🏛️ 架构评审/扫描 — 1 个技能，2 种用法
  💡 方法论注入 — 9 个（TDD、系统化调试等）
  📐 规划与支撑 — 6 个（领域建模、进度跟踪等）
  🔧 扩展能力   — 6 个（UI设计、工作流执行器等）
  🎤 多模态 MCP — 4 个 MiMo 技能（含 tonyq-mimo-mcp-server）

📄 AGENTS.md 进度检查规则已就位
🔌 MiMo MCP 已配置（需设置 MIMO_API_KEY 环境变量）

使用方式：
  /wf-orchestrator <项目名>: <需求描述>

详细文档：
  https://github.com/TonyQ-AI/agents-workflow
```

## 一键升级（针对已安装用户）

当用户说「升级工作流」时，按以下步骤执行。

### 步骤0：检测旧版本并告知迁移（必须执行）

⚠️ **此步骤是强制交互。未完成前，禁止继续升级。**

检查以下特征判断用户是否使用旧版：
- 技能路径包含 `reasonix-workflow` 或 `workflow-task`
- 技能目录在 `.reasonix/skills/` 下
- MCP 配置使用 `mimo-mcp-server`（而非 `tonyq-mimo-mcp-server`）

> 如果检测到旧版，**必须先展示迁移说明**，等待用户确认后再继续：
>
> 📢 **工作流已升级为 agents-workflow！**
>
> 旧版 `reasonix-workflow` 已整合为全新 `agents-workflow`，主要变化：
> - 🏷️ **去品牌化**：Reasonix 品牌名已全面移除，改为中性名称
> - 🔀 **双模式自适应**：有 task 用正常模式，没有自动 fallback inline，所有工具都能用
> - 🛠️ **MCP 修复**：MiMo 迁移至 `tonyq-mimo-mcp-server`，修复 401/404 问题
> - 📦 **仓库迁移**：新地址 `github.com/TonyQ-AI/agents-workflow`（旧地址自动跳转）
>
> 是否继续升级？
> - 用户确认 → 继续步骤1
> - 用户拒绝 → 终止升级，保持旧版

> 如果未检测到旧版（已是新版），跳过此步骤，直接进入步骤1。

### 步骤1：拉取最新技能
```powershell
git clone --depth 1 https://github.com/TonyQ-AI/agents-workflow.git $env:TEMP/rw-upgrade
# 清理旧版残留
if (Test-Path ".reasonix") { Remove-Item -Recurse -Force ".reasonix" }
if (Test-Path "reasonix-workflow") { Remove-Item -Recurse -Force "reasonix-workflow" }
if (Test-Path "workflow-task") { Remove-Item -Recurse -Force "workflow-task" }
Copy-Item -Recurse "$env:TEMP/rw-upgrade/skills/*" ".workflow/skills/" -Force
```

### 步骤2：更新 MCP 配置（关键修复）
```powershell
$configPath = "workflow.toml"
if (Test-Path $configPath) {
    $content = Get-Content $configPath -Raw
    $content = $content -replace 'mimo-mcp-server', 'tonyq-mimo-mcp-server'
    $content = $content -replace 'MIMO_API_BASE', 'MIMO_API_URL'
    $content = $content -replace 'MIMO_API_URL = "https://api.xiaomimimo.com/v1"', 'MIMO_API_URL = "https://api.xiaomimimo.com/v1/chat/completions"'
    Set-Content -Path $configPath -Value $content -NoNewline
    Write-Output "MCP 配置已升级"
}
```

### 步骤3：清除 MCP 缓存
```powershell
$cacheFile = "$env:LOCALAPPDATA\workflow\mcp\mimo-multimodal.json"
if (Test-Path $cacheFile) { Remove-Item $cacheFile -Force; Write-Output "MCP 缓存已清除" }
```

### 步骤4：更新 AGENTS.md
```powershell
$agentsCheck = Select-String -Path "AGENTS.md" -Pattern "进度检查规则" -SimpleMatch -ErrorAction SilentlyContinue
if (-not $agentsCheck) { Add-Content -Path "AGENTS.md" -Value "`n## 进度检查规则`n在任何操作开始之前，检查 docs/superpowers/plans/task_plan.md 是否存在。`n如果存在，读取它以了解当前执行进度。" }
```

### 步骤5：报告升级结果
```
🎉 工作流升级完成！已从 reasonix-workflow 迁移至 agents-workflow！

✨ 新版本变化：
  🏷️  品牌中性化 — Reasonix 品牌名已全面移除
  🔀 双模式自适应 — 有 task 用正常模式，没有自动 fallback inline
  🛠️ MCP 修复 — mimo-mcp-server → tonyq-mimo-mcp-server（修复 401/404）
  📦 新仓库 — github.com/TonyQ-AI/agents-workflow

📋 技术细节：
  技能已更新到最新版（40 个）
  MCP 配置：mimo-mcp-server → tonyq-mimo-mcp-server
  环境变量：MIMO_API_BASE → MIMO_API_URL（带 /chat/completions）
  旧版残留已清理
  MCP 缓存已清除

请重启工具使新配置生效
```

## 注意事项

- 安装前会检测是否已存在技能，不覆盖用户自定义修改
- AGENTS.md 以追加方式写入，不覆盖已有内容
- 如果 git clone 失败会自动切到 ZIP 下载模式
- 安装完成后**建议重启工具** 使新配置生效
- MiMo MCP 需要 `MIMO_API_KEY` 环境变量（存 `.env` 不硬编码）
