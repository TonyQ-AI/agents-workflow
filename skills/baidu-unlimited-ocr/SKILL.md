---
name: baidu-unlimited-ocr
description: >
  使用百度智能云 Unlimited-OCR 进行文档解析 — 一次性长视野文档解析，支持图片、PDF、表格、手写体。
  当用户需要从图片或 PDF 中提取文字、解析表格、识别文档内容时使用。
  也适用于：发票识别、身份证识别、营业执照识别、试卷分析等场景。
license: MIT
metadata:
  version: "1.0"
  category: ai-ocr-document
  sources:
    - https://cloud.baidu.com/doc/OCR/s/fmr1p39gb
---

# Baidu Unlimited-OCR

## Prerequisites

Requires `ocr_document` or `ocr_document_run` tool from `baidu-ocr-document` MCP server. If not available, configure the MCP server in reasonix.toml:

```toml
[[plugins]]
name    = "baidu-ocr-document"
type    = "http"
url     = "https://aip.baidubce.com/mcp/document/sse"
headers = { Authorization = "Bearer ${BAIDU_OCR_API_KEY}" }
```

## Supported Formats

- **图片格式：** JPEG, PNG, BMP, TIFF
- **文档格式：** PDF（多页自动转换）
- **文件上限：** 建议不超过 4MB（受 MCP 协议限制）

## Capabilities

基于百度智能云 Unlimited-OCR 模型（基于 DeepSeek-OCR 改进），支持：

| 功能 | 说明 |
|:----|:------|
| **文档解析** | 一次性长视野解析整页文档，保持版面结构 |
| **表格识别** | 自动识别表格结构，输出 HTML 表格 |
| **手写体识别** | 支持手写与印刷体混排场景 |
| **多页 PDF** | 自动将 PDF 转换为图片分批处理 |
| **版面分析** | 识别标题、段落、图表、表格等版面元素 |

## Usage

### Basic Document OCR

```
"解析这张图片中的文字：C:/document.png"
"提取这个PDF的内容：C:/report.pdf"
"识别这张发票上的信息：C:/invoice.jpg"
```

### Structured Recognition

百度 OCR 还支持大量结构化识别场景（需使用对应 MCP 工具）：

| 场景 | 工具 | 返回字段 |
|:----|:-----|:---------|
| 身份证 | `ocr_id_card` | 姓名、身份证号、地址、有效期等 |
| 营业执照 | `ocr_business_license` | 公司名、法人、信用代码、经营范围等 |
| 增值税发票 | `ocr_vat_invoice` | 发票号、金额、税额、购买方等 |
| 银行卡 | `ocr_bank_card` | 卡号、有效期、发卡行、卡片类型 |
| 车牌 | `ocr_license_plate` | 车牌号、地域编号 |

## Workflow Integration

在工作流中，百度 Unlimited-OCR 作为多模态 MCP 的一种补充，在以下场景调用：

- **📋 规划阶段** — 用户提供纸质文档/手写需求/PDF 规格说明时，提取文字内容
- **💻 编码阶段** — 需要从文档截图中提取表格数据时
- **🧪 测试阶段** — 验证生成的文档/报表与原始设计是否一致
- **🔍 审查阶段** — 审查文档类项目的输出质量

## Comparison with MiMo MCP

| 维度 | Baidu Unlimited-OCR (API) | MiMo MCP (本地模型) |
|:----|:-------------------------|:-------------------|
| 运行环境 | 无需 GPU，纯 API 调用 | 需 Node.js 运行时 |
| 核心能力 | 文档解析、OCR、结构化识别 | 通用多模态（图片/音频/视频/TTS） |
| 文档识别 | ⭐ 更优（专用 OCR 模型） | 通用能力 |
| 多模态 | 仅文字识别 | 图片/音频/视频/TTS 全支持 |
| 成本 | 按调用量计费 | 按 MiMo API Token 计费 |

## Notes

- 不支持 Base64 内联图片输入，请提供文件路径或 URL
- 请求总大小建议不超过 4MB
- 大文档建议拆分处理
- API Key 从百度智能云控制台获取：https://console.bce.baidu.com/iam/#/iam/apikey/list
- 更多 MCP 工具列表参考：https://cloud.baidu.com/doc/OCR/s/8mmk82q0j
