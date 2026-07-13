# agents-workflow 安装指南

> 一键安装/升级多Agent协同开发工作流 · 双模式自适应 · 40 个技能

---

## 快速安装

**方式一：让 AI 安装（推荐）**

把下面这句话发给你的 AI：

```
请帮我安装 agents-workflow：
git clone --depth 1 https://github.com/TonyQ-AI/agents-workflow.git
然后把 skills/ 下的所有技能注册到技能系统。
配置 MCP 时用 npx -y tonyq-mimo-mcp-server，安装器会自动检测 MiMo 端点。
```

**方式二：手动安装**

```bash
git clone --depth 1 https://github.com/TonyQ-AI/agents-workflow.git
```

在工具配置中添加技能路径：

```toml
[skills]
paths = ["agents-workflow/skills"]
```
---

## 双模式说明

本工作流安装后**自动检测**平台能力，无需手动选择：

| 模式 | 条件 | 执行方式 |
|------|------|---------|
| **正常模式** | task 工具可用 | 编排器 → task(引擎 subagent) → task(子Agent) |
| **inline 模式** | task 不可用 | 编排器直接执行所有阶段 |

---

## 一键升级

已有旧版工作流的用户，直接说：

```
升级工作流
```

AI 会自动执行：
1. 拉取最新 40 个技能
2. 更新 MCP 配置（`mimo-mcp-server` → `tonyq-mimo-mcp-server`）
3. 修复环境变量（`MIMO_API_BASE` → `MIMO_API_URL`）
4. 清除 MCP 缓存
5. 更新 AGENTS.md

---

## 使用

```bash
# 标准全流程
/wf-orchestrator 项目名: 需求描述

# 轻量模式（跳过架构评审+扫描）
/wf-orchestrator 项目名: 需求描述 --lite

# 从指定阶段开始
/wf-orchestrator 项目名: 需求描述 --from coding

# 指定结束阶段
/wf-orchestrator 项目名: 需求描述 --to testing
```

---

## 系统要求

| 组件 | 要求 |
|------|------|
| AI 工具 | 支持 skill 调用的任意平台（自动适配 task 能力） |
| npx | Node.js >= 18，用于运行 `tonyq-mimo-mcp-server` |
| API Key | DeepSeek（工作流推理）+ MiMo（多模态，可选） |
| 网络 | 可访问 github.com 和 npmjs.com |

---

## 常见问题

### MCP 报 401 / 404
- 确认 `MIMO_API_URL` 完整：`https://api.xiaomimimo.com/v1/chat/completions`
- 确认 API key 在 `.env` 中正确
- 运行「升级工作流」自动修复配置

### npx 找不到包
- 确认使用 `tonyq-mimo-mcp-server` 而非 `mimo-mcp-server`
- 运行 `npx --yes tonyq-mimo-mcp-server` 测试

### 工具名不匹配
- 新工具名统一加 `mimo_` 前缀：`mimo_understand_image` 等
- 重启清除 MCP 缓存即可

### wf-orchestrator 没反应
- 确认 skills/ 下有 `wf-orchestrator` 和 `wf-orchestrator-engine`
- 确认 AGENTS.md 存在
- 确认 DeepSeek API key 有效
- **inline 模式无需 wf-orchestrator-engine**，编排器会直接执行

---

## 相关链接

- GitHub 仓库：https://github.com/TonyQ-AI/agents-workflow
- MiMo MCP 包：https://www.npmjs.com/package/tonyq-mimo-mcp-server
