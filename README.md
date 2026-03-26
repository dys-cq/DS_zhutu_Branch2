# DS_zhutu_Branch2

调用 Coze 工作流处理产品文档，并默认输出**工作流原始 JSON 结果**。

这个技能适合用于：
- 读取 DOCX / PDF / TXT 产品资料
- 将 `Product_Information` 和 `Product_name` 传给 Coze workflow
- 获取卖点审核结果
- 在审核未通过时保留工作流返回的**修正后的最终内容**

---

## 目录结构

```text
DS_zhutu_Branch2/
├─ scripts/
│  └─ call_coze_workflow.py
├─ SKILL.md
└─ README.md
```

---

## 功能特点

- 支持读取以下文件格式：
  - `.docx`
  - `.pdf`
  - `.txt`
- 支持手工传入 `Product_name`
- 支持从文件名自动识别产品名
- Coze 请求超时已提高到 `120s`
- 默认输出为**工作流原始 JSON**
- 显式加 `--json` 时输出解析后的扩展 JSON
- 审核状态支持三类识别：
  - `passed`
  - `corrected_passed`
  - `failed`
- 兼容问题字段：
  - `问题描述`
  - `问题内容`

---

## 输入参数

工作流会接收两个核心参数：

- `Product_Information`
- `Product_name`

其中：
- `Product_Information` 来自输入文档提取出的全文内容
- `Product_name` 优先级如下：
  1. `--product-name` 手工传入
  2. 从文件名自动识别
  3. 默认值 `酷洛菲黑松露润浸洁面乳`

### 对话中使用示例

```text
文档路径：C:\...\xxx.docx
产品名称：酷洛菲黑松露润浸洁面乳
```

也可以直接把文档**拖放到对话框**，再补一句：

```text
产品名称：酷洛菲黑松露润浸洁面乳
```

---

## 用法

### 1）默认用法（输出工作流原始 JSON）

```bash
uv run python scripts/call_coze_workflow.py "C:\path\to\product.docx" \
  --api-token "sat_xxx"
```

### 2）手工指定产品名

```bash
uv run python scripts/call_coze_workflow.py "C:\path\to\product.docx" \
  --product-name "酷洛菲黑松露润浸洁面乳" \
  --api-token "sat_xxx"
```

### 3）保存结果到文件

```bash
uv run python scripts/call_coze_workflow.py "C:\path\to\product.docx" \
  --product-name "酷洛菲黑松露润浸洁面乳" \
  --api-token "sat_xxx" \
  -o "C:\path\to\result.json"
```

### 4）输出解析后的扩展 JSON

```bash
uv run python scripts/call_coze_workflow.py "C:\path\to\product.docx" \
  --product-name "酷洛菲黑松露润浸洁面乳" \
  --api-token "sat_xxx" \
  --json
```

### 5）只生成成分报告

```bash
uv run python scripts/call_coze_workflow.py "C:\path\to\product.docx" \
  --api-token "sat_xxx" \
  --ingredients
```

---

## 输出说明

### 默认输出

默认返回 **工作流原始 JSON**，尽量保持 Coze 原输出结构不变。

常见字段示例：

```json
{
  "修正后的最终内容": "...",
  "是否通过审查": "是",
  "问题清单": []
}
```

### `--json` 输出

`--json` 返回的是脚本解析后的扩展 JSON，通常包含：

```json
{
  "content": "...",
  "sections": {},
  "review_status": "是",
  "normalized_review_status": "passed",
  "issues": [],
  "execute_id": "...",
  "usage": {},
  "debug_url": "...",
  "raw_workflow_output": {}
}
```

---

## 审核逻辑

### 审核通过
返回工作流最终内容。

### 修正后通过
返回工作流修正后的最终内容，并标记为 `corrected_passed`。

### 审核未通过
仍然保留并输出工作流原始返回结果；如果工作流提供了 `修正后的最终内容`，该内容也会保留在 JSON 中。

---

## 依赖

建议使用 `uv` 运行。

依赖包括：
- `requests`
- `python-docx`（可选，用于 DOCX 备用解析）
- `pdfplumber`（用于 PDF）

安装示例：

```bash
uv pip install requests python-docx pdfplumber
```

---

## 环境变量

推荐通过环境变量传入 Coze Token：

```env
COZE_API_TOKEN=sat_xxx
```

也可以直接使用命令行参数：

```bash
--api-token "sat_xxx"
```

---

## 主要脚本

- `scripts/call_coze_workflow.py`：主调用脚本

---

## 当前状态

当前版本已支持：
- 手工传入 `Product_name`
- 默认输出原始 JSON
- 审核状态三类归一化
- 120 秒超时
- 审核未通过时保留修正后的最终内容

如果后续要继续增强，推荐方向：
- 优化文件名识别产品名
- 增加更严格的结构化校验
- 清理历史临时测试脚本
