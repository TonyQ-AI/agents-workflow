# Reasonix Workflow — 完整技术手册 v4.1.2

> 版本：4.1.2 · 最后更新：2026-07-13

---

## 目录

1. [架构概览](#1-架构概览)
2. [双引擎设计](#2-双引擎设计)
3. [完整执行流程](#3-完整执行流程)
4. [硬门控机制](#4-硬门控机制)
5. [兜底与降级](#5-兜底与降级)
6. [进度跟踪与断点续跑](#6-进度跟踪与断点续跑)
7. [原始需求交叉验证](#7-原始需求交叉验证)
8. [知识沉淀](#8-知识沉淀)
9. [子Agent调度](#9-子agent调度)
10. [MCP 多模态集成](#10-mcp-多模态集成)
11. [技能目录](#11-技能目录)
12. [v4.x 版本演进](#12-v4x-版本演进)

---

## 1. 架构概览

### 1.1 一句话描述

用户输入一句话需求，编排器自动串起从需求澄清到版本登记的全流程。

### 1.2 整体架构

```
用户: /wf-orchestrator MediaManager: 新增批量导出

编排器 (wf-orchestrator) — 主会话
  ├── 解析参数 + 创建会话目录
  ├── 交互式需求澄清（与用户一问一答）
  ├── 产出: task_plan.md + specs/*.md
  └── 打包 JSON -> task(wf-orchestrator-engine)

执行引擎 (wf-orchestrator-engine) — 独立上下文
  ├── 接收 JSON 包裹
  ├── 启动自检（显示项目/模式/范围）
  ├── 12项任务依次执行
  └── 返回 JSON 摘要给编排器
```

### 1.3 v4.x 核心设计决策

| 设计决策 | 原因 |
|:---------|:-----|
| 双引擎拆分 | 编排器交互产生大量上下文，引擎独立会话不受干扰 |
| 任务清单替代阶段编号 | 消除计数矛盾。AI不数"第几个"，只做"下一项" |
| 硬门控 + 回退上限3次 | 架构问题必须修，但不能无限循环 |
| fallback兜底 | task()不可用时引擎直执行，工作流不中断 |
| 原文交叉验证 | 每层子Agent对照用户确认的specs校验，防需求漂移 |

---

## 2. 双引擎设计

### 2.1 为什么拆分（v3.x -> v4.0）

v3.x 单引擎存在三个问题：

1. **上下文膨胀** — 规划阶段50轮问答后，执行阶段还要背这段历史
2. **子Agent混乱** — 子Agent可能翻到前面对话，误解当前任务
3. **恢复困难** — 中断后要从头回溯整个对话历史

### 2.2 引擎1：编排器

四步操作：

- **步骤1** — 参数解析、创建会话目录、检测中断恢复
- **步骤2** — 交互式需求澄清（brainstorming逐轮问答），产出 task_plan.md + specs
- **步骤3** — 打包JSON -> task(wf-orchestrator-engine)
- **步骤4** — 接收引擎返回，展示结果

### 2.3 引擎2：执行引擎

12项任务清单：

1. 领域建模  2. 架构设计  3. 架构评审(硬门控)  4. 编码
5. 测试  6. 架构扫描(硬门控)  7. 审查  8. 部署
9. 文档检查  10. 分支收尾  11. 版本登记  12. 总结+知识沉淀

启动时自检输出关键信息（项目/模式/范围），fallback时显式告知用户。

### 2.4 v4.0 -> v4.1 主要演进

| | v4.0 | v4.1 |
|:--|:-----|:-----|
| 描述格式 | "阶段 4/13：架构评审" | ">>> 架构评审" |
| 计数方式 | 编号（常出错） | 任务清单（不数数） |
| 评分标准 | 35/60 | 18/24（统一） |
| --to | 声明未实现 | 每个任务检查 |
| --parallel | 声明未实现 | 任务4/5并行调度 |
| --timeout | 硬编码300 | 参数化 |
| 回退上限 | 无 | 最多3次 |
| fallback | 硬编码false | 失败2次自动降级 |

---

## 3. 完整执行流程

### 3.1 正常模式（task可用）

编排器完成交互式规划后，将 JSON 包裹传给执行引擎，引擎在独立上下文中按清单执行：

```
启动自检 -> 显示项目/模式/范围 -> 逐项执行
```

**任务1: 领域建模** — 引擎直执行
- 读取 specs + task_plan
- 注入 reasonix-domain-modeling 方法论
- 提取实体、定义关系、建立通用语言
- 产出: domain-model.md
- 交叉验证: 概念是否与specs一致

**任务2: 架构设计** — task(wf-architect) Pro模型
- 传入 specs + domain-model + task_plan + 项目根目录
- 如为回退重试，注入上次评审意见
- 产出: 02-design.md

**任务3: 架构评审(硬门控)** — 引擎直执行
- 注入 reasonix-arch-review，6维评分(满分24)
- Go(>=18): 产出 03-arch-review.md，进任务4
- No-Go(<18): 回退任务2，最多3次，超限强制Go

**任务4: 编码** — task(wf-developer) Flash模型
- 启动前验证: 03-arch-review.md 必须存在且为Go
- 传入 specs + design + task_plan + 项目根目录
- --parallel时同时启动任务5
- 注入 TDD/系统化调试/UI设计(按需)
- 产出: 代码 + CHANGES.md

**任务5: 测试** — task(wf-tester) Flash模型
- --parallel时等待已在任务4启动的测试完成
- 注入 dispatching-parallel-agents + verification
- 产出: TEST_REPORT.md

**任务6: 架构扫描(硬门控)** — 引擎直执行
- 扫描代码结构，检测循环依赖、模块耦合
- 无严重问题: 产出 architecture-scan.html，进任务7
- 有严重问题: 回退任务4，最多3次

**任务7: 审查** — task(wf-reviewer) Pro模型
- 启动前验证: architecture-scan.html 必须存在
- 注入代码审查 + 中文审查 + 反馈处理
- 产出: 05-review.md

**任务8: 部署** — task(wf-deployer) Flash模型
- 检测项目类型，执行构建命令
- 产出: DEPLOY.md

**任务9: 文档检查** — 引擎直执行
- 注入 chinese-documentation
- 检查排版、术语、标点
- OCR处理图片型文档
- 产出: doc-check-report.md

**任务10: 分支收尾** — 引擎直执行
- git add + chinese-commit-conventions 格式提交
- 询问是否推送/创建PR
- 注入 finishing-a-development-branch + git-worktrees

**任务11: 版本登记** — 引擎直执行
- 注入 vt，更新 .vt.json

**任务12: 生成总结** — 引擎直执行
- 读取所有产出摘要，写入 SUMMARY.md
- 知识沉淀: 提炼关键发现写入 KNOWLEDGE.md
- 返回 JSON 摘要给编排器

### 3.2 硬门控介入

任务3（架构评审）No-Go -> 回退任务2（最多3次）
任务6（架构扫描）严重问题 -> 回退任务4（最多3次）
任务4启动前检查03-arch-review.md必须Go
任务7启动前检查architecture-scan.html必须存在

### 3.3 兜底模式

fallback=true -> 所有task()替换为引擎直执行，质量略降但不中断

### 3.4 --lite 轻量模式

跳过任务3（架构评审）+ 任务6（架构扫描），适合小型修复

### 3.5 --parallel 并行

任务4/5同时启动 wf-developer + wf-tester

### 3.6 各任务产出

| 任务 | 产出文件 | 执行者 |
|:-----|:--------|:------|
| 1 领域建模 | domain-model.md | 引擎 |
| 2 架构设计 | 02-design.md | wf-architect |
| 3 架构评审 | 03-arch-review.md | 引擎 |
| 4 编码 | 代码 + CHANGES.md | wf-developer |
| 5 测试 | TEST_REPORT.md | wf-tester |
| 6 架构扫描 | architecture-scan.html | 引擎 |
| 7 审查 | 05-review.md | wf-reviewer |
| 8 部署 | DEPLOY.md | wf-deployer |
| 9 文档检查 | doc-check-report.md | 引擎 |
| 10 分支收尾 | git commit + PR | 引擎 |
| 11 版本登记 | .vt.json | 引擎 |
| 12 总结 | SUMMARY.md | 引擎 |

---

## 4. 硬门控机制

### 4.1 架构评审门控（任务3）

6维评分，每维1-4分，满分24。Go: >=18/24 (75%)

| 维度 | 说明 |
|:-----|:-----|
| 功能完整性 | 是否覆盖所有需求点 |
| 可扩展性 | 架构是否支持未来扩展 |
| 性能 | 是否有性能考量 |
| 安全性 | 安全设计 |
| 可维护性 | 清晰、可测试 |
| 成本 | 技术选型合理 |

No-Go时回退任务2（最多3次，超限强制Go+记录风险）

### 4.2 架构扫描门控（任务6）

检测循环依赖、模块耦合、架构腐化。严重问题回退任务4（最多3次）

### 4.3 双重验证

- 任务4启动前: 必须03-arch-review.md存在且为Go
- 任务7启动前: 必须architecture-scan.html存在
- 不满足则回退到对应修复任务

---

## 5. 兜底与降级

### 触发条件
1. 编排器检测task()可用性
2. task()连续失败2次自动降级

### 启动告知
```
正常: 执行模式: 正常模式（task派发子Agent）
兜底: 执行模式: 兜底模式（task不可用，引擎直执行）
      当前环境不支持task()子Agent调用，已切换为兜底模式
```

工具: ZCode/Reasonix 有task()，其他自动fallback

---

## 6. 进度跟踪与断点续跑

三文件: task_plan.md + progress.md + findings.md
加 checkpoint.json 记录断点

--from 恢复时验证前置文件存在
--to 命中跳任务12
阶段名映射: planning > domain_modeling > architecture > arch_review > coding > testing > arch_scan > review > deploy > doc_check > branch_finish > version_tracking

progress标签映射表确保中文标签与英文checkpoint值对应

---

## 7. 原始需求交叉验证

每个子Agent收到两份输入: 前序衍生文档(承上) + 用户确认的specs(校验)
偏离时显式标注。防需求层层传递中漂移

---

## 8. 知识沉淀

每次会话结束提炼 -> 写入 KNOWLEDGE.md -> 下次启动自动注入
格式: 关键决策/经验教训/领域知识/注意事项/数据洞察

---

## 9. 子Agent调度

5个task子Agent: wf-architect/wf-developer/wf-tester/wf-reviewer/wf-deployer
Pro模型(架构/审查) Flash模型(编码/测试/部署)
调用格式: task(subagent_name, prompt, model, timeout)
产品验证: 1重试2标记失败3继续

---

## 10. MCP 多模态集成

### 10.1 当前配置

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

**MiMo 工具清单（7个）：**

| MCP 工具名 | 对应技能 | 用途 |
|:-----------|:---------|:-----|
| `mimo_understand_image` | mimo-image-understanding | 图片分析/OCR/UI审查 |
| `mimo_understand_audio` | mimo-audio-understanding | 音频转写/理解 |
| `mimo_understand_video` | mimo-video-understanding | 视频内容分析 |
| `mimo_speech_recognition` | — | 语音识别(ASR) |
| `mimo_speech_synthesis` | mimo-tts | 预置音色语音合成 |
| `mimo_voice_design` | mimo-tts | 文本描述自定义音色 |
| `mimo_voice_clone` | mimo-tts | 音频样本克隆音色 |

### 10.2 v4.1.2 迁移说明

**为什么从 `mimo-mcp-server` 换成 `tonyq-mimo-mcp-server`？**

原 `mimo-mcp-server` 由第三方 alionsss 发布。npm 发布版（v0.1.2）的工具名加了 `mimo_` 前缀（如 `mimo_understand_image`），但 GitHub 源码未同步，仍然用旧名（`understand_image`）。导致 npx 拉到的版本和缓存文件不一致，MCP 连接断开。

解决方案：fork 到 `TonyQ-AI/mimo-mcp-server`，修复源码对齐 npm 版，以 `tonyq-mimo-mcp-server` 名称发布到 npm，完全脱离对 alionsss 的依赖。

**为什么 `MIMO_API_BASE` 改成 `MIMO_API_URL`？**

MCP Server 源码通过 `process.env.MIMO_API_URL` 读取 API 地址，默认值是 Token Plan 端点（`token-plan-cn.xiaomimimo.com`）。如果用标准 API key 但配的是 `MIMO_API_BASE`，服务器读不到自定义地址，默认走 Token Plan 端点导致 401。

**为什么工具名加了 `mimo_` 前缀？**

npm 发布版 v0.1.2 将所有工具名加了 `mimo_` 前缀（`understand_image` → `mimo_understand_image`），但 Reasonix 缓存文件仍用旧名。调用时名字不匹配，MCP Server 返回 `read: EOF`。修复方式：
1. 更新缓存文件中 7 个工具名匹配新名
2. 重新发布 npm 包时保持工具名一致

**为什么配置文件不能直接写 key？**

Reasonix 的 `redact_tool_output` 安全机制会自动将显示中的密钥替换为 `*`。如果直接复制显示中的 key（带星号）写入配置文件，实际传入的是带星号的无效 key。必须通过环境变量引用（`${MI*******KEY}`）或使用脚本从 `.env` 文件读取写入。

---

## 11. 技能目录

40技能: 编排2 + 子Agent5 + 参考1 + 架构4 + 方法7 + 审查4 + 提交5 + 辅助3 + MCP5 + 扩展4

---

## 12. v4.x 版本演进

### v4.1.2 (2026-07-13) — MCP 独立发布

**MCP 服务迁移**
- 自建 npm 包 `tonyq-mimo-mcp-server`，脱离对 alionsss 的依赖
- fork 源码到 TonyQ-AI，修复工具名对齐、构建 dist、bin 名统一
- GitHUb 源码仓库：https://github.com/TonyQ-AI/mimo-mcp-server

**环境变量修fux**
- `MIMO_API_BASE` → `MIMO_API_URL`（MCP Server 实际读的是后者）
- URL 补全 `/chat/completions` 路径（之前缺路径导致 404）
- 配置文件禁止硬编码 key，改用 `${MIMO_API_KEY}` 环境变量引用（防止遮罩写错）

**npx 参数修复**
- bin 名从 `mimo-mcp-server` 改为 `tonyq-mimo-mcp-server`，与包名一致，npx 可直接找到入口
- 配置示例更新为 `args = ["-y", "tonyq-mimo-mcp-server"]`（无需 `--` 分隔符）

**一键安装/升级**
- reasonix-workflow 安装器（`/reasonix-workflow`）新增升级模式
- 说「升级工作流」自动拉最新技能 + 修复 MCP 配置 + 清除缓存

### v4.0 (2026-07-12) — 双引擎架构

- 编排器+引擎拆分
- 硬门控(Go/No-Go + 扫描回退)
- 启动自检
- task_plan.md统一基准

### v3.x (2026-07-07~12) — 13阶段扩展

- 8->13阶段、交互澄清、领域建模、交叉验证、知识沉淀

### v2.x (2026-07-01~06) — 基础流水线

- 8阶段、TDD/调试、checkpoint追踪

---

> 版本: 4.1.2 · 最后更新: 2026-07-13
