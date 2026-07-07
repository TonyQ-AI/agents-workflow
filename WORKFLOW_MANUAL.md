# Reasonix Workflow 完整技术说明书（v2.0）

> **多 Agent 协同开发工作流** — 8 阶段全流程自动化，27 个技能，2 套多模态 MCP（MiMo + 百度 OCR），6 层防幻觉机制
>
> 仓库：https://github.com/TonyQ-AI/reasonix-workflow

---

## 目录

1. [概述](#1-概述)
2. [快速上手指南](#2-快速上手指南)
3. [系统架构](#3-系统架构)
4. [8 阶段工作流详解](#4-8-阶段工作流详解)
5. [模型分配与资源规划](#5-模型分配与资源规划)
6. [多模态 MCP 集成详解](#6-多模态-mcp-集成详解)
7. [防幻觉机制（6 层防线）](#7-防幻觉机制6-层防线)
8. [中断恢复与会话交接](#8-中断恢复与会话交接)
9. [知识沉淀体系](#9-知识沉淀体系)
10. [文件结构与产出物规范](#10-文件结构与产出物规范)
11. [错误处理与异常流程](#11-错误处理与异常流程)
12. [安装与配置](#12-安装与配置)
13. [使用场景与最佳实践](#13-使用场景与最佳实践)
14. [附录：技能完整清单](#14-附录技能完整清单)

---

## 1. 概述

### 1.1 什么是 Reasonix Workflow？

Reasonix Workflow 是一套基于 AI Agent 的全流程自动化软件开发流水线。它通过 **8 个专业化 Agent** 分工协作，实现软件开发从需求分析到部署方案的全流程自动化。

**核心理念：**
- **分工专业化** — 每个 Agent 只做自己最擅长的事（默认规划用 Pro 模型深度推理，编码用 Flash 模型高速实现，可自行配置其他模型）
- **方法论驱动** — 每个阶段注入对应的软件工程方法论（TDD、架构评审、系统化调试等）
- **防幻觉优先** — 6 层机制防止 AI 在长上下文中产生幻觉
- **可中断可恢复** — 任意时刻中断，任意会话续做

### 1.2 适用场景

| 场景 | 推荐方式 | 说明 |
|:----|:---------|:------|
| 🆕 新功能开发 | 全流程 | `/wf-orchestrator 项目: 需求` |
| 🐛 Bug 修复 | 局部执行 | `--from dev --to review` |
| 🏗️ 系统重构 | 从规划开始 | 先架构扫描再做计划 |
| 🖼️ 含 UI 设计 | 全流程 + 多模态 | MCP 自动分析设计稿 |
| 📝 纯文档/计划 | 单 Agent | `/wf-planner ...` |
| 🎙️ 媒体处理 | 单技能 | `/mimo-audio-understanding ...` |

---

## 2. 快速上手指南

### 2.1 一键启动完整工作流示例

```
/wf-orchestrator MediaManager: 新增视频批量导出功能，支持按标签筛选导出为Excel
```

工作流会自动：
1. 创建 `workflow/<时间戳_MediaManager>/` 会话目录
2. 运行规划 Agent → 输出 `01-plan.md`
3. 运行领域建模 → 输出领域模型
4. 运行架构 Agent → 输出 `02-design.md`
5. 运行架构评审 → 输出 `03-arch-review.md` + 健康评分
6. 运行编码 Agent → 产出代码
7. 运行测试 Agent → 输出 `TEST_REPORT.md`
8. 运行架构扫描 → 输出 HTML 诊断报告
9. 运行审查 Agent → 输出 `05-review.md`
10. 运行部署 Agent → 输出 `DEPLOY.md`
11. 生成会话总结 `SUMMARY.md`

### 2.2 局部执行

从编码阶段开始，到审查阶段结束示例：
```
/wf-orchestrator MediaManager: 修复登录页面样式问题 --from dev --to review
```

### 2.3 覆盖模型

全部 Agent 使用同一模型（不推荐，仅测试用）示例：
```
/wf-orchestrator MediaManager: 重构用户模块 --model deepseek
```

### 2.4 禁用方法论注入示例

跳过所有 superpowers 方法论指引（不推荐）：
```
/wf-orchestrator MediaManager: 简单配置修改 --no-superpowers
```

### 2.5 直接调用单个 Agent示例

```
/wf-planner 设计一个用户权限管理模块
/wf-developer 实现上述计划中导出功能
/mimo-image-understanding 分析这个设计稿：C:/design.png
```

---

## 3. 系统架构

### 3.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         用户输入                                       │
│   /wf-orchestrator <项目名>: <需求描述> [--from X] [--to Y] [--model M] │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     wf-orchestrator（主编排器）                          │
│                                                                         │
│  ① 解析参数 → ② 创建会话目录 → ③ 写入 MANIFEST → ④ 串联各阶段          │
│     │                                                                   │
│     │  每个阶段：                                                       │
│     │  1. 输出阶段信息（Agent → 模型）                                   │
│     │  2. 注入方法论指引 + 进度文件到 prompt                              │
│     │  3. 用 task 工具派发子 Agent                                       │
│     │  4. 验证产出文件存在                                               │
│     │  5. 记录到会话目录                                                 │
│     │                                                                   │
└─────────────────────────────────────────────────────────────────────────┘
         │
         │   8 个阶段由编排器自动串联
         ▼
┌─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐
│ 📋 规划  │ 🧩 领域  │ 🏗️ 架构  │ 🏛️ 架构   │ 💻 编码  │ 🧪 测试  │ 🔬 架构   │ 🔍 审查  │
│         │ 建模     │ 设计    │ 评审     │         │         │ 扫描    │         │
│ Pro 🧠  │ Pro 🧠  │ Pro 🧠  │ Pro 🧠   │Flash ⚡ │Flash ⚡ │ Pro 🧠  │ Pro 🧠  │
│         │         │         │         │         │         │         │         │
│ 01-plan │ 领域模型 │ 02-design│03-arch- │ 代码变更 │ TEST_   │ HTML    │ 05-review│
│ .md     │         │ .md     │ review  │         │ REPORT  │ 诊断报告 │ .md     │
│         │         │         │ .md     │         │ .md     │         │         │
└─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      🚀 部署方案                                         │
│                      DEPLOY.md + 构建配置                                │
└─────────────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      📊 会话总结 SUMMARY.md                               │
└─────────────────────────────────────────────────────────────────────────┘

         ┌──────────────────────────────────────────────────────────┐
         │             多模态 MCP 层（可选）                           │
         │                                                          │
         │  ┌──────────────────┐  ┌──────────────────┐              │
         │  │  MiMo API        │  │  MCP Server       │              │
         │  │  (xiaomimimo)    │◄─│  (dist/index.js)  │◄── MiMo 技能  │
         │  │                  │  └──────────────────┘              │
         │  │  百度智能云 API   │  ┌──────────────────┐              │
         │  │  (baidubce)      │◄─│  HTTP SSE         │◄── 百度 OCR  │
         │  └──────────────────┘  └──────────────────┘              │
         │                                                          │
         └──────────────────────────────────────────────────────────┘
```

### 3.2 技能体系结构

26 个技能按用途分为 7 组：

```
reasonix-workflow/
│
├── 核心工作流（7个）       ← 8 阶段流水线的 Agent 技能
│   ├── wf-orchestrator     ← 主编排器（入口点）
│   ├── wf-planner          ← 规划 Agent
│   ├── wf-architect        ← 架构 Agent
│   ├── wf-developer        ← 编码 Agent
│   ├── wf-tester            ← 测试 Agent
│   ├── wf-reviewer         ← 审查 Agent
│   └── wf-deployer         ← 部署 Agent
│
├── 架构评审与扫描（1个技能，2种用法）
│   └── reasonix-arch-review
│       ├── 用法一：设计评审（编码前）——专家模拟 + 规则评分
│       └── 用法二：架构扫描（编码后）——删除测试 + 依赖图 + HTML 报告
│
├── 方法论注入（5个）      ← 注入到对应阶段的工程方法论
│   ├── test-driven-development       → 编码阶段
│   ├── systematic-debugging          → 编码阶段（修 Bug）
│   ├── dispatching-parallel-agents   → 测试阶段
│   ├── requesting-code-review        → 审查阶段
│   └── chinese-code-review           → 审查阶段
│
├── 规划与支撑（5个）      ← 辅助规划、执行、交接
│   ├── brainstorming          ← 需求分析、方案对比
│   ├── writing-plans          ← 计划生成 + 三文件进度跟踪
│   ├── executing-plans        ← 自动检测进度、中断续做
│   ├── reasonix-domain-modeling ← 领域建模
│   └── reasonix-handoff       ← 会话交接
│
├── 扩展能力（5个）        ← 扩展开发周期各环节
│   ├── reasonix-ui-design              ← 专业 UI 设计
│   ├── workflow-runner                 ← YAML 工作流执行器
│   ├── finishing-a-development-branch  ← 分支集成规范
│   ├── verification-before-completion  ← 完成前自动验证
│   └── subagent-driven-development     ← 子代理驱动开发
│
├── MiMo 多模态（4个）     ← 小米 MiMo 多模态能力
│   ├── mimo-image-understanding  → understand_image
│   ├── mimo-audio-understanding  → understand_audio
│   ├── mimo-video-understanding  → understand_video
│   └── mimo-tts                  → tts
│
└── 百度 OCR 文档解析（1个）← 百度 Unlimited-OCR
    └── baidu-unlimited-ocr        → ocr_document
```

### 3.3 编排器工作机制（wf-orchestrator 内部流程）

编排器执行每个阶段的统一流程：

```
┌─────────────────────────────────────────────────────────────┐
│                   每个阶段的执行流程                          │
│                                                             │
│  1. 输出阶段标识：                                           │
│     🟢 阶段 N/8：阶段名 - Agent名 → 模型名                   │
│                                                             │
│  2. 构造子 Agent prompt：                                    │
│     a. 如果是规划/编码/测试/审查阶段                          │
│        → 前置插入对应方法论指引块（===== 🎯 =====）           │
│     b. 如果进度文件存在                                       │
│        → 插入进度跟踪块（===== 📋 进度跟踪 =====）            │
│     c. 如果涉及媒体文件                                       │
│        → 插入 MCP 可用性块（===== 🎯 多模态 MCP =====）       │
│     d. 追加原始任务描述                                       │
│                                                             │
│  3. 用 task 工具启动子 Agent：                                │
│     - 模型：默认分配（Pro/Flash）或 --model 覆盖值             │
│     - max_steps：编码阶段放宽，其他阶段适中                    │
│                                                             │
│  4. 等待子 Agent 完成                                        │
│                                                             │
│  5. 验证产出文件：                                           │
│     - 规划阶段 → 检查 01-plan.md 是否存在                     │
│     - 架构阶段 → 检查 02-design.md 是否存在                   │
│     - 架构评审 → 检查 03-arch-review.md 是否存在              │
│     - 编码阶段 → 检查 03-implementation/ 是否有产出           │
│     - 测试阶段 → 检查 TEST_REPORT.md 是否存在                 │
│     - 审查阶段 → 检查 05-review.md 是否存在                   │
│     - 部署阶段 → 检查 DEPLOY.md 是否存在                      │
│                                                             │
│  6. 产出缺失 → 提示用户，提供重试/跳过选项                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. 8 阶段工作流详解

### 阶段① 📋 规划（wf-planner → DeepSeek Pro 🧠）

**目标：** 将模糊的需求转化为清晰的设计规格和可执行的实施计划。

**注入的方法论（2个）：**

| 方法论 | 作用 |
|:------|:-----|
| **brainstorming** | 先分析需求，识别隐含假设和模糊点，输出设计规格，给用户 2-3 个可选方案对比 |
| **writing-plans** | 将需求拆解为具体实施步骤，每步含任务描述、验收标准、产出物，识别可并行任务 |

**详细流程：**

```
① 规划子 Agent 启动
  │
  ├── 读取项目上下文（文件、文档、最近的 commit）
  │
  ├── [brainstorming 注入]
  │   ├── 分析需求，不要直接写方案
  │   ├── 识别隐含假设和模糊点，列出开放问题
  │   ├── 每次只问一个问题（选择题优先）
  │   ├── 提出 2-3 种方案 + 权衡分析
  │   ├── 展示设计，每节获批后再继续
  │   └── 边问边记术语到 CONTEXT.md
  │
  ├── 编写设计规格文档
  │   └── docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md
  │
  ├── 规格自检
  │   ├── 占位符扫描（TODO/待定/后续实现）
  │   ├── 内部一致性检查（各章节无矛盾）
  │   ├── 范围检查（是否可被一个计划覆盖）
  │   └── 模糊性检查（是否有歧义）
  │
  ├── 用户审查关卡 → 等待用户批准
  │
  └── [writing-plans 注入]
      ├── 将设计拆解为小步骤任务（每步 2-5 分钟）
      ├── 每个任务含：
      │   ├── 文件路径（创建/修改/测试）
      │   ├── 完整代码块（禁止占位符）
      │   ├── 验证命令 + 预期输出
      │   └── Commit 命令
      ├── 生成计划文档
      │   └── docs/superpowers/plans/YYYY-MM-DD-<feature>.md
      └── 生成三个进度跟踪文件
          ├── task_plan.md  ← 可勾选任务清单
          ├── findings.md   ← 发现记录
          └── progress.md   ← 进度摘要
```

**产出物：**
- `{SESSION_DIR}/01-plan.md` — 规划说明和需求文档
- `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` — 设计规格
- `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` — 实施计划
- `docs/superpowers/plans/task_plan.md` — 可勾选任务清单
- `docs/superpowers/plans/findings.md` — 发现记录
- `docs/superpowers/plans/progress.md` — 进度摘要

---

### 阶段② 🧩 领域建模（reasonix-domain-modeling → DeepSeek Pro 🧠）

**目标：** 在动手架构设计之前，把问题域的核心概念、实体关系、业务规则理清楚。

**核心理念：** 代码是领域模型的映射。模型不清楚，架构就不可能清楚。

**详细流程：**

```
① 提取核心概念
   通读需求/规格，识别所有业务概念（非技术名词）
   示例：
   - 用户 → 系统的注册使用者
   - 播放列表 → 视频的有序集合
   - 视频 → 可播放的媒体内容

② 定义实体和关系
   用文本描述实体之间的关系：
   用户 1---* 播放列表  (一个用户可有多个播放列表)
   播放列表 *---* 视频  (多对多关系)

③ 建立通用语言
   | 术语 | 定义 | 别称（禁用） |
   |------|------|-------------|
   | 用户 | 注册使用者 | 客户、会员 |
   | 共享 | 设为其他用户可见 | 分享、公开 |

④ 识别业务规则
   1. 非公开播放列表对其他用户不可见
   2. 一个播放列表最多包含 500 个视频

⑤ 识别限界上下文
   将大系统拆成有清晰边界的子域：
   ┌─ 用户上下文 ──┐  ┌─ 内容上下文 ──┐
   │ 注册/登录/好友 │  │ 上传/转码/存储│
   └──────────────┘  └──────────────┘

⑥ 实时知识沉淀
   - 通用语言 → 实时写入 CONTEXT.md
   - 关键决策 → 实时写 ADR（架构决策记录）
```

**产出物：**
- `docs/superpowers/domain-models/YYYY-MM-DD-<feature>-domain.md` — 领域模型文档
- CONTEXT.md 追加通用语言
- `docs/adr/` 追加架构决策记录

---

### 阶段③ 🏗️ 架构设计（wf-architect → DeepSeek Pro 🧠）

**目标：** 基于领域模型做系统架构设计，定义模块边界、接口规范和技术选型。

**输入：** 领域模型输出 + 规划阶段的规格文档

**详细流程：**

```
① 读取领域模型
   基于领域建模输出，理解核心实体、关系和业务规则

② 系统架构设计
   - 分层结构（展示层/业务层/数据层）
   - 模块划分与职责
   - 模块间接口定义
   - 数据流设计
   - 错误处理策略

③ 技术选型
   - 框架/库选择 + 理由
   - 数据库设计（ER 图）
   - API 设计（RESTful / GraphQL 等）

④ 输出架构设计文档
   {SESSION_DIR}/02-design.md
```

**产出物：**
- `{SESSION_DIR}/02-design.md` — 架构设计文档

---

### 阶段④ 🏛️ 架构评审（reasonix-arch-review → DeepSeek Pro 🧠）

**目标：** 在编码之前审查架构设计，使用专家模拟 + 规则检测，给出量化健康评分和 Go/No-Go 决策。

**这是工作流中最重要的防幻觉关卡之一。**

**两阶段评审法：**

#### 阶段一：专家视角（发散）

模拟 2-3 位业界顶尖架构师，从各自擅长的视角审视设计：

| 专家 | 擅长视角 | 关注点 |
|:----|:---------|:-------|
| **Martin Fowler** | 重构、演进式架构 | 是否过度设计、代码异味 |
| **Rich Hickey** | 简单性、复杂度管理 | 是否过度工程化、不必要的抽象 |
| **Linus Torvalds** | 工程实现、实用主义 | 接口是否合理、能不能跑 |
| **Sam Newman** | 微服务、系统边界 | 服务划分、耦合度 |
| **John Ousterhout** | 深度模块、信息隐藏 | 模块边界、接口简洁度 |

#### 阶段二：规则检测（收敛）

6 维度 24 条规则逐条检查：

| 维度 | 检查项（示例） | 分值 |
|:----|:-------------|:----:|
| ① 模块化 | 模块有单一职责？依赖反转？接口通信？ | /4 |
| ② 耦合度 | 扇入/扇出合理？无循环依赖？外部依赖隔离？ | /4 |
| ③ 可测试性 | 核心逻辑可脱离依赖测试？有清晰测试边界？ | /4 |
| ④ 演进能力 | 新增功能改几个文件？支持部分替换？ | /4 |
| ⑤ 技术债务 | 无 TODO/FIXME？无重复代码？测试覆盖合理？ | /4 |
| ⑥ 接口设计 | API 自文档化？错误处理统一？输入输出最小化？ | /4 |

#### Go/No-Go 决策

| 总分 | 判定 | 处理 |
|:---:|:----:|:-----|
| 🟢 **≥18** | 架构健康 | 直接进入开发阶段 |
| 🟡 **12-17** | 存在一般问题 | 修复后再开发，问题项追加到开发 prompt |
| 🔴 **<12** | 架构存在严重问题 | **暂停**，向用户报告，询问是否返回架构阶段重做 |

**产出物：**
- `{SESSION_DIR}/03-arch-review.md` — 架构评审报告（含评分、专家意见、依赖图）
- 发现的架构问题追加到 `findings.md`

---

### 阶段⑤ 💻 编码（wf-developer → DeepSeek Flash ⚡）

**目标：** 按照实施计划编写代码，遵循 TDD 方法论。

**注入的方法论（条件判断）：**

| 场景 | 方法论 |
|:----|:-------|
| 新功能开发 | **test-driven-development**（红→绿→重构） |
| Bug 修复 | **systematic-debugging**（定位→分析→假设→修复→验证） |

#### TDD 流程（新功能）

```
┌─────────┐    ┌──────────┐    ┌─────────┐    ┌──────────┐    ┌────────┐
│ 红灯     │───►│ 验证失败  │───►│ 绿灯     │───►│ 验证通过  │───►│ 重构    │
│ 写失败   │    │ 确认正确  │    │ 最少代码  │    │ 确认通过  │    │ 清理    │
│ 的测试   │    │ 失败原因  │    │ 让测试   │    │ 且不破坏  │    │ 代码    │
│         │    │          │    │ 通过     │    │ 其他测试  │    │        │
└─────────┘    └──────────┘    └─────────┘    └──────────┘    └────────┘
     │              │              │              │              │
     └──────────────┴──────────────┴──────────────┴──────────────┘
                        TDD 循环，持续迭代
```

**铁律：**
- 没有失败的测试，就不写生产代码
- 测试必须看到失败（证明测试有效）
- 先写了代码再写测试？删掉重来
- 后补测试 = "这做了什么？" 先写测试 = "这应该做什么？"

#### 系统化调试流程（Bug 修复）

```
① 定位 → 确认 bug 发生的具体位置和复现条件
② 分析 → 找出根因（不是表象）
③ 假设 → 提出修复方案并评估影响
④ 修复 + 验证 → 执行修复并确认 bug 不再复现
```

**产出物：**
- `{SESSION_DIR}/03-implementation/` — 代码变更目录
- `CHANGES.md` — 变更说明

---

### 阶段⑥ 🧪 测试（wf-tester → DeepSeek Flash ⚡）

**目标：** 设计并执行测试用例，生成测试报告。

**注入的方法论：**

| 方法论 | 作用 |
|:------|:-----|
| **dispatching-parallel-agents** | 识别独立测试，并行执行 |

**并行测试策略：**

```
① 分析测试用例
   识别可独立执行的测试套件

② 并行分派
   Task("修复 test_A 失败")
   Task("修复 test_B 失败")  ← 并发运行
   Task("修复 test_C 失败")

③ 收集结果
   汇总各并行任务的测试报告
```

**产出物：**
- `{SESSION_DIR}/04-test/TEST_REPORT.md` — 测试报告（覆盖率、通过率、失败详情）

---

### 阶段⑦ 🔬 架构扫描（reasonix-arch-review 用法二 → DeepSeek Pro 🧠）

**目标：** 在编码完成后扫描真实代码，诊断架构腐化程度，输出 HTML 诊断报告。

**与阶段④的区别：**
- 阶段④ **设计评审** — 评审的是**纸面上的架构设计**，在编码前
- 阶段⑦ **架构扫描** — 扫描的是**真实的代码实现**，在编码后

**扫描流程：**

```
① 读取领域模型
   - 从 CONTEXT.md 获取通用语言
   - 从 ADR 获取历史决策

② 扫描代码结构
   - 遍历关键目录
   - 识别模块边界和文件组织
   - 分析模块间依赖关系

③ 执行删除测试
   删掉一个模块，复杂度是集中了还是转移了？
   - 集中 → 模块划分好
   - 转移 → 模块太浅

④ 生成 HTML 报告
   自包含 HTML（无外部依赖），包含：
   - 模块深度评估卡
   - Mermaid 依赖图
   - 删除测试结果
   - 候选改进列表（按推荐强度排序）
```

**产出物：**
- 自包含 HTML 架构诊断报告
- 扫描结果追加到 `findings.md`
- 关键决策写到 `docs/adr/`

---

### 阶段⑧ 🔍 审查（wf-reviewer → DeepSeek Pro 🧠）

**目标：** 逐文件检查代码质量，给出建设性审查意见。

**注入的方法论（2个）：**

| 方法论 | 作用 |
|:------|:-----|
| **requesting-code-review** | 逐文件标注问题位置（文件:行号），按严重程度分类 |
| **chinese-code-review** | 建设性中文反馈，关注命名/注释/可读性，给出修改示例 |

**审查分类标准：**

| 级别 | 说明 | 处理方式 |
|:----|:-----|:---------|
| 🚨 **严重** | 功能 bug、安全漏洞、性能问题 | 必须修复 |
| ⚠️ **一般** | 代码异味、设计不合理、测试缺失 | 建议修复 |
| 💡 **建议** | 可读性改进、命名建议 | 可选项 |

**产出物：**
- `{SESSION_DIR}/05-review.md` — 审查报告（逐文件、逐问题标注）

---

### 阶段⑨ 🚀 部署（wf-deployer → DeepSeek Flash ⚡）

**目标：** 生成构建方案和部署计划。

**内容：**
- 构建命令和脚本
- 环境配置（开发/测试/生产）
- 版本号建议
- 回滚方案

**产出物：**
- `{SESSION_DIR}/06-deploy/DEPLOY.md` — 部署方案文档

---

### 阶段⑩ 📊 会话总结

编排器在最后生成完整的会话总结 `SUMMARY.md`：

```markdown
# 工作流会话总结

## 会话信息
- 项目：{项目名}
- 需求：{需求描述}
- 完成时间：{时间戳}
- 总耗时：{估算}

## 各阶段摘要
| 阶段 | 模型 | 核心产出 |
|-----|------|---------|
| 📋 规划 | Pro 🧠 | 01-plan.md |
| 🏗️ 架构 | Pro 🧠 | 02-design.md |
| 🏛️ 架构评审 | Pro 🧠 | 03-arch-review.md（评分: X/24） |
| 💻 编码 | Flash ⚡ | 代码变更（新增 N 文件，修改 N 文件） |
| 🧪 测试 | Flash ⚡ | TEST_REPORT.md（通过率 X%） |
| 🔬 架构扫描 | Pro 🧠 | 代码架构诊断报告 |
| 🔍 审查 | Pro 🧠 | 05-review.md（总体评价：通过） |
| 🚀 部署 | Flash ⚡ | DEPLOY.md（构建状态：成功） |
```

---

## 5. 模型分配与资源规划

### 5.1 默认模型分配表

| Agent | 阶段 | 默认模型 | 角色 | 单次调用成本 |
|-------|------|---------|:----:|:------------:|
| wf-planner | 规划 | DeepSeek Pro 🧠 | 深度推理 | ¥3+¥6/百万token |
| reasonix-domain-modeling | 领域建模 | DeepSeek Pro 🧠 | 深度推理 | ¥3+¥6/百万token |
| wf-architect | 架构 | DeepSeek Pro 🧠 | 深度推理 | ¥3+¥6/百万token |
| reasonix-arch-review | 评审/扫描 | DeepSeek Pro 🧠 | 深度推理 | ¥3+¥6/百万token |
| wf-developer | 编码 | DeepSeek Flash ⚡ | 高效实现 | ¥1+¥2/百万token |
| wf-tester | 测试 | DeepSeek Flash ⚡ | 分析执行 | ¥1+¥2/百万token |
| wf-reviewer | 审查 | DeepSeek Pro 🧠 | 深度推理 | ¥3+¥6/百万token |
| wf-deployer | 部署 | DeepSeek Flash ⚡ | 流程执行 | ¥1+¥2/百万token |

### 5.2 模型选择逻辑

```
默认策略：Pro 用于规划和推理密集型任务，Flash 用于实现和执行密集型任务
    │
    ├── 用户指定 --model deepseek → 全部使用 DeepSeek（Pro/Flash 按角色分配）
    ├── 用户指定 --model 小米mimo → 全部使用 MiMo
    └── 未指定 → 按上表默认分配
```

### 5.3 Token 消耗估算

| 完整工作流 | 约 50 万-200 万 token（视项目复杂度） |
|:----------|:--------------------------------------|
| 规划阶段 | 5 万-20 万 token |
| 架构阶段 | 5 万-15 万 token |
| 编码阶段 | 20 万-100 万 token |
| 测试阶段 | 10 万-30 万 token |
| 审查阶段 | 5 万-20 万 token |
| 部署阶段 | 1 万-5 万 token |

---

## 6. 多模态 MCP 集成详解

### 6.1 架构

```
┌─────────────────────────────────────────────────────────────┐
│                  多模态调用链路                               │
│                                                             │
│  Agent 检测到媒体文件引用（本地路径/URL）                      │
│         │                                                    │
│         ▼                                                    │
│  Skill SKILL.md 检测文件类型，选择分析模式                     │
│         │                                                    │
│         ▼                                                    │
│  MCP 工具调用                                                │
│  mcp__mimo-multimodal__<tool_name>(source, prompt, ...)      │
│         │                                                    │
│         ▼                                                    │
│  MCP Server (stdio)                                          │
│  读取文件 → Base64编码 → 调用 MiMo API                        │
│         │                                                    │
│         ▼                                                    │
│  小米 MiMo API                                               │
│  https://api.xiaomimimo.com/v1                               │
│  (api-key 认证)                                              │
│         │                                                    │
│         ▼                                                    │
│  AI Agent 收到结构化结果                                      │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 MCP 工具详细规格

#### understand_image — 图片理解

| 属性 | 值 |
|:----|:----|
| 工具名 | `understand_image` |
| 对应技能 | `mimo-image-understanding` |
| 支持格式 | JPEG, PNG, GIF, WebP, BMP |
| 文件上限 | 50MB |
| 输入类型 | 本地路径 / URL / Base64 / Data URI / Claude cache ref |
| 后端模型 | mimo-v2.5（主）/ mimo-v2-omni（备） |
| 调用方式 | Anthropic 兼容端点（`/anthropic/v1/messages`） |
| Thinking 模式 | 已禁用（`thinking: { type: "disabled" }`） |

**分析模式：**

| 模式 | prompt 模板 | 典型场景 |
|:----|:-----------|:---------|
| **describe** | "Provide a detailed description of this image..." | 通用图片描述 |
| **ocr** | "Extract all text visible in this image verbatim..." | 文字识别 |
| **ui-review** | "You are a UI/UX design reviewer..." | UI 设计评审 |
| **chart-data** | "Extract all data from this chart or graph..." | 图表数据提取 |
| **object-detect** | "List all distinct objects, people, and activities..." | 物体检测 |
| **web-debug** | "You are a frontend debugging assistant..." | 前端截图调试 |

#### understand_audio — 音频理解

| 属性 | 值 |
|:----|:----|
| 工具名 | `understand_audio` |
| 对应技能 | `mimo-audio-understanding` |
| 支持格式 | MP3, WAV, FLAC, M4A, OGG |
| URL 上限 | 100MB |
| Base64 上限 | 50MB |
| Token 估算 | `tokens ≈ 秒数 × 6.25` |
| 后端模型 | mimo-v2.5（主）/ mimo-v2-omni（备） |
| 调用方式 | OpenAI 兼容端点 |

**时长与可靠性建议：**

| 时长 | Token | 可靠性 | 建议 |
|:---:|:-----:|:------:|:-----|
| < 2min | < 750 | 🟢 高 | 直接处理 |
| 2-5min | 750-1,875 | 🟡 良好 | 直接处理 |
| 5-10min | 1,875-3,750 | 🟠 中等 | 建议分片 |
| 10-20min | 3,750-7,500 | 🔴 低 | 必须分片 |
| > 20min | > 7,500 | ⚫ 不可靠 | 分片+压缩 |

**音频分段处理命令：**
```bash
# 压缩（仅语音）：单声道、16kHz、64kbps
ffmpeg -i input.m4a -ar 16000 -ac 1 -b:a 64k input_compressed.mp3

# 分割为 2 分钟片段
ffmpeg -i input_compressed.mp3 -f segment -segment_time 120 -c copy chunk_%02d.mp3
```

#### understand_video — 视频理解

| 属性 | 值 |
|:----|:----|
| 工具名 | `understand_video` |
| 对应技能 | `mimo-video-understanding` |
| 支持格式 | MP4, MOV, AVI, WMV |
| URL 上限 | 300MB |
| Base64 上限 | 50MB |
| 帧率范围 | 0.1-10 fps（默认 2） |
| 分辨率 | `default`（平衡） / `max`（高细节） |
| 后端模型 | mimo-v2.5（主）/ mimo-v2-omni（备） |

**帧率选择指南：**

| 内容类型 | 推荐 fps | 说明 |
|:--------|:--------:|:-----|
| 讲座/演讲 | 0.1-0.5 | 场景变化少，低帧率足够 |
| 通用 | 1-2 | 大多数场景的平衡选择 |
| 教程/操作 | 3-5 | 需要捕捉操作细节 |
| 运动/动作 | 5-10 | 快速变化场景需要高帧率 |

#### tts — 语音合成

| 属性 | 值 |
|:----|:----|
| 工具名 | `tts` |
| 对应技能 | `mimo-tts` |
| 输出格式 | WAV（完整文件）/ PCM16（流式） |
| 后端模型 | mimo-v2.5-tts / mimo-v2.5-tts-voicedesign / mimo-v2.5-tts-voiceclone |

**预设音色：**

| 音色 ID | 语言 | 性别 |
|:--------|:----|:----:|
| `mimo_default` | 自动 | — |
| `冰糖` | 中文 | 女 |
| `茉莉` | 中文 | 女 |
| `苏打` | 中文 | 男 |
| `白桦` | 中文 | 男 |
| `Mia` | 英文 | 女 |
| `Chloe` | 英文 | 女 |
| `Milo` | 英文 | 男 |
| `Dean` | 英文 | 男 |

### 6.3 编排器 MCP 注入机制

当编排器检测到子 Agent 任务涉及媒体文件时（用户提供了截图/音频/视频路径），会在 prompt 中自动注入：

```
===== 🎯 多模态 MCP 可用 =====
本会话已配置 mimo-multimodal MCP 服务器，支持：
- 图片理解（本地路径 / URL → describe / ocr / ui-review / chart-data / object-detect / web-debug）
- 音频理解（本地路径 / URL → transcribe / describe / summarize）
- 视频理解（本地路径 / URL → describe / summarize / action-detect）
- TTS 语音合成（文本→语音，支持多音色）
使用方法：调用 mcp__mimo-multimodal__<tool_name> 工具
环境要求：需设置 MIMO_API_KEY 环境变量
=====
```

### 6.4 各阶段多模态调用场景详解

| 阶段 | 用户提供的媒体 | MCP 调用 | 注入时机 |
|:----|:--------------|:---------|:---------|
| 📋 规划 | 需求截图/竞品截图/UI 设计稿 | `understand_image`（ocr/ui-review） | 需求分析时自动检测 |
| 💻 编码 | 前端参考截图/设计稿 | `understand_image`（web-debug/describe） | 编码子 Agent 启动时 |
| 🧪 测试 | 测试结果截图/UI 对比图 | `understand_image`（ui-review/web-debug） | 测试子 Agent 启动时 |
| 🔍 审查 | 代码截图/UI 实现截图 | `understand_image`（web-debug） | 审查子 Agent 启动时 |
| 全阶段 | 会议录音/播客 | `understand_audio`（transcribe/summarize） | 用户提及时自动调用 |
| 全阶段 | 教程视频/演示 | `understand_video`（describe/summarize） | 用户提及时自动调用 |
| 全阶段 | 文字转语音需求 | `tts` | 用户提及时自动调用 |

### 6.5 多模态 MCP 配置

```toml
# reasonix.toml 中的配置
[[plugins]]
name    = "mimo-multimodal"
command = "node"
args    = [".../mimo-agent-skill/mcp-server/dist/index.js"]
env     = { MIMO_API_BASE = "https://api.xiaomimimo.com/v1", MIMO_API_KEY = "${MIMO_API_KEY}" }
```

**环境要求：**
- Node.js 18+
- 环境变量 `MIMO_API_KEY`（小米 MiMo API Key，从 https://platform.xiaomimimo.com 获取）
- MCP 服务器需要先构建（`npm install && npm run build`）

---

## 7. 防幻觉机制（6 层防线）

### 整体架构

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       6 层防幻觉防线                                     │
│                                                                         │
│  层① 进度文件跟踪系统     ← 防止"以为做完/漏做"                           │
│  层② AGENTS.md 兜底规则   ← 防止断点恢复时丢失进度                        │
│  层③ 完成前验证门控       ← 防止虚假成功声明                              │
│  层④ 架构评审 Go/No-Go   ← 防止在错误架构上构建                          │
│  层⑤ 进度文件注入         ← 防止跨 Agent 状态不一致                       │
│  层⑥ 拒绝猜测原则         ← 防止"合理推断"式幻觉                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### 层① 进度文件跟踪系统（三文件制）

**涉及技能：** writing-plans, executing-plans, wf-orchestrator

**工作机制：**

```
writing-plans 创建计划时
    │
    ├── 生成 docs/superpowers/plans/YYYY-MM-DD-<feature>.md
    │   （实施计划，含所有任务和步骤的详细代码）
    │
    └── 生成三个进度跟踪文件：
        │
        ├── task_plan.md
        │   - 从计划中提取的所有任务和步骤
        │   - 用复选框语法跟踪每步完成状态
        │   - 示例：
        │     - [ ] 任务 1：添加验证功能
        │       - [ ] 步骤 1：编写失败的测试
        │       - [x] 步骤 2：运行测试验证失败  ← 已勾选
        │       - [x] 步骤 3：编写最少实现代码
        │       - [ ] 步骤 4：运行测试验证通过
        │
        ├── findings.md
        │   - 记录执行过程中的决策和发现
        │   - 格式：
        │     | 时间 | 任务 | 发现 | 影响 |
        │     |------|------|------|------|
        │     | 07-07 | 任务2 | Redis 配置从 env 改为 yaml | 需更新配置文档 |
        │
        └── progress.md
            - 简明记录当前执行状态
            - 格式：
              总任务数：8
              已完成：3
              当前断点：任务 4 / 步骤 2
              阻塞项：无
```

**为什么这能防幻觉：**

| 问题 | 没有进度文件时 | 有进度文件时 |
|:----|:--------------|:------------|
| Agent 忘记做到哪一步 | 凭记忆推断，可能漏做或重做 | 精确读取 task_plan.md 的勾选状态 |
| 跨会话中断 | 重新开始，浪费之前的工作 | 检测到 task_plan.md，从断点续做 |
| 多个 Agent 状态不一致 | 各 Agent 各自记忆，互相矛盾 | 所有 Agent 读取同一份文件 |
| "以为做了实际上没做" | Agent 自信地说"已完成" | 复选框没勾 = 没做，无可争辩 |

---

### 层② AGENTS.md 兜底规则

**文件位置：** 项目根目录 `AGENTS.md`

**规则内容：**

```
在任何操作开始之前（包括响应澄清性问题之前），
检查 docs/superpowers/plans/task_plan.md 是否存在。
如果存在，读取它以了解当前执行进度。
这一规则适用于所有对话、所有场景。
```

**关键设计：**

- "在任何操作开始之前" — 包括看起来无关的澄清问题之前
- "包括响应澄清性问题之前" — 防止 Agent 先回答问题再检查
- "所有对话、所有场景" — 不论是否通过 wf-orchestrator 启动

**兜底效果：**

```
场景：用户打开新会话说"继续昨天的开发"
                                  │
      没有 AGENTS.md               │          有 AGENTS.md
                                  │
  Agent 可能回答：                  │   Agent 先检查 task_plan.md：
  "好的，让我们开始。请问           │   → 发现：已完成 3/8 任务
   你要做什么功能？"                │   → 断点：任务 4/步骤 2
                                  │   → 回答："检测到之前的执行进度。
                                  │      已完成 3/8 任务，断点在任务 4/
                                  │      步骤 2。是否续做？"
```

---

### 层③ 完成前验证门控（verification-before-completion）

**涉及技能：** verification-before-completion

**核心铁律：**

```
没有新鲜的验证证据，不许宣称完成
```

**门控函数（必须按顺序执行）：**

```
宣称任何状态之前：
    │
    ① 确定：什么命令能证明这个结论？
    ② 运行：执行完整命令（重新运行，完整执行）
    ③ 阅读：完整输出，检查退出码，统计失败数
    ④ 验证：输出是否支持这个结论？
       ├── 否 → 用证据说明实际状态
       └── 是 → 带证据陈述结论
    ⑤ 只有这时 → 才能做出结论
```

**禁止的信号词（看到就触发电警）：**

| 信号词 | 问题 |
|:-------|:-----|
| "应该能行了" | 没验证 |
| "我有信心" | 信心 ≠ 证据 |
| "看起来没问题" | 没运行命令 |
| "代理说成功了" | 信任代理报告而非独立验证 |
| "Linter 通过了" | Linter ≠ 编译器 |
| "就这一次" | 没有例外 |

**验证证据示例：**

```
✅ 正确的验证方式：
  $ npm test
  > 34/34 pass
  → "全部测试通过"

❌ 错误的验证方式：
  → "应该能通过了" / "看起来对了" / "我之前跑过"
```

---

### 层④ 架构评审 Go/No-Go

**涉及技能：** reasonix-arch-review, wf-orchestrator

**位置：** 阶段③（架构设计）之后、阶段⑤（编码）之前

**6 维度健康评分：**

| 维度 | 分值 | 核心检查 |
|:----|:----:|:---------|
| 模块化 | /4 | 单一职责？依赖反转？接口通信？ |
| 耦合度 | /4 | 扇入/扇出？循环依赖？框架解耦？ |
| 可测试性 | /4 | 脱离依赖测试？测试边界？DI 可替换？ |
| 演进能力 | /4 | 新增改几个文件？部分替换？ |
| 技术债务 | /4 | TODO？重复代码？测试覆盖？ |
| 接口设计 | /4 | API 自文档？错误处理统一？版本策略？ |

**决策逻辑：**

```
评分 ≥ 18 ──→ 🟢 进入编码阶段
                │
评分 12-17 ──→ 🟡 修复一般问题后再编码
                │   问题项自动追加到开发 prompt
                │
评分 < 12 ────→ 🔴 向用户报告严重问题
                    │
                    ├── 用户选择 "返回重做"
                    │   → 回到架构阶段
                    │
                    └── 用户选择 "继续"
                        → 记录风险，继续编码
```

**为什么这能防幻觉：**
- 编码前设置硬性质检关卡
- 量化评分而非模糊判断
- 多位专家视角交叉验证
- < 12 分是硬性拦截，防止在不可靠的架构上"编出正确但无用的代码"

---

### 层⑤ 进度文件注入

**涉及技能：** wf-orchestrator, executing-plans

**机制：**

编排器在派发每个子 Agent 时，**自动读取进度文件并拼入 prompt**：

```
===== 📋 进度跟踪 =====
计划文件：docs/superpowers/plans/YYYY-MM-DD-feature.md
当前进度：已完成 3/8 个任务
断点位置：任务 4 / 步骤 2
之前的发现：
  - 任务 2 中 Redis 配置从 env 改为 config.yaml
  - 任务 3 发现测试环境需要 mock 外部 API
=====
```

**子 Agent 每次拿到的都是精确的当前状态：**
- 不需要记忆之前的执行
- 不需要推断进度
- 不能"认为"某个步骤已完成
- 所有历史决策在 findings.md 中有记录

**跨 Agent 一致性保障：**
```
Agent A（规划）→ 写 task_plan.md
Agent B（架构）→ 读 task_plan.md + findings.md
Agent C（编码）→ 读 task_plan.md + findings.md
Agent D（测试）→ 读 task_plan.md + findings.md
         ↑                     ↑
    各自独立执行         但读到同一份事实
```

---

### 层⑥ 拒绝猜测原则

**涉及技能：** brainstorming, executing-plans, systematic-debugging

**规则：**

> 不要猜测意图，不要"合理推断"
> 列出你的理解和困惑，让用户澄清
> 等待回复后再继续

**触发条件：**

| 场景 | 禁止的行为 | 正确的做法 |
|:----|:----------|:-----------|
| 需求模糊 | Agent 自行假设用户意图 | 列出开放问题，让用户确认 |
| 依赖缺失 | 使用"假数据"继续推进 | 停下来报告缺失 |
| 指令不清 | 选择最可能的解释 | 列出理解，请用户澄清 |
| 验证反复失败 | 修改测试让它能过 | 报告失败，分析根因 |
| 计划有缺陷 | 边做边改 | 停下来，向用户报告并建议修正 |

**来自 executing-plans 的具体规则：**

```
在以下情况立即停止执行：
- 遇到阻塞（缺少依赖、测试失败、指令不清）
- 计划有严重缺陷导致无法开始
- 你不理解某条指令
- 验证反复失败（同一测试失败 2 次以上）

不确定时就问，不要猜测。
不要硬闯阻塞 — 停下来问。
```

---

### 防幻觉机制总表

| # | 防线 | 触发时机 | 防什么幻觉 | 涉及的技能/文件 |
|:-:|:-----|:---------|:-----------|:----------------|
| ① | 进度文件跟踪 | writing-plans 创建计划后持续生效 | 防止"以为做完/漏做/做重" | writing-plans, executing-plans |
| ② | AGENTS.md 兜底 | 每次 AI 响应前（所有对话） | 防止断点恢复时丢失进度 | AGENTS.md |
| ③ | 完成前验证门控 | 每次声称完成/通过/成功时 | 防止虚假成功声明 | verification-before-completion |
| ④ | 架构评审 Go/No-Go | 编码阶段开始前（阶段④） | 防止在错误架构上构建 | reasonix-arch-review, wf-orchestrator |
| ⑤ | 进度文件注入 | 每个子 Agent 派发时 | 防止跨 Agent 状态不一致 | wf-orchestrator, executing-plans |
| ⑥ | 拒绝猜测原则 | 遇到模糊/缺失/失败信息时 | 防止"合理推断"式幻觉 | brainstorming, executing-plans, systematic-debugging |

---

## 8. 中断恢复与会话交接

### 8.1 自动恢复流程

```
新会话启动
    │
    ▼
AGENTS.md 规则触发
    │
    ▼
检查 docs/superpowers/plans/task_plan.md
    │
    ├── 不存在 → 正常开始新对话
    │
    └── 存在 →
        │
        ▼
    读取 trio 文件：
    ├── task_plan.md  → 已勾选的任务和步骤
    ├── progress.md   → 断点位置、完成数
    └── findings.md   → 历史发现和决策
        │
        ▼
    向用户报告：
    "检测到之前的执行进度。
     已完成 3/8 个任务，
     断点在任务 4 / 步骤 2。
     是否续做？"
        │
        ├── 是 → 从断点处继续
        │
        └── 否 → 备份旧文件（.bak），从头开始
```

### 8.2 reasonix-handoff 会话交接

**适用场景：**
- 当前对话太长，想开新会话继续
- 要把任务分给另一个 AI 或同事
- 今天做不完明天继续

**交接流程：**

```
源会话
    │
    ├── 用户说"交接一下"
    │
    ├── reasonix-handoff 技能触发
    │   ├── 读取 task_plan.md
    │   ├── 读取 findings.md
    │   ├── 读取 progress.md
    │   └── 生成交接文档
    │       docs/superpowers/handoffs/YYYY-MM-DD-HHmm-项目名-handoff.md
    │
    └── 交接文档包含：
        1. 当前状态（项目/阶段/进度/阻塞项）
        2. 关键决策（技术选型/方案取舍）
        3. 待办事项（立即要做/需要注意）
        4. 发现和坑
        5. 上下文摘要

目标会话
    │
    ├── 加载交接文档
    ├── 检查 task_plan.md
    └── 询问用户是否载入上下文继续
```

**交接文档模板：**

```markdown
# 会话交接文档

**生成时间：** 2026-07-07 14:30
**来源会话：** Reasonix - MediaManager
**交接原因：** 对话过长

## 1. 当前状态
- **项目：** MediaManager
- **阶段：** 编码阶段
- **进度：** 已完成 3/8 个任务，断点在 任务 4/步骤 2
- **阻塞项：** 无

## 2. 关键决策
| 决策 | 选择 | 理由 |
|------|------|------|
| 导出库 | exceljs | 支持流式写入大数据量 |
| 认证方案 | JWT | 无状态，适合微服务 |

## 3. 待办事项
- [ ] 任务 4：实现 Excel 导出功能
- [ ] 任务 5：添加导出进度条

## 4. 发现和坑
- exceljs 的样式 API 和 xlsx 不同
- 测试时需要 mock 文件系统
```

### 8.3 不同中断场景的处理

| 中断场景 | 恢复方式 | 数据完整性 |
|:---------|:---------|:-----------|
| 当前会话正常继续 | 无需操作 | ✅ 完整 |
| 当前会话 `/clear` | executing-plans 自动触发 handoff | ✅ 三文件完整 |
| 意外断线 | AGENTS.md 下次启动时检测 | ✅ task_plan.md 记录最后完成步骤 |
| 跨设备切换 | reasonix-handoff 导出 → 导入 | ✅ 含决策和发现 |
| 跨天继续 | AGENTS.md + task_plan.md | ✅ progress.md 记录断点 |

---

## 9. 知识沉淀体系

### 9.1 三载体架构

```
┌─────────────────────────────────────────────────────────────┐
│                   知识沉淀体系                               │
│                                                             │
│  CONTEXT.md ← 通用语言（领域术语表）                          │
│     由 brainstorming 和 domain-modeling 共同维护              │
│     每次对话中只要涉及术语澄清就立即更新                       │
│                                                             │
│  docs/adr/ ← 架构决策记录                                    │
│     由 domain-modeling 和 arch-review 在关键技术选型时写入    │
│     3-5 句话，轻量级，不追求格式完美                          │
│                                                             │
│  findings.md ← 发现记录                                      │
│     由 executing-plans 在执行过程中实时写入                   │
│     踩坑记录、关键发现、偏离计划的决定                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 9.2 CONTEXT.md 维护规则

**触发规则（来自 brainstorming）：**
- 用户说了一个业务概念并给出定义 → 立即追加
- 发现两个术语是同义词 → 记录禁用别称
- 发现概念之间有特殊关系 → 更新实体关系描述

**示例变化过程：**

```
第 1 轮：用户说"一个用户可以创建多个播放列表"
  → 立即追加：
    ## 通用语言
    - 播放列表（Playlist）：视频的有序集合

第 2 轮：用户说"播放列表以前叫歌单，现在统一叫播放列表"
  → 追加：
    - 播放列表（Playlist）：视频的有序集合
    - 禁止使用：歌单、收藏夹
```

### 9.3 ADR 写入规则

**什么时候写 ADR：**
- ✅ 技术选型（用 A 库还是 B 库）
- ✅ 方案取舍（方案二是我们选的，为什么放弃方案一）
- ✅ 约束条件（必须用某个框架、必须兼容某个版本）
- ❌ 不写：业务逻辑细节、实现步骤、临时决定

**ADR 格式（存到 docs/adr/）：**

```markdown
# ADR-0001：使用 exceljs 而非 xlsx 库

## 背景
需要导出 Excel 文件，社区主流有 xlsx 和 exceljs 两个库。

## 决策
选用 exceljs

## 理由
- 支持流式写入，大数据量不 OOM（xlsx 全量加载）
- 活跃维护（上次 commit 在 1 周内）
- 支持样式定制（xlsx 不支持）

## 后果
- 学习成本：团队需熟悉 exceljs API
- xlsx 的相关代码需迁移
```

### 9.4 findings.md 维护规则

**由 executing-plans 在执行过程中实时写入：**

```
| 时间 | 任务 | 发现 | 影响 |
|------|------|------|------|
| 07-07 | 任务2 | Redis 配置从 env 改为 config.yaml | 需更新配置文档 |
| 07-07 | 任务3 | 测试环境需要 mock 外部 API | 需添加 mock 依赖 |
```

**偏离计划的决定：**

```
| 时间 | 原计划 | 实际做法 | 原因 |
|------|--------|---------|------|
| 07-07 | 用 axios | 用 fetch | 项目已内置 fetch，避免多一个依赖 |
```

---

## 10. 文件结构与产出物规范

### 10.1 工作流会话目录结构

```
workflow/<YYYYMMDD_HHmmss_项目名>/
├── MANIFEST.md              ← 会话清单（项目/需求/模型分配/阶段范围）
├── 01-plan.md               ← 规划文档（阶段①产出）
├── 02-design.md             ← 架构设计文档（阶段③产出）
├── 03-arch-review.md        ← 架构评审报告（阶段④产出）
├── 03-implementation/       ← 代码变更目录（阶段⑤产出）
│   └── CHANGES.md           ← 变更说明
├── 04-test/
│   └── TEST_REPORT.md       ← 测试报告（阶段⑥产出）
├── 05-review.md             ← 审查报告（阶段⑧产出）
├── 06-deploy/
│   └── DEPLOY.md            ← 部署方案（阶段⑨产出）
├── SUMMARY.md               ← 会话总结（阶段⑩产出）
└── ERRORS.log               ← 错误日志
```

### 10.2 项目级知识目录结构

```
docs/
├── superpowers/
│   ├── specs/               ← 设计规格（brainstorming 产出）
│   │   └── YYYY-MM-DD-<topic>-design.md
│   ├── plans/               ← 实施计划（writing-plans 产出）
│   │   ├── YYYY-MM-DD-<feature>.md
│   │   ├── task_plan.md     ← 可勾选任务清单
│   │   ├── findings.md      ← 发现记录
│   │   ├── progress.md      ← 进度摘要
│   │   └── ARCH_REVIEW.md   ← 架构评审报告
│   ├── domain-models/       ← 领域模型（domain-modeling 产出）
│   │   └── YYYY-MM-DD-<feature>-domain.md
│   └── handoffs/            ← 会话交接文档
│       └── YYYY-MM-DD-HHmm-<project>-handoff.md
├── adr/                     ← 架构决策记录
│   ├── ADR-0001-<title>.md
│   └── ADR-0002-<title>.md
└── CONTEXT.md               ← 通用语言/领域术语表
```

### 10.3 各阶段产出物检查清单

| 阶段 | 产出物 | 验证方式 | 缺失处理 |
|:----|:-------|:---------|:---------|
| 📋 规划 | `01-plan.md` | 文件存在 | 重试/跳过 |
| 🧩 领域建模 | 领域模型文档 | 文件存在 | 继续（非强制） |
| 🏗️ 架构 | `02-design.md` | 文件存在 | 重试/跳过 |
| 🏛️ 评审 | `03-arch-review.md` | 文件存在 | 重试/跳过 |
| 💻 编码 | `03-implementation/` 有内容 | 目录非空 | 重试/跳过 |
| 🧪 测试 | `TEST_REPORT.md` | 文件存在 | 重试/跳过 |
| 🔬 扫描 | HTML 报告 | 文件存在 | 继续（非强制） |
| 🔍 审查 | `05-review.md` | 文件存在 | 重试/跳过 |
| 🚀 部署 | `DEPLOY.md` | 文件存在 | 重试/跳过 |

---

## 11. 错误处理与异常流程

### 11.1 子 Agent 调用失败

```
子 Agent 调用超时/报错
    │
    ▼
记录错误到 {SESSION_DIR}/ERRORS.log
    │
    ▼
向用户报告：
"[阶段名] 执行失败：{错误信息}"
    │
    ├── 重试该阶段
    ├── 跳过该阶段
    └── 中止工作流
```

### 11.2 产出验证失败

```
阶段完成后，预期产出文件不存在
    │
    ▼
向用户报告：
"「{阶段名}」已完成，但未检测到 {预期产出的文件}。"
    │
    ├── 重新执行该阶段
    └── 跳过（手动处理）
```

### 11.3 架构评审未通过

```
reasonix-arch-review 评分 < 12
    │
    ▼
向用户报告严重问题：
"架构评审未通过（评分 X/24），存在以下严重问题："
    │
    ├── 返回架构阶段重做
    └── 继续（记录风险）
```

### 11.4 构建失败

```
部署阶段构建失败
    │
    ▼
记录错误详情
    │
    ▼
给出可能的修复建议
    │
    ▼
询问用户是否修复后重试
```

### 11.5 测试反复失败

```
同一测试失败 2 次以上
    │
    ▼
停止执行，向用户报告
    │
    ├── 分析根因后重试
    └── 跳过该测试
```

---

## 12. 安装与配置

### 12.1 一键安装

在 Reasonix 中执行：
```
安装工作流，仓库 https://github.com/TonyQ-AI/reasonix-workflow
```

### 12.2 手动安装

```bash
git submodule add https://github.com/TonyQ-AI/reasonix-workflow
cp -r reasonix-workflow/skills/* .reasonix/skills/
cp reasonix-workflow/configs/AGENTS.md AGENTS.md
# 可选：MiMo 多模态 MCP 参考配置
cp reasonix-workflow/configs/mcp-mimo.json .
```

### 12.3 环境变量配置

```powershell
# 需要设置的环境变量（在 Reasonix 全局 .env 中）
DEEPSEEK_API_KEY=sk-xxx          # 必需：所有 Agent 模型
MIMO_API_KEY=sk-xxx              # MiMo 多模态 MCP（图片/音频/视频/TTS）
BAIDU_OCR_API_KEY=bce-v3/xxx     # 百度 OCR 文档解析 MCP
```

### 12.4 MiMo MCP 服务器构建

```bash
cd mimo-agent-skill/mcp-server
npm install
npm run build
```

### 12.5 百度 OCR MCP 配置

无需安装任何本地依赖，直接在 reasonix.toml 中添加：

```toml
[[plugins]]
name    = "baidu-ocr-document"
type    = "http"
url     = "https://aip.baidubce.com/mcp/document/sse"
headers = { Authorization = "Bearer ${BAIDU_OCR_API_KEY}" }
```

### 12.6 reasonix.toml 完整配置

```toml
[skills]
paths = [".../reasonix-workflow/skills"]

[[plugins]]
name    = "mimo-multimodal"
command = "node"
args    = [".../mimo-agent-skill/mcp-server/dist/index.js"]
env     = { MIMO_API_BASE = "https://api.xiaomimimo.com/v1", MIMO_API_KEY = "${MIMO_API_KEY}" }
```

---

## 13. 使用场景与最佳实践

### 13.1 新功能开发（推荐全流程）

```
/wf-orchestrator MediaManager: 新增视频批量导出功能
```

**最佳实践：**
- 如果需求涉及 UI，先准备好设计稿截图
- 如果需求模糊，让规划阶段多问几个澄清问题
- 架构评审如果 🟡 12-17 分，先修复问题再编码

### 13.2 Bug 修复（局部执行）

```
/wf-orchestrator MediaManager: 修复登录页样式错乱 --from dev --to review
```

**最佳实践：**
- 准备 Bug 截图，方便多模态 MCP 分析
- 提供复现步骤，让 systematic-debugging 更准确

### 13.3 系统重构（从架构扫描开始）

```
/wf-orchestrator MediaManager: 重构用户模块 --from plan
```

**最佳实践：**
- 先执行架构扫描看当前代码问题
- 根据扫描结果制定重构计划
- 小步迭代，每次重构一个模块

### 13.4 含媒体文件的项目

```
/wf-orchestrator MediaManager: 参考这张设计稿实现登录页面
```

**最佳实践：**
- 提供本地文件路径（如 `C:/design.png`）
- MCP 自动分析设计稿，提取 UI 元素和样式
- 审查阶段可以拿实现截图和原设计稿对比

### 13.5 防幻觉最佳实践

| 做法 | 说明 |
|:----|:------|
| 工作流执行中不要频繁中断 | 进度文件虽然支持恢复，但频繁中断增加 Agent 上下文切换成本 |
| 架构评审 🟡 时认真修复 | 12-17 分意味着架构有问题，跳过修复会导致编码阶段产生更多幻觉 |
| 用 `--from`/`--to` 缩小范围 | 减小每个 Agent 的上下文窗口，降低幻觉概率 |
| findings.md 看到就记 | 踩坑不记 = 下次可能接着踩 |
| 不确定就重新执行 | 编排器支持重试单个阶段，无需从头开始 |

### 13.6 成本优化建议

| 场景 | 建议 | 节省 |
|:----|:-----|:-----|
| 小改动 | `--from dev --to review` 跳过规划/架构 | 50% Token |
| 纯编码 | 单 Agent `/wf-developer` | 80% Token |
| 禁用多模态 | 不配置 MIMO_API_KEY，MCP 自动不启动 | — |
| 熟悉领域 | `--no-superpowers` 跳过方法论注入 | 20% Token |

---

## 14. 附录：技能完整清单

### 14.1 核心工作流（7个）

| 技能名 | 作用 | 模型 | `/` 命令 |
|:-------|:-----|:----|:---------|
| wf-orchestrator | 主编排器，串联 8 阶段全流程 | — | `/wf-orchestrator` |
| wf-planner | 需求分析、任务分解、计划生成 | DeepSeek Pro 🧠 | `/wf-planner` |
| wf-architect | 系统架构设计、接口定义 | DeepSeek Pro 🧠 | `/wf-architect` |
| wf-developer | 代码实现、单元测试（TDD） | DeepSeek Flash ⚡ | `/wf-developer` |
| wf-tester | 测试用例设计、并行执行 | DeepSeek Flash ⚡ | `/wf-tester` |
| wf-reviewer | 代码审查、质量裁决 | DeepSeek Pro 🧠 | `/wf-reviewer` |
| wf-deployer | 构建打包、部署方案 | DeepSeek Flash ⚡ | `/wf-deployer` |

### 14.2 架构评审与扫描（1个技能，2种用法）

| 技能名 | 用法 | 作用 | 模型 |
|:-------|:----|:-----|:----|
| reasonix-arch-review | 设计评审（编码前） | 专家模拟 + 6维评分 + Go/No-Go | DeepSeek Pro 🧠 |
| reasonix-arch-review | 架构扫描（编码后） | 删除测试 + 依赖图 + HTML报告 | DeepSeek Pro 🧠 |

### 14.3 方法论注入（5个）

| 技能名 | 注入阶段 | 作用 |
|:-------|:---------|:-----|
| test-driven-development | 💻 编码 | 红→绿→重构循环 |
| systematic-debugging | 💻 编码（修Bug） | 定位→分析→假设→修复→验证 |
| dispatching-parallel-agents | 🧪 测试 | 并行执行独立测试 |
| requesting-code-review | 🔍 审查 | 逐文件标注行号 |
| chinese-code-review | 🔍 审查 | 建设性中文反馈 |

### 14.4 规划与支撑（5个）

| 技能名 | 作用 | `/` 命令 |
|:-------|:-----|:---------|
| brainstorming | 需求分析、方案对比、术语积累 | `/brainstorming` |
| writing-plans | 计划生成 + 三文件进度跟踪 | `/writing-plans` |
| executing-plans | 自动检测进度、中断续做 | `/executing-plans` |
| reasonix-domain-modeling | 领域建模、通用语言、ADR | `/reasonix-domain-modeling` |
| reasonix-handoff | 会话交接（跨设备/跨会话） | `/reasonix-handoff` |

### 14.5 扩展能力（5个）

| 技能名 | 作用 |
|:-------|:-----|
| reasonix-ui-design | 专业UI设计（防AI模板 + 设计令牌） |
| workflow-runner | 通用 YAML 工作流执行器 |
| finishing-a-development-branch | 分支集成、PR、合并规范 |
| verification-before-completion | 完成前自动验证 |
| subagent-driven-development | 子代理驱动开发 |

### 14.6 MiMo 多模态（4个）

| 技能名 | MCP 工具 | 作用 |
|:-------|:---------|:-----|
| mimo-image-understanding | `understand_image` | 图片理解、OCR、UI评审、图表提取、前端调试 |
| mimo-audio-understanding | `understand_audio` | 音频转写、会议记录、内容摘要 |
| mimo-video-understanding | `understand_video` | 视频描述、教程总结、动作检测 |
### 14.7 百度 OCR 文档解析（1个）

| 技能名 | MCP 工具 | 作用 |
|:-------|:---------|:-----|
| baidu-unlimited-ocr | `ocr_document` / `ocr_id_card` 等 | 文档解析、表格识别、PDF提取、手写体识别、30+种证件结构化识别 |

---

## 版本信息

| 版本 | 日期 | 变更 |
|:----|:----|:-----|
| v2.1 | 2026-07-07 | 新增百度 OCR 文档解析 MCP + 智能选择策略，技能总数 26→27 |
| v2.0 | 2026-07-07 | 新增 MiMo 多模态 MCP 集成（4个技能），技能总数 22→26 |
| v1.0 | — | 初始版本，22 个技能 |

---

*本文档由 Reasonix Workflow 自动生成*
*仓库：https://github.com/TonyQ-AI/reasonix-workflow*
