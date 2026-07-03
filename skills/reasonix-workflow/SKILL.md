---
name: reasonix-workflow
description: "一键安装 Reasonix 全流程开发工作流——包含 7 步编排、进度跟踪、架构评审、UI 设计。需要 GitHub 仓库 URL 作为参数。"
---

# Reasonix Workflow 安装器

## 概述

本技能用于将 Reasonix 多Agent协同开发工作流安装到任意 Reasonix 项目中。

**包含 22 个技能，按用途分为 5 组：**

### 核心工作流（7个）
| 技能 | 作用 | 模型 |
|------|------|------|
| **wf-orchestrator** | 主编排器，串联 8 阶段全流程 | — |
| **wf-planner** | 需求分析、任务分解 | Pro 🧠 |
| **wf-architect** | 系统架构设计、接口定义 | Pro 🧠 |
| **wf-developer** | 代码实现、单元测试 | Flash ⚡ |
| **wf-tester** | 测试用例设计、执行 | Flash ⚡ |
| **wf-reviewer** | 代码审查、质量裁决 | Pro 🧠 |
| **wf-deployer** | 构建打包、部署方案 | Flash ⚡ |

### 架构评审与扫描（1个技能，2种用法）
| 技能 | 用法 | 作用 |
|------|------|------|
| **reasonix-arch-review** | 架构评审（编码前） | 专家模拟 + 6维评分 + Go/No-Go |
| | 架构扫描（编码后） | 删除测试 + 依赖图 + HTML报告 |

### 方法论注入（5个）
| 技能 | 注入阶段 | 作用 |
|------|---------|------|
| **test-driven-development** | 编码 | 红→绿→重构循环 |
| **systematic-debugging** | 编码（修Bug） | 定位→分析→假设→修复→验证 |
| **dispatching-parallel-agents** | 测试 | 并行执行独立测试 |
| **requesting-code-review** | 审查 | 逐文件标注行号 |
| **chinese-code-review** | 审查 | 建设性中文反馈 |

### 规划与支撑（5个）
| 技能 | 作用 |
|------|------|
| **brainstorming** | 需求分析、方案对比 |
| **writing-plans** | 计划生成 + 三文件跟踪 |
| **executing-plans** | 自动检测进度、中断续做 |
| **reasonix-domain-modeling** | 领域建模、通用语言、ADR |
| **reasonix-handoff** | 会话交接 |

### 扩展能力（4个）
| 技能 | 作用 |
|------|------|
| **reasonix-ui-design** | 专业UI设计（防AI模板） |
| **workflow-runner** | 通用 YAML 工作流执行器 |
| **finishing-a-development-branch** | 分支集成规范 |
| **verification-before-completion** | 完成前自动验证 |

**额外安装：**
- `AGENTS.md` — 进度检查兜底规则
- 可选：`mimo-multimodal` MCP 配置（需 MiMo API Key）

## 安装方式

### 直接跟我说

```
你：装 Reasonix 工作流
→ 我会引导你输入 GitHub 仓库 URL，自动完成安装
```

### 或用你准备好的仓库

如果你已经把自己的工作流推到 GitHub，直接给我仓库地址：

```
你：装 Reasonix 工作流，仓库 https://github.com/xxx/reasonix-workflow
→ 一键安装
```

## 仓库结构要求

你的 GitHub 仓库应包含以下文件：

```
reasonix-workflow/
├── README.md
├── LICENSE
│
├── skills/
│   ├── wf-orchestrator/SKILL.md
│   ├── writing-plans/SKILL.md
│   ├── executing-plans/SKILL.md
│   ├── reasonix-ui-design/SKILL.md
│   └── reasonix-arch-review/SKILL.md
│
├── configs/
│   ├── AGENTS.md              ← 进度检查规则
│   └── mcp-mimo.json          ← MiMo MCP 配置（可选）
│
└── scripts/
    └── install.ps1            ← Windows 安装脚本（可选）
    └── install.sh             ← Mac/Linux 安装脚本（可选）
```

## 安装流程

执行安装时按以下步骤操作：

### 1. 获取仓库地址

