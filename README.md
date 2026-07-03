# Reasonix Workflow — 多Agent协同开发工作流

> 一套基于 AI Agent 的全流程自动化软件开发流水线。  
> 从需求分析到代码实现、测试、审查、部署，全部由专业 Agent 分工协作完成。

---

## 快速体验

在 Reasonix 中启动一个完整的开发工作流，如：

```
/wf-orchestrator 我的项目: 新增用户登录模块，支持邮箱和微信登录
```

一句话 → 自动完成规划、设计、编码、测试、审查、部署全流程。

---

## 工作流全景

```
┌─────────────────────────────────────────────────────────────────────┐
│  wf-orchestrator — 主编排器                                        │
│  输入: /wf-orchestrator <项目名>: <需求描述>                         │
│  输出: workflow/<时间戳_项目名>/ 结构化会话目录                       │
└─────────────────────────────────────────────────────────────────────┘
         │
         ▼   8 个阶段由编排器自动串联
┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
│ 📋   │ 🧩   │ 🏗️   │ 🏛️   │ 💻   │ 🧪   │ 🔬   │ 🔍   │
│规划  │领域  │架构  │架构  │编码  │测试  │架构  │审查  │
│      │建模  │设计  │评审  │      │      │扫描  │      │
│ Pro  │ Pro  │ Pro  │ Pro  │Flash │Flash │ Pro  │ Pro  │
└──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘
         │
         ▼
┌──────────────────┐
│ 🚀 部署方案      │
│ (构建+配置+版本) │
└──────────────────┘
```

---

## 包含的全部技能（22个）

### 核心工作流（7个）

| 技能 | 作用 | 模型 |
|------|------|------|
| **wf-orchestrator** | 主编排器，串联全流程 | — |
| **wf-planner** | 需求分析、任务分解 | DeepSeek Pro 🧠 |
| **wf-architect** | 系统架构设计、接口定义 | DeepSeek Pro 🧠 |
| **wf-developer** | 代码实现、单元测试 | DeepSeek Flash ⚡ |
| **wf-tester** | 测试用例设计、执行 | DeepSeek Flash ⚡ |
| **wf-reviewer** | 代码审查、质量裁决 | DeepSeek Pro 🧠 |
| **wf-deployer** | 构建打包、部署方案 | DeepSeek Flash ⚡ |

### 架构评审与扫描（1个技能，2种用法）

| 原始技能 | 重命名 | 作用 | 模型 |
|------|------|------|------|
| **基于reasonix-arch-review** | arch-review 架构评审（编码前） | 专家模拟 + 6维度评分 + Go/No-Go | DeepSeek Pro 🧠 |
| | arch-scan 架构扫描（编码后） | 删除测试 + 依赖图 + HTML报告 | DeepSeek Pro 🧠 |

### 方法论注入（5个）

| 技能 | 注入阶段 | 作用 |
|------|---------|------|
| **test-driven-development** | 💻 编码 | 红→绿→重构循环 |
| **systematic-debugging** | 💻 编码（修Bug） | 定位→分析→假设→修复→验证 |
| **dispatching-parallel-agents** | 🧪 测试 | 并行执行独立测试 |
| **requesting-code-review** | 🔍 审查 | 逐文件标注行号 |
| **chinese-code-review** | 🔍 审查 | 建设性中文反馈 |

### 规划与支撑（5个）

| 技能 | 作用 |
|------|------|
| **brainstorming** | 需求分析、方案对比 |
| **writing-plans** | 计划生成 + 三文件进度跟踪 |
| **executing-plans** | 自动检测进度、中断续做 |
| **reasonix-domain-modeling** | 领域建模、通用语言、ADR |
| **reasonix-handoff** | 会话交接（跨设备/跨会话） |

### 扩展能力（4个）

| 技能 | 作用 |
|------|------|
| **reasonix-ui-design** | 专业UI设计（防AI模板 + 设计令牌） |
| **workflow-runner** | 通用 YAML 工作流执行器 |
| **finishing-a-development-branch** | 分支集成、PR、合并规范 |
| **verification-before-completion** | 完成前自动验证 |

---

## 安装

### 方式一：一键安装（推荐）

在 Reasonix 中执行：

