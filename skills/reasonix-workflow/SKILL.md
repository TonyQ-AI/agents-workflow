---
name: reasonix-workflow
description: "一键安装 Reasonix 全流程开发工作流——说「装工作流」自动搞定"
---

# Reasonix Workflow 一键安装器

当用户说「装工作流」、`安装 Reasonix 工作流`、`/reasonix-workflow` 时，按以下步骤执行。

**如果用户说「升级工作流」**，跳转到最后的「一键升级」部分。

## 一句话安装流程

### 步骤1：克隆仓库

```powershell
# 克隆到临时目录
git clone --depth 1 https://github.com/TonyQ-AI/reasonix-workflow.git $env:TEMP/reasonix-workflow

# 如果 git clone 失败（网络问题），尝试下载 ZIP：
# Invoke-WebRequest -Uri "https://github.com/TonyQ-AI/reasonix-workflow/archive/refs/heads/main.zip" -OutFile $env:TEMP/workflow.zip
# Expand-Archive $env:TEMP/workflow.zip $env:TEMP/reasonix-workflow
```

### 步骤2：复制技能

```powershell
# 复制所有 40 个技能到项目
New-Item -Path ".reasonix/skills" -ItemType Directory -Force | Out-Null
Copy-Item -Recurse "$env:TEMP/reasonix-workflow/skills/*" ".reasonix/skills/" -Force
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

在 `reasonix.toml` 或对应配置文件中添加：

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

### 步骤5：配置 API key

**提示用户**设置以下环境变量（存入 `~/.reasonix/.env`）：

```env
DEEPSEEK_API_KEY=sk-xxxxx    # 工作流推理用（必填）
MIMO_API_KEY=sk-xxxxx         # 多模态分析用（可选）
```

如果用户提供了 key，立即写入 `.env` 文件（先存后读规则）。

### 步骤6：验证安装

确认以下关键文件已就位：

```
- [ ] .reasonix/skills/wf-orchestrator/SKILL.md
- [ ] .reasonix/skills/wf-orchestrator-engine/SKILL.md
- [ ] .reasonix/skills/wf-architect/SKILL.md
- [ ] .reasonix/skills/wf-developer/SKILL.md
- [ ] .reasonix/skills/wf-tester/SKILL.md
- [ ] .reasonix/skills/wf-reviewer/SKILL.md
- [ ] .reasonix/skills/wf-deployer/SKILL.md
- [ ] AGENTS.md 包含进度检查规则
```

### 步骤7：报告结果

```
✅ Reasonix 全流程开发工作流安装完成！

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
  https://github.com/TonyQ-AI/reasonix-workflow
```

## 一键升级（针对已安装用户）

当用户说「升级工作流」时，自动执行以下操作：

### 1. 拉取最新技能
```powershell
git clone --depth 1 https://github.com/TonyQ-AI/reasonix-workflow.git $env:TEMP/rw-upgrade
Copy-Item -Recurse "$env:TEMP/rw-upgrade/skills/*" ".reasonix/skills/" -Force
```

### 2. 更新 MCP 配置（关键修复）
```powershell
$configPath = "reasonix.toml"
if (Test-Path $configPath) {
    $content = Get-Content $configPath -Raw
    $content = $content -replace 'mimo-mcp-server', 'tonyq-mimo-mcp-server'
    $content = $content -replace 'MIMO_API_BASE', 'MIMO_API_URL'
    $content = $content -replace 'MIMO_API_URL = "https://api.xiaomimimo.com/v1"', 'MIMO_API_URL = "https://api.xiaomimimo.com/v1/chat/completions"'
    Set-Content -Path $configPath -Value $content -NoNewline
    Write-Output "MCP 配置已升级"
}
```

### 3. 清除 MCP 缓存
```powershell
$cacheFile = "$env:LOCALAPPDATA\reasonix\mcp\mimo-multimodal.json"
if (Test-Path $cacheFile) { Remove-Item $cacheFile -Force; Write-Output "MCP 缓存已清除" }
```

### 4. 更新 AGENTS.md
```powershell
$agentsCheck = Select-String -Path "AGENTS.md" -Pattern "进度检查规则" -SimpleMatch -ErrorAction SilentlyContinue
if (-not $agentsCheck) { Add-Content -Path "AGENTS.md" -Value "`n## 进度检查规则`n在任何操作开始之前，检查 docs/superpowers/plans/task_plan.md 是否存在。`n如果存在，读取它以了解当前执行进度。" }
```

### 5. 报告升级结果
```
工作流升级完成！
  技能已更新到最新版（40 个）
  MCP 配置：mimo-mcp-server → tonyq-mimo-mcp-server
  环境变量：MIMO_API_BASE → MIMO_API_URL（带 /chat/completions）
  MCP 缓存已清除

请重启 Reasonix 使新配置生效
```

## 注意事项

- 安装前会检测是否已存在技能，不覆盖用户自定义修改
- AGENTS.md 以追加方式写入，不覆盖已有内容
- 如果 git clone 失败会自动切到 ZIP 下载模式
- 安装完成后**建议重启 Reasonix** 使新技能生效
- MiMo MCP 需要 `MIMO_API_KEY` 环境变量（存 `.env` 不硬编码）
