# Reasonix Workflow

> 多 Agent 协同开发全流程自动化流水线 · 13 阶段 · 40 技能（+3 辅助独立技能） · 3 套多模态 MCP · 知识沉淀  
> 版本：4.0 · 仓库：https://github.com/TonyQ-AI/reasonix-workflow

---

## 概述

Reasonix Workflow 是一套多 Agent 协同开发流水线。你给一句话需求，编排器自动串联 **规划→领域建模→架构设计→架构评审→编码→测试→架构扫描→审查→部署→文档检查→分支收尾→版本登记→总结** 全流程。

### 核心能力

| 能力 | 说明 |
|:-----|:------|
| **交互式需求澄清** | 编排器与用户逐轮对话，每次一个问题，不猜测 |
| **领域建模** | 规划后提取实体关系与通用语言，消除术语歧义 |
| **原始需求交叉验证** | 每个子 Agent 对照用户确认的 specs 文件校验，防止需求漂移 |
| **智能模型分配** | Pro 推理（规划/架构/评审/审查），Flash 执行（编码/测试/部署/文档） |
| **方法论注入** | TDD / 系统化调试 / 代码审查 / 中文规范等按阶段自动注入 |
| **并行调度** | `--parallel` 参数实现编码与测试并行执行 |
| **断点续跑** | `--from` / `--to` + checkpoint 三文件体系，中断后可精确恢复 |
| **全产出门控** | 每个阶段末尾验证产出文件，不凭子 Agent 口头声称 |
| **轻量模式** | `--lite` 跳过架构评审和架构扫描，加速小型需求 |
| **自动 OCR** | 用户上传图片/PDF 需求时自动提取文字 |
| **大任务拆分** | 复杂编码自动拆分为并行子任务 |
| **知识沉淀** | 每次会话自动提炼关键发现，跨会话积累项目知识库 |

---

## 快速开始

```bash
# 标准全流程
/wf-orchestrator MediaManager: 新增视频批量导出功能，支持按标签筛选

# 轻量模式（跳架构评审+扫描）
/wf-orchestrator MediaManager: 修复登录页样式 --lite

# 断点恢复
/wf-orchestrator MediaManager: 新增导出 --from coding

# 编码测试并行
/wf-orchestrator MediaManager: 重构用户模块 --parallel
```

---

## 参数

| 参数 | 说明 |
|------|------|
| `--from <阶段>` | 起始阶段：planning / domain_modeling / architecture / arch_review / coding / testing / arch_scan / review / deploy / doc_check / branch_finish / version_tracking |
| `--to <阶段>` | 结束阶段（同上） |
| `--parallel` | 编码与测试并行执行 |
| `--lite` | 跳过架构评审（步骤5）和架构扫描（步骤8） |
| `--model <模型>` | 全局覆盖所有子 Agent 的模型（deepseek / 小米mimo） |
| `--timeout <秒>` | 子 Agent 超时，默认 300 秒（编码阶段 600 秒） |
| `--no-superpowers` | 禁用所有方法论注入 |

---

## 13 阶段流程

```
① 规划(交互) → ② 领域建模 → ③ 架构设计 → ④ 架构评审 → ⑤ 编码(+TDD)
→ ⑥ 测试 → ⑦ 架构扫描 → ⑧ 审查 → ⑨ 部署 → ⑩ 文档检查
→ ⑪ 分支收尾 → ⑫ 版本登记 → ⑬ 总结
```

### 各阶段详解

