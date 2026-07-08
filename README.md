# Reasonix Workflow

> 多 Agent 协同开发全流程自动化流水线 · 13 阶段 · 36 技能 · 2 套多模态 MCP

---

## 概述

Reasonix Workflow 是一套多 Agent 协同开发流水线，通过**编排器 + 6 个专业化子 Agent** 分工协作，配合**交互式需求澄清**、**领域建模**、**智能方法论注入**和**完整的产出门控**，实现软件开发全流程自动化。

### 核心理念

- **交互式澄清** — 规划阶段编排器亲自与用户逐轮对话，每次一个问题
- **领域驱动** — 规划后执行领域建模，提取实体关系、通用语言
- **分工专业化** — Pro 模型负责推理（规划/架构/评审），Flash 负责执行（编码/测试/部署）
- **方法论驱动** — 根据任务类型智能注入 TDD / 系统化调试 / 代码审查
- **容错设计** — task 不可用时自动降级为单Agent顺序执行
- **全产出门控** — 每个阶段都有明确的产出验证
- **可恢复** — checkpoint + progress.md + findings.md 三文件体系

## 13 阶段流程

```
①规划(交互) → ②领域建模 → ③架构 → ④架构评审(--lite跳过)
→ ⑤编码(含UI按需) → ⑥测试 → ⑦架构扫描(--lite跳过)
→ ⑧审查 → ⑨部署 → ⑩文档检查 → ⑪分支收尾 → ⑫版本登记 → ⑬总结
```

### 执行者与模型

| 阶段 | 执行者 | 模型 |
|------|--------|------|
| ①规划 | 编排器(交互) | — |
| ②领域建模 | task → 子Agent | DeepSeek Pro 🧠 |
| ③架构 | task → 子Agent | DeepSeek Pro 🧠 |
| ④架构评审 | 编排器注入方法论 | — |
| ⑤编码 | task → 子Agent | DeepSeek Flash ⚡ |
| ⑥测试 | task → 子Agent | DeepSeek Flash ⚡ |
| ⑦架构扫描 | 编排器注入方法论 | — |
| ⑧审查 | task → 子Agent | DeepSeek Pro 🧠 |
| ⑨部署 | task → 子Agent | DeepSeek Flash ⚡ |
| ⑩文档检查 | 编排器 | — |
| ⑪分支收尾 | 编排器 | — |
| ⑫版本登记 | 编排器 | — |
| ⑬总结 | 编排器 | — |

## 快速开始

```bash
/wf-orchestrator 项目名: 需求描述
/wf-orchestrator 项目名: 需求描述 --lite       # 跳过架构评审和扫描
/wf-orchestrator 项目名: 需求描述 --from coding  # 从编码阶段开始
/wf-orchestrator 项目名: 需求描述 --model deepseek # 覆盖模型
```

## 参数

| 参数 | 说明 |
|------|------|
| `--from <阶段>` | 起始阶段：planning/domain_modeling/architecture/arch_review/coding/testing/arch_scan/review/deploy/doc_check/branch_finish/version-tracking |
| `--to <阶段>` | 结束阶段（同上） |
| `--model <模型>` | 全局覆盖模型 |
| `--lite` | 轻量模式，跳过架构评审和架构扫描 |
| `--timeout <秒>` | 子Agent超时，默认300秒 |
| `--no-superpowers` | 禁用方法论注入 |

## 技能体系（36个）

| 分类 | 数量 |
|------|:----:|
| 核心工作流 | 6 (orchestrator + 5 sub-agents) |
| 架构评审/扫描 | 1 (2 种用法) |
| 方法论注入 | 6 |
| 规划与支撑 | 5 |
| 领域建模 | 1 |
| 文档/提交/版本 | 4 |
| UI 设计 | 1 |
| 多模态 MCP | 4 |
| 其他独立技能 | 8 |
| **总计** | **36** |

## 配置

### 环境变量

```env
DEEPSEEK_API_KEY=sk-xxx
MIMO_API_KEY=sk-xxx
```

### MCP 服务器

```toml
[[plugins]]
name    = "mimo-multimodal"
command = "npx"
args    = ["mimo-mcp-server", "-y"]
env     = { MIMO_API_KEY = "${MIMO_API_KEY}" }
```

## 文档

完整工作流说明书见 [WORKFLOW_MANUAL.md](WORKFLOW_MANUAL.md)。

## 许可证

MIT
