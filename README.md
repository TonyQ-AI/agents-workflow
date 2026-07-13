# Workflow Dualmode

> 多Agent协同开发工作流 · 双模式 · 自动检测 task 可用性 · 40 个技能

[![Version](https://img.shields.io/badge/version-4.1.4-blue)](VERSION)
[![Skills](https://img.shields.io/badge/skills-40-green)](skills/)

## 双模式自动切换

本工作流自动检测平台能力，无需手动选择：

| 模式 | 条件 | 执行方式 |
|------|------|---------|
| **正常模式** | task 工具可用 | 编排器 → task(引擎 subagent) → task(子Agent) |
| **inline 模式** | task 不可用 | 编排器直接执行所有阶段 |

## 安装

```
装工作流
```

## 使用

```
/wf-orchestrator 项目名: 需求描述
/wf-orchestrator 项目名: 需求描述 --lite
/wf-orchestrator 项目名: 需求描述 --from coding
```

## 项目关系

```
workflow-task（支持 task 的工具 · 40 技能）
       ↓ 改造
workflow-inline（纯 inline · 39 技能 · 无 task 依赖）
       ↓ 合并
Workflow-Dualmode（双模式 · 40 技能 · 自适应）
```

## 更新日志

### v4.1.4（2026-07-13）— 引擎技能修复

- 修复 `wf-orchestrator-engine/SKILL.md` YAML 前页缺少闭合 `---` 分隔符

### v4.1.3（2026-07-13）— 全平台适配

- MCP 独立发布：`tonyq-mimo-mcp-server` v0.2.2，默认 Standard API
- 安装器自动检测 MiMo 端点类型
- 技能目录清理：移除 reasonix- 前缀
- 路径中性化：`.reasonix/skills` → `skills/`

### v4.1.2（2026-07-13）

- MCP 服务迁移：`mimo-mcp-server` → `tonyq-mimo-mcp-server`
- 工具名对齐：统一加 `mimo_` 前缀
- 环境变量修复：`MIMO_API_BASE` → `MIMO_API_URL`
