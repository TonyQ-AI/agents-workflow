# Workflow Dualmode 安装指南

> 一键安装/升级多Agent协同开发工作流 · 双模式自适应 · 40 个技能

---

## 快速安装

在 AI 对话中输入：

```
装工作流
```

AI 会自动克隆本仓库（https://github.com/TonyQ-AI/agents-workflow）并完成全部安装。

或者直接说 `/agents-workflow` 也能触发安装。

---

## 手动安装

### 1. 克隆本仓库

```bash
git clone --depth 1 https://github.com/TonyQ-AI/agents-workflow.git
```

### 2. 配置技能路径

在工作流配置文件中添加：

```toml
[skills]
paths = ["agents-workflow/skills"]
```

### 3. 配置 MCP 服务器

```toml
[[plugins]]
name    = "mimo-multimodal"
command = "npx"
args    = ["-y", "tonyq-mimo-mcp-server"]
env     = {
  MIMO_API_URL  = "https://api.xiaomimimo.com/v1/chat/completions",
  MIMO_API_KEY  = "${MI*******KEY}"
}
call_timeout_seconds = 600
```

### 4. 配置 API 密钥

在 `.env` 中添加：

```env
DEEPSEEK_API_KEY=sk-xxx***xxxx    # 工作流推理（必填）
MIMO_API_KEY=sk-xxx***xxxx         # 多模态分析（推荐）
```

### 5. 添加 AGENTS.md

在项目根目录创建 `AGENTS.md`：

```markdown
## 进度检查规则
在任何操作开始之前，检查 docs/superpowers/plans/task_plan.md 是否存在。
如果存在，读取它以了解当前执行进度。
这一规则适用于所有对话，不论是否使用 wf-orchestrator 编排器。
```

### 6. 重启生效

重启工作流使新技能和 MCP 配置生效。

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
- 原版（task 专用）：https://github.com/TonyQ-AI/workflow-task
- MiMo MCP 包：https://www.npmjs.com/package/tonyq-mimo-mcp-server