```
安装工作流，仓库 https://github.com/TonyQ-AI/reasonix-workflow
```

或者直接大白话告诉ai github项目名：reasonix-workflow 让它安装

会自动下载所有技能并配置 AGENTS.md。


### 方式二：Git Submodule

```bash
git submodule add https://github.com/TonyQ-AI/reasonix-workflow
cp -r reasonix-workflow/skills/* .reasonix/skills/
cp reasonix-workflow/configs/AGENTS.md AGENTS.md
```

---

## 使用

### 全流程开发

```
/wf-orchestrator <项目名>: <需求描述>
```

### 局部执行（跳过某些阶段）

```
/wf-orchestrator <项目名>: <需求描述> --from dev --to review
```

### 覆盖模型

```
/wf-orchestrator <项目名>: <需求描述> --model deepseek
```

### 单个Agent调用

```
/wf-planner 设计一个用户权限管理模块
/wf-developer 实现上述计划中的任务1
```

### 更多参数

| 参数 | 说明 |
|------|------|
| `--from <阶段>` | 从指定阶段开始（plan/arch/arch-review/dev/test/arch-scan/review/deploy） |
| `--to <阶段>` | 在指定阶段结束 |
| `--model <模型>` | 全局覆盖模型 |
| `--parallel` | 开发与测试并行 |
| `--no-superpowers` | 禁用方法论注入 |

---

## 进度检查与中断恢复

工作流内置三文件进度跟踪系统，任何时候中断都能无缝恢复：

```
docs/superpowers/plans/
├── task_plan.md     ← 任务勾选框清单
├── findings.md      ← 发现记录（决策/踩坑/注意事项）
└── progress.md      ← 进度摘要（已完成数、断点位置）
```

下一会话开始时自动检测并提示续做。

---

## 知识沉淀

每次开发自动积累：

| 载体 | 位置 | 记录什么 |
|------|------|---------|
| **CONTEXT.md** | 项目根目录 | 领域术语、通用语言 |
| **docs/adr/** | 项目内 | 架构决策记录（方案对比、结论） |
| **findings.md** | plans/ 目录 | 踩坑记录、关键发现 |

---

## 依赖要求

- **Reasonix** 桌面版 ≥ 1.0.0 或 CLI
- **DeepSeek API Key** — 全部 Agent 使用，Pro 模型用于架构评审/扫描
- 可选：**MiMo API Key** — 仅多模态场景
- 可根据需求自行为各agent分配更合适的模型

---

## 目录结构

```
reasonix-workflow/
├── README.md                 ← 本文件
├── LICENSE                   ← MIT
├── configs/
│   └── AGENTS.md             ← 进度检查规则（追加到目标项目）
└── skills/
    ├── wf-orchestrator/      ← 主编排器（入口）
    ├── wf-planner/           ← 规划Agent
    ├── wf-architect/         ← 架构Agent
    ├── wf-developer/         ← 编码Agent
    ├── wf-tester/            ← 测试Agent
    ├── wf-reviewer/          ← 审查Agent
    ├── wf-deployer/          ← 部署Agent
    ├── reasonix-arch-review/ ← 架构评审 + 架构扫描
    ├── reasonix-domain-modeling/ ← 领域建模
    ├── reasonix-handoff/     ← 会话交接
    ├── reasonix-ui-design/   ← UI设计
    ├── brainstorming/        ← 需求分析
    ├── writing-plans/        ← 计划 + 进度跟踪
    ├── executing-plans/      ← 中断续做
    ├── test-driven-development/   ← TDD方法论
    ├── systematic-debugging/      ← 系统化调试
    ├── dispatching-parallel-agents/ ← 并行测试
    ├── requesting-code-review/    ← 代码审查方法
    ├── chinese-code-review/       ← 中文审查风格
    ├── finishing-a-development-branch/ ← 分支集成
    ├── verification-before-completion/ ← 完成验证
    ├── workflow-runner/           ← YAML工作流执行器
    └── ...                       ← 更多辅助技能
```

---

## 分享与分发

想让别人也用上这套工作流？

1. 告诉对方仓库地址
2. 对方只需在 Reasonix 中说：
   安装工作流，仓库 https://github.com/TonyQ-AI/reasonix-workflow

---

## 许可证

MIT — 自由使用、修改、分享。
