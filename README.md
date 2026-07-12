# Reasonix Workflow v4.1

> 多 Agent 协同开发全流程自动化 · 双引擎 · 12项任务 · 40技能 · 硬门控 · fallback兜底

[![Version](https://img.shields.io/badge/version-4.1-blue)](VERSION)
[![Skills](https://img.shields.io/badge/skills-40-green)](skills/)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)

---

## 是什么

一个 ZCode 技能包，让 AI 自动完成从需求到部署的全流程开发。

你说一句话 `"/wf-orchestrator MediaManager: 新增批量导出"`，编排器自动串起需求澄清 -> 领域建模 -> 架构设计 -> 硬门控评审 -> 编码测试 -> 审查部署 -> 文档检查 -> 分支收尾 -> 版本登记 -> 知识沉淀。

**双引擎架构**：编排器在独立上下文中执行12项任务清单，不数"第几个阶段"，只做"下一项是什么"。

---

## 怎么用

```bash
# 标准全流程
/wf-orchestrator MediaManager: 新增视频批量导出

# 轻量模式（跳评审+扫描）
/wf-orchestrator MediaManager: 修复样式 --lite

# 断点恢复
/wf-orchestrator MediaManager: 继续 --from coding

# 并行加速
/wf-orchestrator MediaManager: 重构模块 --parallel
```

---

## 参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `--from <阶段>` | planning / domain_modeling / architecture / arch_review / coding / testing / arch_scan / review / deploy / doc_check / branch_finish / version_tracking | 起始阶段 |
| `--to <阶段>` | 同上 | 结束阶段 |
| `--parallel` | — | 编码与测试并行 |
| `--lite` | — | 跳过架构评审和扫描 |
| `--model <模型>` | deepseek / mimo | 全局模型覆盖 |
| `--timeout <秒>` | 默认300 | 子Agent超时 |

---

## 核心特性

| 特性 | 说明 |
|:-----|:------|
| **双引擎** | 编排器交互规划 + 引擎独立上下文执行12项任务 |
| **硬门控** | 架构评审 18/24 分 Go（75%通过率），最多回退3次 |
| **fallback兜底** | task()不可用时引擎直执行，不中断 |
| **交叉验证** | 每层子Agent对照原始specs校验，防需求漂移 |
| **知识沉淀** | 跨会话积累项目知识库 KNOWLEDGE.md |
| **MCP双模式** | MiMo多模态(4技能) + 百度OCR(SSE+REST双模式) |

---

## 执行流程

```
编排器（主会话）:
  解析参数 -> 交互式需求澄清 -> 产出 task_plan + specs
  打包 JSON -> task(wf-orchestrator-engine)

执行引擎（独立上下文）:
  任务1: 领域建模 -> 任务2: 架构设计 -> 任务3: 架构评审(硬门控)
  -> 任务4: 编码 -> 任务5: 测试 -> 任务6: 架构扫描(硬门控)
  -> 任务7: 审查 -> 任务8: 部署 -> 任务9: 文档检查
  -> 任务10: 分支收尾 -> 任务11: 版本登记 -> 任务12: 总结+知识沉淀
```

---

## 技能体系

| 分类 | 技能 |
|:-----|:-----|
| 编排引擎 (2) | wf-orchestrator, wf-orchestrator-engine |
| 子Agent (5) | wf-architect, wf-developer, wf-tester, wf-reviewer, wf-deployer |
| 架构设计 (4) | arch-review, domain-modeling, ui-design, handoff |
| 方法论注入 (7) | brainstorming, writing/executing-plans, TDD, debug, subagent-driven, parallel |
| 审查规范 (4) | code-review(中/英/反馈) + chinese-documentation |
| 提交版本 (5) | commit规范, git工作流, branch-finish, worktrees, vt |
| 多模态MCP (5) | MiMo(4) + BaiduOCR(1) |
| 辅助扩展 (8) | trigger-commands, workflow-runner, verification, writing-skills, mcp-builder, superpowers, installer, wf-planner |

**总计 40 技能**

---

## 文档

[完整技术手册 (WORKFLOW_MANUAL.md)](WORKFLOW_MANUAL.md) — 双引擎设计、执行流程、硬门控机制、兜底降级、进度跟踪、交叉验证、知识沉淀、版本演进。

---

## v4.1 更新

23项深度审查修复：任务清单替代阶段编号、评分标准统一18/24、回退上限3次、--to/--parallel完整实现、fallback自检降级、所有子Agent输入统一task_plan.md。

详见 [v4.x 版本演进](WORKFLOW_MANUAL.md#12-v4x-版本演进)。

---

MIT
