# Reasonix Workflow 完整工作流说明书

> 版本：2.1 · 最后更新：2026-07-09

---

## 目录

1. [概述](#1-概述)
2. [快速开始](#2-快速开始)
3. [系统架构](#3-系统架构)
4. [8 阶段工作流详解](#4-8-阶段工作流详解)
5. [上下文传递机制](#5-上下文传递机制)
6. [进度跟踪与 checkpoint 系统](#6-进度跟踪与-checkpoint-系统)
7. [方法论智能注入](#7-方法论智能注入)
8. [参数参考](#8-参数参考)
9. [错误处理与会话恢复](#9-错误处理与会话恢复)
10. [多模态 MCP 集成](#10-多模态-mcp-集成)
11. [技能体系](#11-技能体系)
12. [配置说明](#12-配置说明)

---

## 1. 概述

Reasonix Workflow 是一套**多 Agent 协同开发流水线**，通过 6 个专业化子 Agent（规划→架构→编码→测试→审查→部署）分工协作，配合编排器的**交互式需求澄清**和**智能方法论注入**，实现软件开发全流程自动化。

### 核心理念

| 原则 | 说明 |
|------|------|
| **交互式澄清** | 规划阶段编排器亲自与用户逐轮对话，每次一个问题，避免子Agent"盲猜"需求 |
| **分工专业化** | 每个 Agent 只做自己最擅长的事，Pro 模型负责推理密集型任务，Flash 负责执行型任务 |
| **方法论驱动** | 根据任务类型智能注入对应的方法论（TDD / 系统化调试），减少无干扰 |
| **容错设计** | task 不可用时自动降级为单Agent顺序执行，模型超时有 timeout 兜底 |
| **可恢复** | checkpoint + progress.md 双保险，中断后自动从断点续跑 |

---

## 2. 快速开始

### 一键启动完整工作流

```
/wf-orchestrator 项目名: 需求描述
```

### 轻量模式（跳过架构评审和扫描）

```
/wf-orchestrator 项目名: 需求描述 --lite
```

### 局部执行

```
/wf-orchestrator 项目名: 需求描述 --from dev --to review
```

### 覆盖模型

```
/wf-orchestrator 项目名: 需求描述 --model deepseek
```

### 设置超时

```
/wf-orchestrator 项目名: 需求描述 --timeout 600
```

---

## 3. 系统架构

### 角色分工

| 角色 | 技能 | 模型 | 职责 |
|------|------|------|------|
| **编排器** | wf-orchestrator | — | 总控流程、交互式需求澄清、进度跟踪、checkpoint 管理 |
| **架构师** | wf-architect | DeepSeek Pro 🧠 | 系统设计、技术选型、接口定义 |
| **编码器** | wf-developer | DeepSeek Flash ⚡ | 代码实现、单元测试 |
| **测试员** | wf-tester | DeepSeek Flash ⚡ | 测试用例设计、执行 |
| **审查员** | wf-reviewer | DeepSeek Pro 🧠 | 代码审查、质量检查 |
| **部署员** | wf-deployer | DeepSeek Flash ⚡ | 构建打包、部署方案 |

### 数据流

```
用户输入
  │
  ▼
┌─────────────────────────────────────────────────┐
│            编排器 (wf-orchestrator)               │
│                                                 │
│  ① 解析参数 → ② 建会话目录 → ③ 写 MANIFEST      │
│  ④ 交互式需求澄清 (亲自对话, 逐轮提问)             │
│     ↓ 需求明确, 用户批准                           │
│  ⑤ 派发子Agent (或降级顺序执行)                    │
│  ⑥ 验证产出 → 更新进度 → 写 checkpoint             │
│  ⑦ 生成总结 SUMMARY.md                            │
└─────────────────────────────────────────────────┘
         │
         │ task 派发 (或 FALLBACK 顺序执行)
         ▼
┌──────┬──────┬──────┬──────┬──────┬──────┐
│ 架构  │ 编码  │ 测试  │ 审查  │ 部署  │      │
│ Pro  │ Flash│ Flash│ Pro  │ Flash│      │
└──────┴──────┴──────┴──────┴──────┴──────┘
         │
         ▼
┌─────────────────────────────────────────────────┐
│              进度文件 + checkpoint                │
│  progress.md  findings.md  checkpoint.json       │
│  AGENTS.md (自动检测中断 → 触发恢复)               │
└─────────────────────────────────────────────────┘
```

---

## 4. 8 阶段工作流详解

### 阶段① 规划：交互式需求澄清

**执行者：编排器（亲自对话，不使用子Agent）**

这是与传统多Agent工作流最大的区别——编排器**不派发规划子Agent**，而是直接在对话中与用户交互：

```
编排器 → "这个功能主要给谁用的？管理员还是普通用户？"
用户   → "管理员"
编排器 → "权限模型用 RBAC 还是 ACL？"
用户   → "RBAC"
...（每次一个问题，逐步澄清）
编排器 → "需求明确了，这是设计方案..."
用户   → "可以"
编排器 → "这是实施计划，请确认"
用户   → "确认"
  → 进入下一阶段
```

**关键约束：**
- 每次只问一个问题
- 优先用选择题
- 边问边将业务术语记入 `CONTEXT.md`
- 用户确认设计方案和计划后才进入架构阶段

**产出：**
- `{SESSION_DIR}/01-plan.md` — 实施计划
- `docs/superpowers/specs/YYYY-MM-DD-<project>-design.md` — 设计规格

---

### 阶段② 架构设计

**执行者：wf-architect (DeepSeek Pro 🧠)**

**自动读取：**
- `{SESSION_DIR}/01-plan.md` — 规划产出
- `docs/superpowers/domain-models/*.md` — 领域模型

**产出：** `{SESSION_DIR}/02-design.md`

**如果 task 不可用 (`$FALLBACK=true`)：** 编排器自行完成架构设计。

---

### 阶段③ 架构评审

**执行者：编排器（注入 reasonix-arch-review 方法论）**

> **`--lite` 模式：此阶段跳过**

**流程：**
1. 模拟 2-3 位领域专家视角进行架构评审
2. 6 维度健康评分（模块化、耦合度、可测试性、演进能力、技术债务、接口设计）
3. 满分 24 分：🟢 ≥18 健康 / 🟡 12-17 一般 / 🔴 <12 严重
4. 🟡 时在编码 prompt 中附带待修复项
5. 🔴 时询问用户是否返回架构阶段重做

**产出：** `{SESSION_DIR}/03-arch-review.md`

---

### 阶段④ 编码

**执行者：wf-developer (DeepSeek Flash ⚡)**

**自动读取：**
- `{SESSION_DIR}/01-plan.md`
- `{SESSION_DIR}/02-design.md`
- `{SESSION_DIR}/03-arch-review.md`（含健康评分和待修复项）

**方法论注入（智能选择）：**
- 新功能开发 → 注入 TDD（红→绿→重构）
- 修 Bug → 注入 systematic-debugging（定位→分析→修复→验证）
- 改配置 → 不注入方法论

**产出：**
- 代码文件（直接修改项目文件）
- `{SESSION_DIR}/03-implementation/CHANGES.md`

---

### 阶段⑤ 测试

**执行者：wf-tester (DeepSeek Flash ⚡)**

**方法论注入：** dispatching-parallel-agents

**自动读取：** 规划、设计、代码变更

**产出：** `{SESSION_DIR}/04-test/TEST_REPORT.md`

---

### 阶段⑥ 架构扫描

**执行者：编排器（注入 reasonix-arch-review 用法二）**

> **`--lite` 模式：此阶段跳过**

**流程：**
1. 读取 CONTEXT.md 获取通用语言
2. 分析模块结构和依赖
3. 对每个模块执行"删除测试"
4. 生成 HTML 诊断报告（含 Mermaid 依赖图）
5. 扫描结果写入 findings.md

---

### 阶段⑦ 代码审查

**执行者：wf-reviewer (DeepSeek Pro 🧠)**

**自动读取：**
- 规划、设计、变更、测试报告
- `docs/superpowers/plans/findings.md`（架构扫描发现）

**方法论注入：** requesting-code-review + chinese-code-review

**产出：** `{SESSION_DIR}/05-review.md`

---

### 阶段⑧ 部署

**执行者：wf-deployer (DeepSeek Flash ⚡)**

**自动读取：**
- 规划、设计、变更
- `{SESSION_DIR}/04-test/TEST_REPORT.md`
- `{SESSION_DIR}/05-review.md`

**产出：** `{SESSION_DIR}/06-deploy/DEPLOY.md`

---

## 5. 上下文传递机制

### 传递链

```
规划 (编排器交互)  →  01-plan.md
      │
      ▼
领域模型 (domain-models/*.md)
      │
      ▼
架构 (wf-architect)  →  02-design.md
      │
      ▼
架构评审  →  03-arch-review.md
      │
      ▼
编码 (wf-developer)  →  03-implementation/CHANGES.md
      │  同时自动读取: 01-plan.md + 02-design.md + 03-arch-review.md
      ▼
测试 (wf-tester)  →  TEST_REPORT.md
      │
      ▼
架构扫描  →  findings.md
      │
      ▼
审查 (wf-reviewer)  →  05-review.md
      │  同时自动读取: 规划+设计+变更+测试报告+findings.md
      ▼
部署 (wf-deployer)  →  DEPLOY.md
      │  同时自动读取: 规划+设计+变更+测试报告+审查报告
      ▼
总结 SUMMARY.md
```

### 修复的断裂点

| 问题 | 修复 |
|------|------|
| 领域模型→架构师断裂 | wf-architect 新增读取 `domain-models/*.md` |
| 架构评审→编码器断裂 | wf-developer 新增读取 `03-arch-review.md` |
| 架构扫描→审查器断裂 | wf-reviewer 新增读取 `findings.md` |
| 测试报告→部署器断裂 | wf-deployer 新增读取 `TEST_REPORT.md` |

---

## 6. 进度跟踪与 checkpoint 系统

### 三文件体系

| 文件 | 路径 | 用途 |
|------|------|------|
| `progress.md` | `docs/superpowers/plans/` | 阶段进度可视化（✅/⏳/❌/⏭️） |
| `findings.md` | `docs/superpowers/plans/` | 每个阶段的关键发现和决策记录 |
| `checkpoint.json` | `{SESSION_DIR}/` | 机器可读的断点记录，用于中断恢复 |

### findings.md 格式

```markdown
# 发现记录 - 项目名

| 时间 | 阶段 | 发现 | 影响 |
|------|------|------|------|
| 07-09 | 规划 | 用户选择 RBAC 权限模型 | 后续模块需按角色鉴权 |
| 07-09 | 编码 | 第三方 API 调用需添加重试 | 添加 retry 中间件 |
```

### checkpoint.json 格式

```json
{
  "project": "项目名",
  "session_dir": "workflow/20260709_120000_项目名/",
  "started_at": "2026-07-09 12:00",
  "last_completed_phase": "coding",
  "lite_mode": false,
  "fallback_mode": false
}
```

### 恢复流程

1. AGENTS.md 在会话启动时检测 `docs/superpowers/plans/task_plan.md` 或 `checkpoint.json`
2. 读取 `last_completed_phase` 确定断点
3. 编排器从该阶段后继续执行
4. progress.md 中已完成阶段自动标记为 ✅

---

## 7. 方法论智能注入

编排器不再无脑全注入，而是根据**任务类型自动匹配**：

| 任务类型 | 注入的方法论 | 适用阶段 |
|---------|-------------|---------|
| 新功能开发 | TDD（红→绿→重构） | 编码 |
| Bug 修复 | systematic-debugging（定位→分析→修复→验证） | 编码 |
| 配置修改 | **不注入** | 编码 |
| 测试执行 | dispatching-parallel-agents | 测试 |
| 代码审查 | requesting-code-review + chinese-code-review | 审查 |
| 不确定 | 先问用户"这个需求偏新功能还是修Bug？" | — |

---

## 8. 参数参考

| 参数 | 说明 | 示例 |
|------|------|------|
| `--from <阶段>` | 从指定阶段开始 | `--from dev` |
| `--to <阶段>` | 在指定阶段结束 | `--to review` |
| `--model <模型名>` | 全局覆盖模型 | `--model deepseek` |
| `--lite` | 轻量模式（跳过架构评审和扫描） | `--lite` |
| `--timeout <秒>` | 子Agent超时 | `--timeout 600` |
| `--parallel` | 编码与测试并行 | `--parallel` |
| `--no-superpowers` | 禁用方法论注入 | `--no-superpowers` |

### `--lite` 模式跳过的阶段

```
完整模式: 规划 → 架构 → 架构评审 → 编码 → 测试 → 架构扫描 → 审查 → 部署
lite 模式: 规划 → 架构 → [跳过]  → 编码 → 测试 → [跳过]    → 审查 → 部署
```

### FALLBACK 降级

当 `task` 工具不可用时自动触发，所有子Agent阶段改为编排器**单Agent顺序执行**：

```
正常: 编排器 → task 派发子Agent → 等待完成 → 验证
降级: 编排器 → 自行完成该阶段任务  → 写产出 → 验证
```

---

## 9. 错误处理与会话恢复

### 错误场景

| 场景 | 处理方式 |
|------|---------|
| 子Agent 超时 | 记录到 ERRORS.log，询问：重试 / 降级顺序执行 / 跳过 / 中止 |
| 产出验证失败 | 提示缺失文件，提供重试或跳过 |
| 子Agent 调用失败 | 记录错误，从 checkpoint 恢复 |
| 构建失败 | 记录详情，给出修复建议 |

### 会话恢复

1. 检测 `checkpoint.json` 存在 → 读取 `last_completed_phase`
2. 检测 `progress.md` → 确认哪些阶段已完成
3. 编排器从断点后第一个未完成阶段继续
4. 恢复时告知用户当前断点位置

### 兜底规则

`AGENTS.md` 在任何会话启动时自动检测 `docs/superpowers/plans/task_plan.md`，即使没有编排器也能触发进度恢复。

---

## 10. 多模态 MCP 集成

工作流已集成 `mimo-multimodal` MCP 服务器（基于 `mimo-mcp-server` npm 包），提供：

| 工具 | 用途 |
|------|------|
| `mimo_understand_image` | 图片理解、OCR、UI 评审、图表提取 |
| `mimo_understand_audio` | 音频转写、会议记录、内容摘要 |
| `mimo_understand_video` | 视频描述、教程总结、动作检测 |
| `mimo_speech_recognition` | 语音识别 (ASR) |
| `mimo_speech_synthesis` | 语音合成 (TTS) |
| `mimo_voice_design` | 自定义音色设计 |
| `mimo_voice_clone` | 音色克隆 |

**在以下阶段可用：**
- 📋 规划 — 用户提供截图/设计稿时用图片 MCP 分析
- 🧪 测试 — 验证 UI 截图、分析测试日志
- 💻 编码 — 参考前端截图、处理媒体文件
- 🔍 审查 — 审查 UI 实现与设计稿差异

---

## 11. 技能体系

共 36 个技能，分为 7 组：

| 分组 | 技能 | 数量 |
|------|------|:----:|
| **核心工作流** | wf-orchestrator, wf-planner, wf-architect, wf-developer, wf-tester, wf-reviewer, wf-deployer | 7 |
| **架构评审/扫描** | reasonix-arch-review（两种用法） | 1 |
| **方法论注入** | test-driven-development, systematic-debugging, dispatching-parallel-agents, requesting-code-review, chinese-code-review | 5 |
| **规划与支撑** | brainstorming, writing-plans, executing-plans, reasonix-domain-modeling, reasonix-handoff | 5 |
| **扩展能力** | reasonix-ui-design, workflow-runner, finishing-a-development-branch, verification-before-completion, subagent-driven-development | 5 |
| **多模态** | mimo-image-understanding, mimo-audio-understanding, mimo-video-understanding, mimo-tts | 4 |
| **其他** | baidu-unlimited-ocr, chinese-documentation, chinese-git-workflow, chinese-commit-conventions, trigger-commands, vt, using-git-worktrees | 7 |

---

## 12. 配置说明

### 环境变量

在 `F:\ReasonixTest\.env` 中配置：

```env
DEEPSEEK_API_KEY=sk-xxx
MIMO_API_KEY=sk-xxx
BAIDU_OCR_API_KEY=bce-v3/xxx
```

### reasonix.toml

在 `F:\projects\reasonix.toml` 中配置：

```toml
[sandbox]
workspace_root = "F:\\projects"
bash    = "off"
network = true

[[plugins]]
name    = "mimo-multimodal"
command = "npx"
args    = ["mimo-mcp-server", "-y"]
env     = { MIMO_API_KEY = "${MIMO_API_KEY}" }
```

### 文件结构

```
F:\projects\
├── .reasonix\skills\     ← 36 个技能
├── reasonix.toml         ← MCP + sandbox 配置
├── AGENTS.md             ← 进度检查兜底规则
├── WORKFLOW_MANUAL.md    ← 本文档
├── docs\
│   ├── superpowers\
│   │   ├── plans\        ← progress.md, findings.md, task_plan.md
│   │   ├── specs\        ← 设计规格
│   │   └── domain-models\← 领域模型
│   └── adr\              ← 架构决策记录
└── workflow\             ← 会话产出目录
    └── YYYYMMDD_HHmmss_项目名\
        ├── MANIFEST.md
        ├── 01-plan.md
        ├── 02-design.md
        ├── 03-arch-review.md
        ├── 03-implementation\
        ├── 04-test\TEST_REPORT.md
        ├── 05-review.md
        ├── 06-deploy\DEPLOY.md
        ├── SUMMARY.md
        ├── checkpoint.json
        └── ERRORS.log
```