如果用户没提供仓库 URL，询问：
> "请提供你的 Reasonix 工作流 GitHub 仓库地址（如 `https://github.com/xxx/reasonix-workflow`）"

### 2. 下载并安装技能

```powershell
# 克隆仓库
git clone --depth 1 <仓库URL> $env:TEMP/reasonix-workflow

# 如果 git clone 失败（网络问题），尝试下载 ZIP：
# Invoke-WebRequest -Uri "<仓库URL>/archive/refs/heads/main.zip" -OutFile $env:TEMP/workflow.zip
```

### 3. 复制到项目

```powershell
# 复制所有技能
Copy-Item -Recurse "$env:TEMP/reasonix-workflow/skills/*" ".reasonix/skills/" -Force

# 复制 AGENTS.md（追加模式，不覆盖已有内容）
if (Test-Path "AGENTS.md") {
    Add-Content -Path "AGENTS.md" -Value (Get-Content "$env:TEMP/reasonix-workflow/configs/AGENTS.md")
} else {
    Copy-Item "$env:TEMP/reasonix-workflow/configs/AGENTS.md" "AGENTS.md"
}
```

### 4. 验证安装

确认以下关键文件已就位：

```markdown
- [ ] .reasonix/skills/wf-orchestrator/SKILL.md
- [ ] .reasonix/skills/wf-planner/SKILL.md
- [ ] .reasonix/skills/wf-architect/SKILL.md
- [ ] .reasonix/skills/wf-developer/SKILL.md
- [ ] .reasonix/skills/wf-tester/SKILL.md
- [ ] .reasonix/skills/wf-reviewer/SKILL.md
- [ ] .reasonix/skills/wf-deployer/SKILL.md
- [ ] .reasonix/skills/reasonix-arch-review/SKILL.md
- [ ] .reasonix/skills/brainstorming/SKILL.md
- [ ] .reasonix/skills/writing-plans/SKILL.md
- [ ] .reasonix/skills/executing-plans/SKILL.md
- [ ] .reasonix/skills/reasonix-domain-modeling/SKILL.md
- [ ] .reasonix/skills/reasonix-handoff/SKILL.md
- [ ] .reasonix/skills/reasonix-ui-design/SKILL.md
- [ ] .reasonix/skills/test-driven-development/SKILL.md
- [ ] .reasonix/skills/systematic-debugging/SKILL.md
- [ ] .reasonix/skills/dispatching-parallel-agents/SKILL.md
- [ ] .reasonix/skills/requesting-code-review/SKILL.md
- [ ] .reasonix/skills/chinese-code-review/SKILL.md
- [ ] AGENTS.md 包含进度检查规则
```

### 5. 报告结果

```
✅ Reasonix 多Agent协同开发工作流安装完成！

已安装 22 个技能：
  📋 核心工作流  — 7 个（规划→架构→编码→测试→审查→部署）
  🏛️ 架构评审/扫描 — 1 个技能，2 种用法
  💡 方法论注入 — 5 个（TDD、系统化调试等）
  📐 规划与支撑 — 5 个（领域建模、进度跟踪等）
  🔧 扩展能力   — 4 个（UI设计、工作流执行器等）

📄 AGENTS.md 进度检查规则已就位

使用方式：
  /wf-orchestrator <项目名>: <需求描述>

详细文档：
  仓库地址/README.md
```

## 分发给他人

想让别人也能用你的工作流：

1. 把项目推到 GitHub：`https://github.com/你的用户名/reasonix-workflow`
2. 告诉别人把本 SKILL.md 放到 `.reasonix/skills/reasonix-workflow/SKILL.md`
3. 别人说"装 Reasonix 工作流"，输入你的仓库地址即可

或者更简单——直接分享你的 GitHub 仓库地址，别人用以下命令一键装：

```bash
npx skills add https://github.com/你的用户名/reasonix-workflow --skill reasonix-workflow
```

## 注意事项

- 安装前会检测是否已存在同名技能，存在则跳过（不覆盖用户自定义修改）
- AGENTS.md 以追加方式写入，不覆盖已有内容
- 如果 git clone 失败会自动切到 ZIP 下载模式
- 安装完成后建议重启 Reasonix 使新技能生效
