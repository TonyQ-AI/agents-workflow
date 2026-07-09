# Reasonix Workflow 完整工作流说明书

> 版本：3.3 · 最后更新：2026-07-09

---

## 目录

1. [概述](#1-概述)
2. [快速开始](#2-快速开始)
3. [系统架构](#3-系统架构)
4. [13 阶段工作流详解](#4-13-阶段工作流详解)
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

Reasonix Workflow 是一套**多 Agent 协同开发流水线**，通过编排器 + 6 个专业化子 Agent 分工协作，配合**交互式需求澄清**、**领域建模**、**智能方法论注入**和**完整的产出门控**，实现软件开发全流程自动化。

### 核心理念

| 原则 | 说明 |
|------|------|
| **交互式澄清** | 规划阶段编排器亲自与用户逐轮对话，每次一个问题，避免子Agent"盲猜"需求 |
| **领域驱动** | 规划后执行领域建模，提取实体关系、通用语言，确保设计与业务一致 |
| **分工专业化** | Pro 模型负责推理密集型任务（规划/架构/评审），Flash 负责执行型任务（编码/测试/部署） |
| **方法论驱动** | 根据任务类型智能注入对应方法论（TDD / 系统化调试 / 代码审查），减少无干扰 |
| **容错设计** | task 不可用时自动降级为单Agent顺序执行，模型超时有 timeout 兜底 |
| **全产出门控** | 每个阶段都有明确的产出验证，不依赖子Agent口头声称 |
| **可恢复** | checkpoint + progress.md + findings.md 三文件体系，中断后自动续跑 |
| **完整交付链** | 编码后自动执行文档检查、分支收尾、版本登记，不留遗漏 |

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
| **编排器** | wf-orchestrator | — | 总控流程、交互式需求澄清、进度跟踪、checkpoint 管理、文档检查、分支收尾、版本登记 |
| **领域建模** | reasonix-domain-modeling | DeepSeek Pro 🧠 | 提取领域概念、实体关系、通用语言 |
| **架构师** | wf-architect | DeepSeek Pro 🧠 | 系统设计、技术选型、接口定义 |
| **编码器** | wf-developer | DeepSeek Flash ⚡ | 代码实现、单元测试（含UI设计按需、产出验证） |
| **测试员** | wf-tester | DeepSeek Flash ⚡ | 测试用例设计、并行执行 |
| **审查员** | wf-reviewer | DeepSeek Pro 🧠 | 代码审查、质量检查、安全检查 |
| **部署员** | wf-deployer | DeepSeek Flash ⚡ | 构建打包、部署方案 |

### 数据流

```
用户输入
  │
  ▼
┌──────────────────────────────────────────────────────────┐
│                   编排器 (wf-orchestrator)                  │
│                                                          │
│  ① 解析参数 → ② 建会话目录 → ③ 写 MANIFEST               │
│  ④ 初始化进度文件 + checkpoint                            │
│  ⑤ 检查基础设施 (task可用? → $FALLBACK)                   │
│                                                          │
│  ⑥ 交互式需求澄清 ── 亲自对话, 逐轮提问 ── 用户批准后继续     │
│  ⑦ 派发子Agent (或降级顺序执行)                             │
│  ⑧ 验证产出 → 更新进度 → 写 checkpoint                      │
│  ⑨ 文档质量检查 → 分支收尾 → 版本登记                       │
│  ⑩ 生成 SUMMARY.md                                         │
└──────────────────────────────────────────────────────────┘
         │
         │ task 派发 (或 $FALLBACK 顺序执行)
         ▼
┌──────┬──────┬──────┬──────┬──────┬──────┐
│ 领域   │ 架构  │ 编码  │ 测试  │ 审查  │ 部署  │
│ Pro   │ Pro  │ Flash│ Flash│ Pro  │ Flash│
└──────┴──────┴──────┴──────┴──────┴──────┘
         │
         ▼
┌──────────────────────────────────────────────────────────┐
│              三文件进度跟踪系统                              │
│  progress.md  findings.md  checkpoint.json                │
│  AGENTS.md (自动检测中断 → 触发恢复)                        │
│                                                          │
│  完成后自动：文档检查 → 分支收尾+提交 → 版本登记              │
└──────────────────────────────────────────────────────────┘
```

---

## 4. 13 阶段工作流详解

### 完整流程一览

```
┌──────┬──────────┬──────────────┬──────────────────────────┐
│ 步骤 │ 阶段      │ 执行者        │ 产出                     │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  1   │ 初始化    │ 编排器       │ MANIFEST.md              │
│      │          │              │ progress.md (初始化)      │
│      │          │              │ checkpoint.json          │
│      │          │              │ 基础设施检查               │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  2   │ ①规划    │ 编排器(交互) │ 01-plan.md               │
│      │          │              │ specs/*-design.md        │
│      │          │              │ CONTEXT.md (术语)         │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  3   │ ②领域建模 │ task→Pro   │ domain-model.md          │
│      │          │              │ CONTEXT.md (追加)         │
│      │          │              │ ADR (可选)                │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  4   │ ③架构    │ task→Pro   │ 02-design.md              │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  5   │ ④架构评审 │ 编排器      │ 03-arch-review.md         │
│      │          │ (--lite跳过) │ 健康评分 + Go/No-Go       │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  6   │ ⑤编码    │ task→Flash │ 代码文件                   │
│      │          │              │ CHANGES.md               │
│      │          │              │ (UI设计按需触发)           │
│      │          │              │ (产出验证自动执行)          │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  7   │ ⑥测试    │ task→Flash │ TEST_REPORT.md            │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  8   │ ⑦架构扫描 │ 编排器      │ HTML 诊断报告             │
│      │          │ (--lite跳过) │ findings.md (追加)        │
├──────┼──────────┼──────────────┼──────────────────────────┤
│  9   │ ⑧审查    │ task→Pro   │ 05-review.md              │
├──────┼──────────┼──────────────┼──────────────────────────┤
│ 10   │ ⑨部署    │ task→Flash │ DEPLOY.md                 │
├──────┼──────────┼──────────────┼──────────────────────────┤
│ 11   │ ⑩文档检查 │ 编排器      │ 文档质量报告               │
├──────┼──────────┼──────────────┼──────────────────────────┤
│ 12   │ ⑪分支收尾 │ 编排器      │ git commit + (可选推送上库)│
├──────┼──────────┼──────────────┼──────────────────────────┤
│ 13   │ ⑫版本登记 │ 编排器      │ 版本记录                  │
├──────┼──────────┼──────────────┼──────────────────────────┤
│ 14   │ ⑬总结    │ 编排器      │ SUMMARY.md               │
└──────┴──────────┴──────────────┴──────────────────────────┘
```

### 各阶段详细说明

#### 步骤1：初始化（编排器）

- 解析参数（`--lite`, `--timeout`, `--from`, `--to`, `--model`, `--no-superpowers`）
- 创建 `workflow/YYYYMMDD_HHmmss_项目名/` 会话目录
- 写入 `MANIFEST.md`（含模型分配表）
- 初始化 `progress.md`（支持轻量版/完整版，支持 `--from` 回溯标记）
- 初始化 `findings.md`（带格式模板）
- 检查 `task` 工具可用性，不可用时设置 `$FALLBACK=true`
- 写入 `checkpoint.json`（记录项目、会话路径、开始时间、lite/fallback 模式）

#### 步骤2：① 规划 — 交互式需求澄清（编排器亲自完成）

> 不使用 `task` 子Agent，编排器直接在对话中与用户逐轮交互。

**注入方法论：** brainstorming + writing-plans

**流程：**
1. 探索项目上下文（文件结构、README、现有配置）
2. 逐轮提问澄清需求：
   - 每次只问一个问题
   - 优先用选择题
   - 重点了解：业务目的、用户角色、核心功能、约束条件、成功标准
   - 有截图/设计稿时调用 MCP 图片理解分析
   - 边问边记术语到 `CONTEXT.md`
3. 需求明确后提出 2-3 种方案对比，给出推荐
4. 分节展示设计，每节后问用户是否确认
5. 用户确认后编写设计规格到 `docs/superpowers/specs/`
6. 编写实施计划到 `{SESSION_DIR}/01-plan.md`
7. 用户审查并批准计划后才进入下一阶段
8. 更新进度文件 + checkpoint

#### 步骤3：② 领域建模（task → DeepSeek Pro）

**输入：** 设计规格文件 + `01-plan.md`

**注入方法论：** reasonix-domain-modeling

**流程：**
1. 从设计规格中提取核心领域概念
2. 定义实体、值对象、聚合根
3. 建立实体关系（1:1、1:N、N:M）
4. 建立通用语言，更新 `CONTEXT.md`
5. 记录架构决策到 `docs/adr/`（如有技术选型）
6. 输出 `{SESSION_DIR}/domain-model.md`
7. 验证产出文件已生成

**产出：** `{SESSION_DIR}/domain-model.md`

#### 步骤4：③ 架构（task → DeepSeek Pro）

**自动读取：** `01-plan.md` + `domain-models/*.md` + `specs/*-design.md`

**注入方法论：** reasonix-domain-modeling（输入——参考领域模型，确保与业务一致）

**产出：** `{SESSION_DIR}/02-design.md`

> `$FALLBACK` 降级：编排器自行完成架构设计

#### 步骤5：④ 架构评审（编排器注入 reasonix-arch-review）

> **`--lite` 模式跳过此步骤**

**流程：**
1. 模拟 2-3 位领域专家视角
2. 6 维度健康评分（模块化、耦合度、可测试性、演进能力、技术债务、接口设计）
3. 满分 24 分：🟢 ≥18 健康 / 🟡 12-17 一般 / 🔴 <12 严重
4. 🟡 时在编码 prompt 附带待修复项，🔴 时询问用户是否返回重做

**产出：** `{SESSION_DIR}/03-arch-review.md`

> `$FALLBACK` 降级：编排器自行完成评审

#### 步骤6：⑤ 编码（task → DeepSeek Flash）

**自动读取：** `01-plan.md` + `02-design.md` + `03-arch-review.md` + `specs/*-design.md`

**方法论注入（智能选择）：**
- 新功能 → TDD（红→绿→重构）
- 修 Bug → systematic-debugging（定位→分析→修复→验证）
- 改配置 → 不注入
- 涉及 UI → 注入 reasonix-ui-design
- 自动执行产出验证（verification-before-completion）

**产出：** 代码文件 + `{SESSION_DIR}/03-implementation/CHANGES.md`

#### 步骤7：⑥ 测试（task → DeepSeek Flash）

**方法论注入：** dispatching-parallel-agents

**产出：** `{SESSION_DIR}/04-test/TEST_REPORT.md`

> `$FALLBACK` 降级：编排器自行完成测试

#### 步骤8：⑦ 架构扫描（编排器注入 reasonix-arch-review 用法二）

> **`--lite` 模式跳过此步骤**

**流程：**
1. 读取 CONTEXT.md 获取通用语言
2. 分析模块结构和依赖
3. 对每个模块执行"删除测试"
4. 生成 HTML 诊断报告（Mermaid 依赖图）

**产出：** HTML 诊断报告 + findings.md 追加

> `$FALLBACK` 降级：编排器自行完成架构扫描

#### 步骤9：⑧ 审查（task → DeepSeek Pro）

**自动读取：** 规划 + 设计 + 变更 + 测试报告 + findings.md + `specs/*-design.md`

**方法论注入：** requesting-code-review + receiving-code-review + chinese-code-review

**产出：** `{SESSION_DIR}/05-review.md`

> `$FALLBACK` 降级：编排器自行完成审查

#### 步骤10：⑨ 部署（task → DeepSeek Flash）

**自动读取：** 规划 + 设计 + 变更 + 测试报告 + 审查报告 + `specs/*-design.md`

**产出：** `{SESSION_DIR}/06-deploy/DEPLOY.md`

> `$FALLBACK` 降级：编排器自行完成部署方案

#### 步骤11：⑩ 文档质量检查（编排器）

**注入方法论：** chinese-documentation

检查工作流产生的所有文档文件的排版、术语、标点、段落结构是否符合中文技术文档规范，自动修正不合规部分。

**产出：** 文档质量报告

#### 步骤12：⑪ 分支收尾与提交（编排器）

**注入方法论：**
- finishing-a-development-branch（合并/PR 策略）
- chinese-commit-conventions（commit message 格式）
- chinese-git-workflow（国内平台适配）

执行 `git add` + `git commit`，可选推送和 PR。

**产出：** git commit 记录

#### 步骤13：⑫ 版本登记（编排器）

**注入方法论：** vt

记录项目最新路径、功能描述、时间戳。

**产出：** 版本记录

#### 步骤14：⑬ 总结（编排器）

读取各阶段产出摘要，写入 `{SESSION_DIR}/SUMMARY.md`。

**产出：** SUMMARY.md + 最终汇总回复

---

## 5. 上下文传递机制

### 传递链

```
用户原始需求 (在交互式规划中确认)
  │
  ▼
设计规格 specs/*-design.md ──────────────── 所有子Agent交叉验证用
  │
  ├──▶ 领域建模 → domain-model.md ───────── 架构师读取
  │
  ├──▶ 规划 → 01-plan.md ───────────────── 架构/编码/测试/审查/部署读取
  │
  ├──▶ 架构 → 02-design.md ─────────────── 编码/测试/审查/部署读取
  │
  ├──▶ 架构评审 → 03-arch-review.md ────── 编码读取
  │
  ├──▶ 编码 → CHANGES.md ───────────────── 测试/审查/部署读取
  │
  ├──▶ 测试 → TEST_REPORT.md ───────────── 审查/部署读取
  │
  ├──▶ 架构扫描 → findings.md ──────────── 审查读取
  │
  └──▶ 审查 → 05-review.md ─────────────── 部署读取

每个阶段完成后 → 更新 progress.md + findings.md + checkpoint.json
```

### 修复的断裂点

| 问题 | 修复 |
|------|------|
| 领域模型→架构师断裂 | wf-architect 读取 `domain-models/*.md` |
| 架构评审→编码器断裂 | wf-developer 读取 `03-arch-review.md` |
| 架构扫描→审查器断裂 | wf-reviewer 读取 `findings.md` |
| 测试报告→部署器断裂 | wf-deployer 读取 `TEST_REPORT.md` |
| 原始需求缺失 | 所有子Agent读取 `specs/*-design.md` 做交叉验证 |

---

## 6. 进度跟踪与 checkpoint 系统

### 三文件体系

| 文件 | 路径 | 用途 |
|------|------|------|
| `progress.md` | `docs/superpowers/plans/` | 阶段进度可视化（✅/⏳/❌/⏭️） |
| `findings.md` | `docs/superpowers/plans/` | 每个阶段的关键发现和决策记录 |
| `checkpoint.json` | `{SESSION_DIR}/` | 机器可读的断点记录，用于中断恢复 |

### progress.md 完整版模板（13 阶段）

```markdown
| 阶段 | 状态 | 产出 |
|------|------|------|
| 📋 规划 | ⏳ 进行中 | 01-plan.md |
| 🧩 领域建模 | ⏳ 待开始 | 领域模型 |
| 🏗️ 架构 | ⏳ 待开始 | 02-design.md |
| 🏛️ 架构评审 | ⏳ 待开始 | 03-arch-review.md |
| 💻 编码 | ⏳ 待开始 | 代码变更 |
| 🧪 测试 | ⏳ 待开始 | TEST_REPORT.md |
| 🔬 架构扫描 | ⏳ 待开始 | 诊断报告 |
| 🔍 审查 | ⏳ 待开始 | 05-review.md |
| 🚀 部署 | ⏳ 待开始 | DEPLOY.md |
| 📖 文档检查 | ⏳ 待开始 | 文档报告 |
| 🌿 分支收尾 | ⏳ 待开始 | git 提交+PR |
| 📊 版本登记 | ⏳ 待开始 | 版本记录 |
```

### findings.md 格式

```markdown
| 时间 | 阶段 | 发现 | 影响 |
|------|------|------|------|
| 07-09 | 规划 | 用户选择 RBAC 权限模型 | 后续模块需按角色鉴权 |
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

checkpoint 的 `last_completed_phase` 采用 snake_case 命名，如 `planning`, `domain_modeling`, `architecture`, `arch_review`, `coding`, `testing`, `arch_scan`, `review`, `deploy`, `doc_check`, `branch_finish`, `version_tracking`。

### 恢复流程

1. AGENTS.md 在会话启动时检测 `checkpoint.json`
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
| 涉及 UI 设计 | reasonix-ui-design | 编码（按需） |
| 测试执行 | dispatching-parallel-agents | 测试 |
| 代码审查 | requesting-code-review + receiving-code-review + chinese-code-review | 审查 |
| 文档检查 | chinese-documentation | 文档检查 |
| 分支收尾 | finishing-a-development-branch + chinese-commit-conventions + chinese-git-workflow | 分支收尾 |
| 版本登记 | vt | 版本登记 |
| 不确定 | 先问用户"这个需求偏新功能还是修Bug？" | — |

### 注入映射表

| 阶段 | 注入的技能 | 作用 |
|------|-----------|------|
| 📋 规划 (交互) | brainstorming + writing-plans | 需求分析+计划拆解 |
| 🧩 领域建模 | reasonix-domain-modeling | 提取实体关系 |
| 🏗️ 架构 | reasonix-domain-modeling（输入） | 参考领域模型 |
| 🏛️ 架构评审 | reasonix-arch-review | 6维健康评分 |
| 💻 编码 | TDD / systematic-debugging / reasonix-ui-design(按需) | 高质量编码 |
| 🧪 测试 | dispatching-parallel-agents | 并行测试 |
| 🔬 架构扫描 | reasonix-arch-review（用法二） | 代码腐化诊断 |
| 🔍 审查 | requesting-code-review + receiving-code-review + chinese-code-review | 全面审查 |
| 📖 文档检查 | chinese-documentation | 中文规范 |
| 🌿 分支收尾 | finishing-a-development-branch + commit-conventions + git-workflow | 提交规范化 |
| 📊 版本登记 | vt | 版本追踪 |
| 🔄 全阶段可用 | reasonix-handoff | 会话交接 |
| ✅ 产出验证 | verification-before-completion | 自动验证 |

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
完整模式: 规划 → 领域 → 架构 → 评审 → 编码 → 测试 → 扫描 → 审查 → 部署 → 文档检查 → 分支收尾 → 版本登记
lite 模式: 规划 → 领域 → 架构 → [跳过] → 编码 → 测试 → [跳过] → 审查 → 部署 → 文档检查 → 分支收尾 → 版本登记
```

### FALLBACK 降级

当 `task` 工具不可用时自动触发，所有子Agent阶段改为编排器**单Agent顺序执行**。

---

## 9. 错误处理与会话恢复

### 错误场景

| 场景 | 处理方式 |
|------|---------|
| 子Agent 超时 | 记录到 ERRORS.log，询问：重试 / 降级顺序执行 / 跳过 / 中止 |
| 产出验证失败 | 提示缺失文件，提供重试或跳过 |
| 子Agent 调用失败 | 记录错误，从 checkpoint 恢复 |
| 构建失败 | 记录详情，给出修复建议 |
| 某个阶段跳过/失败 | progress.md 中标记为 ⏭️ 已跳过 / ❌ 失败 |

### 会话恢复

1. 检测 `checkpoint.json` → 读取 `last_completed_phase`
2. 检测 `progress.md` → 确认已完成阶段
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
- 💻 编码 — 参考前端截图、处理媒体文件
- 🧪 测试 — 验证 UI 截图、分析测试日志
- 🔍 审查 — 审查 UI 实现与设计稿差异

---

## 11. 技能体系

共 36 个技能：

| 分组 | 技能 | 融入流程 |
|------|------|:--------:|
| **核心工作流** | wf-orchestrator, wf-architect, wf-developer, wf-tester, wf-reviewer, wf-deployer | ✅ |
| **领域建模** | reasonix-domain-modeling | ✅ |
| **架构评审/扫描** | reasonix-arch-review（两种用法） | ✅ |
| **方法论注入** | test-driven-development, systematic-debugging, dispatching-parallel-agents, requesting-code-review, receiving-code-review, chinese-code-review | ✅ |
| **规划与支撑** | brainstorming, writing-plans, executing-plans, reasonix-handoff | ✅ |
| **文档与提交** | chinese-documentation, finishing-a-development-branch, chinese-commit-conventions, chinese-git-workflow | ✅ |
| **版本与验证** | vt, verification-before-completion | ✅ |
| **UI 设计** | reasonix-ui-design | ✅（按需） |
| **多模态** | mimo-image-understanding, mimo-audio-understanding, mimo-video-understanding, mimo-tts | ✅（MCP 可用） |
| **独立技能** | workflow-runner, subagent-driven-development, chinese-documentation(独立), trigger-commands, using-git-worktrees | ⏳ 手动 |
| **无法工作** | baidu-unlimited-ocr | ❌ MCP 未就绪 |

---

## 12. 配置说明

### 环境变量

在 `${REASONIX_HOME}\.env` 中配置：

```env
DEEPSEEK_API_KEY=sk-xxx
MIMO_API_KEY=sk-xxx
BAIDU_OCR_API_KEY=bce-v3/xxx
```

### reasonix.toml

在 `${WORKSPACE}\reasonix.toml` 中配置：

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
${WORKSPACE}\
├── .reasonix\skills\         ← 36 个技能
├── reasonix.toml             ← MCP + sandbox 配置
├── AGENTS.md                 ← 进度检查兜底规则
├── WORKFLOW_MANUAL.md        ← 本文档
├── docs\
│   ├── superpowers\
│   │   ├── plans\            ← progress.md, findings.md
│   │   ├── specs\            ← 设计规格
│   │   └── domain-models\    ← 领域模型
│   └── adr\                  ← 架构决策记录
└── workflow\                 ← 会话产出目录
    └── YYYYMMDD_HHmmss_项目名\
        ├── MANIFEST.md
        ├── 01-plan.md
        ├── domain-model.md
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

---

## 独立工具：YAML 工作流执行器（workflow-runner）

除了预制的 13 阶段开发流水线（wf-orchestrator），本工作流还包含一个通用 YAML 工作流执行器。

### 定位

`workflow-runner` 是一个**通用引擎**，可以读取任意 `.yaml` 文件定义的多角色协作工作流，在当前会话中依次扮演每个角色完成任务。无需额外 API key，当前 LLM 就是执行引擎。

### 使用场景

- 用户提供了一个 `.yaml` 工作流文件
- 需要多角色协作完成非开发任务（如 PRD 评审、方案评估、头脑风暴）
- 想要自定义工作流但不修改 wf-orchestrator

### 与 wf-orchestrator 的关系

| | wf-orchestrator | workflow-runner |
|:--|:---------------|:----------------|
| 定位 | 软件开发专用流水线 | 通用多角色工作流引擎 |
| 输入 | `/wf-orchestrator 项目名: 需求` | `/workflow-runner 工作流.yaml` |
| 角色 | 固定 7 个子 Agent | YAML 定义的任意角色 |
| 适用 | 编码/测试/部署等开发任务 | 评审/创作/分析等任意协作任务 |

详见 `skills/workflow-runner/SKILL.md`。

---

## 百度 OCR 双模式调用

百度 Unlimited-OCR 支持两种调用方式，按工具能力自动选择：

| 模式 | 传输方式 | 适用工具 | 配置 |
|:-----|:--------|:---------|:-----|
| **MCP SSE** | Server-Sent Events | Reasonix、ZCode | `type = "http"` 插件 |
| **REST API** | HTTP POST + Base64 | AutoClaw、Cursor 等 | `BAIDU_OCR_API_KEY` + `BAIDU_OCR_SECRET_KEY` |

调用时自动检测 MCP SSE 能力 → 有则用 SSE，无则 fallback REST API。
详细调用示例见 `skills/baidu-unlimited-ocr/SKILL.md`。