| # | 阶段 | 执行方式 | 模型 | 产出 | 交叉验证 |
|:--|:-----|:-------|:----:|:-----|:-------:|
| ① | 📋 规划 | 编排器+用户交互（inline） | Pro 🧠 | task_plan.md + specs/*.md | — |
| ②~⑬ | 🧩 领域建模~总结 | subagent 引擎（wf-orchestrator-engine） | Pro/Flash | 各阶段产出 | ✅ |

---

---

## 兜底机制：task() 工具不可用时的降级策略

工作流正常模式下，引擎通过 `task()` 工具派发子Agent在独立上下文中执行：

| 阶段 | 正常模式 | 兜底模式 |
|:-----|:-------|:--------|
| 架构设计 | task(wf-architect) | 引擎自行设计 |
| 编码 | task(wf-developer) | 引擎自行编码 |
| 测试 | task(wf-tester) | 引擎自行测试 |
| 审查 | task(wf-reviewer) | 引擎自行审查 |
| 部署 | task(wf-deployer) | 引擎自行部署 |

以下阶段始终由引擎直接执行，不依赖 `task()`：
领域建模、架构评审、架构扫描、文档检查、分支收尾、版本登记、总结

**工具兼容性：**

| 工具 | task() |
|:-----|:------:|
| ZCode | ✅ |
| Reasonix | ✅ |
| AutoClaw / Cursor 等 | ❓ 自动 fallback |

引擎启动时自动检测 `task()` 可用性，不可用时设置 `fallback=true`——**所有子Agent任务由引擎亲自完成，工作流不会中断。**

## 防需求漂移机制

每个子 Agent 同时收到两份输入：

1. **前序阶段的衍生文档**（承上：如架构设计读领域模型）
2. **用户确认的原始需求基准** `specs/*-design.md`（校验：对照原件查偏离）

每层独立交叉验证，发现偏离时显式标注。

---

## 技能体系（40个）

### 核心工作流（8个）

| 技能 | 角色 | 模型 |
|------|:----|:----:|
| wf-orchestrator | 主编排器（inline 交互层） | — |
| wf-orchestrator-engine | 执行引擎（subagent，独立上下文运行阶段2~13） | Pro 🧠 |
| wf-architect | 架构设计 | Pro 🧠 |
| wf-developer | 编码实现 | Flash ⚡ |
| wf-tester | 测试执行 | Flash ⚡ |
| wf-reviewer | 代码审查 | Pro 🧠 |
| wf-deployer | 部署发布 | Flash ⚡ |
| wf-planner | 参考技能（已被编排器取代） | — |

### 架构与设计（4个）

| 技能 | 作用 |
|------|------|
| reasonix-arch-review | 架构评审（6维评分 + Go/No-Go）+ 架构扫描（依赖检测 + 循环检测） |
| reasonix-domain-modeling | 实体提取、关系定义、通用语言建立 |
| reasonix-ui-design | 专业 UI/UX 设计（防AI模板化） |
| reasonix-handoff | 会话交接 |

### 方法论注入（7个）

| 技能 | 注入阶段 | 作用 |
|------|---------|------|
| brainstorming | 规划 | 需求探索与方案对比 |
| writing-plans | 规划 | 结构化任务分解 |
| executing-plans | 规划 | 三文件进度追踪（plan/progress/findings） |
| test-driven-development | 编码 | 红→绿→重构循环 |
| systematic-debugging | 编码 | 定位→分析→假设→修复→验证 |
| subagent-driven-development | 编码 | 大任务拆分为并行子 Agent |
| dispatching-parallel-agents | 测试 | 独立测试并行执行 |

### 审查与规范（4个）

| 技能 | 作用 |
|------|------|
| requesting-code-review | 逐文件标注行号 |
| receiving-code-review | 审查反馈后验证实施 |
| chinese-code-review | 符合国内团队文化的中文审查 |
| chinese-documentation | 排版、术语、结构规范 |

### 提交与版本（5个）

| 技能 | 作用 |
|------|------|
| chinese-commit-conventions | 中文 Git 提交规范 + changelog |
| chinese-git-workflow | Gitee/Coding/极狐 GitLab 工作流 |
| finishing-a-development-branch | 分支集成/PR/合并规范 |
| using-git-worktrees | 隔离 Git 工作树并行开发 |
| vt | 项目版本追踪 |

### 辅助工具（6个）

| 技能 | 作用 |
|------|------|
| trigger-commands | 醒醒/开新坑/存档/读档 |
| workflow-runner | 通用 YAML 工作流执行器 |
| verification-before-completion | 完成前自动验证产出 |
| writing-skills | 创建/编辑/验证技能文件 |
| mcp-builder | 系统化构建 MCP 工具 |
| using-superpowers | 技能查找与使用方法论 |

### 多模态 MCP（3套 / 5个技能）

| MCP | 技能 | 用途 |
|:----|:----|:-----|
| **MiMo 多模态** | mimo-image-understanding | 图片理解、OCR、UI 审查 |
| | mimo-audio-understanding | 音频转写、会议记录 |
| | mimo-video-understanding | 视频内容分析 |
| | mimo-tts | 文字转语音 |
| **百度 OCR** | baidu-unlimited-ocr | 图片/PDF 文字提取、发票/身份证识别（双模式：MCP SSE / REST API） |

---

## 配置

### 环境变量

```env
DEEPSEEK_API_KEY=sk-xxx
MIMO_API_KEY=sk-xxx
BAIDU_OCR_API_KEY=sk-xxx
```

### MCP 服务器

```toml
# MiMo 多模态
[[plugins]]
name    = "mimo-multimodal"
command = "npx"
args    = ["mimo-mcp-server", "-y"]
env     = { MIMO_API_KEY = "${MIMO_API_KEY}" }

# 百度 OCR（支持 SSE 的工具）
[[plugins]]
name    = "baidu-ocr-document"
type    = "http"
url     = "https://aip.baidubce.com/mcp/document/sse"
headers = { Authorization = "Bearer ${BAIDU_OCR_API_KEY}" }

# 百度 OCR（不支持 SSE 的工具 → REST API 直调）
# 额外需要: BAIDU_OCR_SECRET_KEY
# 详见 skills/baidu-unlimited-ocr/SKILL.md
```

---

## 会话目录结构

```
workflow/20260709_143000_新增批量导出/
├── task_plan.md               ← 规划产出（与 AGENTS.md 规则一致）
├── domain-model.md            ← 领域模型
├── 02-design.md               ← 架构设计
├── 03-arch-review.md          ← 架构评审
├── 03-implementation/         ← 编码产出
│   └── CHANGES.md
├── 04-test/                   ← 测试产出
│   └── TEST_REPORT.md
├── architecture-scan.html     ← 架构扫描报告
├── 05-review.md               ← 审查报告
├── 06-deploy/                 ← 部署方案
│   └── DEPLOY.md
├── doc-check-report.md        ← 文档检查报告
├── checkpoint.json            ← 断点恢复状态
├── SUMMARY.md                 ← 会话总结
└── ocr-extracted.md           ← OCR 提取结果（条件生成）
```

---


## 独立 YAML 工作流执行器

除了预制的 13 阶段开发流水线，还包含一个**通用 YAML 工作流执行器**（`workflow-runner`），支持运行任意自定义的多角色协作工作流。

### 与 wf-orchestrator 的区别

| | wf-orchestrator | workflow-runner |
|:--|:---------------|:----------------|
| 定位 | 预制的开发流水线 | 通用的工作流引擎 |
| 输入 | 一句话需求描述 | 一个 `.yaml` 工作流文件 |
| 阶段 | 固定的 13 阶段 | YAML 定义什么就跑什么 |
| 角色 | 固定的 7 个子 Agent | 任意自定义角色 |

### 使用方式

```bash
# 运行自定义 YAML 工作流
/workflow-runner workflows/story-creation.yaml

# YAML 示例结构
name: "故事创作工作流"
steps:
  - id: brainstorm
    role: "product/brainstormer"
    task: "根据主题生成 3 个故事创意"
  - id: review
    role: "editor/reviewer"
    task: "评审创意并选出最佳方案"
    depends_on: [brainstorm]

# 也适用于：PRD 评审、技术方案评估、多角色头脑风暴等
```

详见 `skills/workflow-runner/SKILL.md`。

---

## 完整文档

详见 [WORKFLOW_MANUAL.md](WORKFLOW_MANUAL.md) — 完整的13阶段执行说明、防幻觉机制、中断恢复、会话交接。

---

## 更新日志

### v4.0（2026-07-12）

- ✅ **架构重构**：wf-orchestrator 拆分为 inline 交互层 + subagent 执行引擎
- ✅ **原因**：原 1222 行 inline 技能在单轮上下文无法完整执行，导致阶段跳动和进度遗漏
- ✅ **task_plan.md 统一**：规划阶段产出 `task_plan.md`，与 AGENTS.md 进度检查规则对齐，子Agent 交叉验证基准
- ✅ **进度更新双重保障**：引擎内部强制更新 + 编排器兜底补写，不再遗漏
- ✅ **新增 wf-orchestrator-engine**：subagent 执行引擎，独立上下文完整跑完 12 个阶段
  

### v3.3（2026-07-09）

- ✅ 知识沉淀机制：每次会话结束自动提炼关键决策/经验/领域知识到 KNOWLEDGE.md
- ✅ 新会话自动加载项目知识库，子 Agent 获取跨会话积累的项目洞察
- ✅ 成本表/产出物列表/产出验证全面补全

### v3.2（2026-07-09）

- ✅ 原始需求交叉验证：每层子 Agent 对照 specs 校验，防止需求漂移
- ✅ `--parallel` 实现：编码与测试并行执行
- ✅ `--to` 实现：指定结束阶段后自动跳转总结
- ✅ `--from` 恢复：前置阶段产出摘要注入后续 prompt
- ✅ 百度 OCR 集成：自动检测图片/PDF → 提取文字 → 注入需求
- ✅ 大任务拆分：复杂编码自动拆并行子 Agent
- ✅ Git worktree 支持：分支隔离防 main 污染
- ✅ CHECKPOINT 恢复检测：中断后自动提示恢复
- ✅ 26 项 P0-P3 健壮性修复
- ✅ SUMMARY 模板 + 成本表 + 产出物列表全面补全

### v3.1（2026-07-09）

- 规划/领域建模改由编排器直接执行（非 task）
- 步骤编号修正、checkpoint 与 --from 命名统一（snake_case）

### v3.0（2026-07-07）

- 从 8 阶段扩展为 13 阶段
- 新增领域建模、文档检查、分支收尾、版本登记
- wf-planner 改为参考技能

---

## 许可证

MIT
