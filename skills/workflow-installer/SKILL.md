---
name: workflow-installer
description: "agents-workflow 安装器——用户提供仓库地址即可自动完成全部安装配置"
---

# agents-workflow 安装器

当用户说「装工作流」、「安装 agents-workflow」时触发。用户需提供仓库地址（默认 `github.com/TonyQ-AI/agents-workflow`）。

**默认仓库地址**：`https://github.com/TonyQ-AI/agents-workflow`

> 如果用户只说了「装工作流」没给地址，先确认：
> 📦 将从 `github.com/TonyQ-AI/agents-workflow` 安装，是否继续？
> （你也可以提供自己的仓库地址或 fork 地址）
> - 用户确认 → 继续步骤1
> - 用户提供了其他地址 → 使用用户提供的地址

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


### 步骤4：配置 MiMo MCP（先拿 key，后写配置）

⚠️ **此步骤是强制交互。未完成前，禁止进入步骤5。**

**第〇步 — 判断是否需要 MiMo MCP：**
本工作流内置 MiMo MCP 提供多模态能力（图片分析、音频转写、视频理解等）。先检测你当前使用的推理模型是否已原生支持这些能力：

▸ 如果模型原生支持多模态（GPT-4o、Claude 3.5/4、Gemini 等）：
> 🔍 本工作流内置 MiMo MCP 提供多模态能力。检测到你的推理模型为：**{模型名}**
> ℹ️ 你当前的模型已原生支持多模态，无需 MiMo MCP 即可处理图片/音频/视频。
> 是否仍要安装 MiMo MCP？（如果以后切换至 DeepSeek 等不支持多模态的模型时有用）
> - **是** → 继续第一步，安装 MiMo MCP
> - **否** → 跳过 MiMo MCP，直接进入步骤5

▸ 如果模型不支持多模态（DeepSeek、Qwen 纯文本版等）：
> 🔍 本工作流内置 MiMo MCP 提供多模态能力。检测到你的推理模型为：**{模型名}**
> ⚠️ 你当前的模型不支持图片/音频/视频理解，需要 MiMo MCP 补充多模态能力。
> 是否安装 MiMo MCP？
> - **是** → 继续第一步，安装 MiMo MCP
> - **否** → 跳过 MiMo MCP，**必须告知**："⚠️ 已跳过 MiMo MCP，图片/音频/视频理解功能不可用"，进入步骤5

**第一步 — 检测本地是否已有 MiMo key：**
检查当前工具的配置文件（如工具自己的 config、.env 等）中是否已存在 `MIMO_API_KEY`。

**第二步 — 获取 key：**

▸ 如果检测到已有 MiMo key：
> 🔍 检测到你已配置 MiMo API Key，是否直接使用？
> - **是** → 读取已有 key，跳到第三步
> - **重新输入** → 提供新的 key

▸ 如果未检测到：
> 请提供 MiMo API Key：
> 格式：sk-xxx***xxxx

**第三步 — 自动检测端点（拿到 key 后立即执行，不可跳过）：**

⚠️ **不要猜测或写死 URL。用用户的 key 逐个测试，哪个通就用哪个。**

1. 先用 key 请求 `https://api.xiaomimimo.com/v1/chat/completions`
   - 返回 200 → 标准 API，写入 `MIMO_API_URL = "https://api.xiaomimimo.com/v1/chat/completions"`
2. 如果返回 401/403 → 再试 `https://token-plan-cn.xiaomimimo.com/v1/chat/completions`
   - 返回 200 → Token Plan，写入对应的 URL
3. 两个都失败 → 提示用户「API key 无效，请检查」，回到第二步重新询问

**第四步 — 写入 MCP 配置（用检测到的正确 URL）：**

检测成功后，写入工具配置文件：

```toml
[[plugins]]
name    = "mimo-multimodal"
command = "npx"
args    = ["-y", "tonyq-mimo-mcp-server"]
env     = {
  MIMO_API_URL  = "<检测到的URL>",
  MIMO_API_KEY  = "${MIMO_API_KEY}"
}
call_timeout_seconds = 600
```

同时将 key 写入 `.env`（先存后读规则）。

**第五步 — 确认完成：**
> ✅ MiMo MCP 配置完成（端点：<检测到的URL>）。是否继续验证安装？

### 步骤5：验证安装

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

### 步骤6：报告结果

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
> - 🏷️ **不再依赖单一工具**：品牌中性化，支持所有 AI coding 工具
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

### 步骤2：清除 MCP 缓存
```powershell
$cacheFile = "$env:LOCALAPPDATA\workflow\mcp\mimo-multimodal.json"
if (Test-Path $cacheFile) { Remove-Item $cacheFile -Force; Write-Output "MCP 缓存已清除" }
```

### 步骤3：配置 MiMo MCP

→ **直接执行上方安装流程的「步骤4：配置 MiMo MCP」**（先拿 key → 测端点 → 写正确配置）。
升级和安装共用同一套逻辑，不维护两套。

### 步骤4：更新 AGENTS.md
```powershell
$agentsCheck = Select-String -Path "AGENTS.md" -Pattern "进度检查规则" -SimpleMatch -ErrorAction SilentlyContinue
if (-not $agentsCheck) { Add-Content -Path "AGENTS.md" -Value "`n## 进度检查规则`n在任何操作开始之前，检查 docs/superpowers/plans/task_plan.md 是否存在。`n如果存在，读取它以了解当前执行进度。" }
```

### 步骤5：报告升级结果
```
🎉 工作流升级完成！已从 reasonix-workflow 迁移至 agents-workflow！

✨ 新版本变化：
  🏷️  不再依赖单一工具 — 品牌中性化，支持所有 AI coding 工具
  🔀 双模式自适应 — 有 task 用正常模式，没有自动 fallback inline
  🛠️ MCP 修复 — 自动检测端点，不再 401/404
  📦 新仓库 — github.com/TonyQ-AI/agents-workflow

📋 技术细节：
  技能已更新到最新版（40 个）
  MCP 缓存已清除
  MCP 端点已重新检测
  旧版残留已清理

请重启工具使新配置生效
```

## 注意事项

- 安装前会检测是否已存在技能，不覆盖用户自定义修改
- AGENTS.md 以追加方式写入，不覆盖已有内容
- 如果 git clone 失败会自动切到 ZIP 下载模式
- 安装完成后**建议重启工具** 使新配置生效
- MiMo MCP 需要 `MIMO_API_KEY` 环境变量（存 `.env` 不硬编码）
