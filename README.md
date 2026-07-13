# Multi-Agent Automated Workflow

> 多Agent协同开发全流程自动化工作流 · 双模式 · 40 个技能 · 13 阶段

## 双模式自动切换

本工作流自动检测平台能力，无需手动选择：

| 模式 | 条件 | 执行方式 |
|------|------|---------|
| **正常模式** | task 工具可用 | 编排器 → task(引擎 subagent) → task(子Agent) |
| **inline 模式** | task 不可用 | 编排器直接执行所有阶段 |

启动时自动输出当前模式：
```
✅ 正常模式 — 通过 task 派发子Agent
⚠️ inline 模式 — task 不可用，编排器直接执行
```

## 一句话安装

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
reasonix-workflow（Reasonix 专用 · 40技能）
       ↓ 改造
workflow-inline（纯 inline · 39技能 · 无 task 依赖）
       ↓ 合并
Multi-Agent Automated Workflow（双模式 · 40技能 · 自适应）
```

## 配置

- `MIMO_API_URL`：`https://api.xiaomimimo.com/v1/chat/completions`
- `MIMO_API_KEY`：存入 `.env`，配置引用 `${MI*******KEY}`
- `DEEPSEEK_API_KEY`：工作流推理
