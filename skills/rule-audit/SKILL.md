---
name: rule-audit
description: "Rust 编程规范规则审计 - 按指定规范编号对项目执行定向检查"
---

# Rust 编程规范规则审计 (Rule Audit)

## 概述

本 Skill 指导 AI 助手对 Rust 项目执行**特定规则**的编程规范检查，用户可指定一个或多个规范编号进行定向审计。与 [`full-audit`](../full-audit/SKILL.md)（全量检测）不同，本 Skill 的核心是**路由查询**——根据用户指定的规范编号，在路由表中查找对应条目，然后分发到不同的处理路径：

- **Tool 组**：通过 `guidelines_runner` 工具链执行指定 lint 的静态检测
- **LLM 组**：通过大模型代码审查覆盖工具无法检测的规则

最终输出符合统一 schema 的 JSON 审计报告，`checkMode` 标记为 `"specific"`。

---

## 前置条件

执行本 Skill 前，请确认以下环境和工具已就绪：

1. **用户已提供规范编号** — 至少指定一个规范编号（如 `G.NAM.01`、`G.CMT.02` 等）

2. **`guidelines_runner` 工具链已注册**（如果涉及 Tool 组规则）
   ```bash
   # 验证工具链是否可用
   rustup toolchain list | grep guidelines_runner
   # 如未注册，需先执行（路径根据实际安装位置调整）：
   rustup toolchain link guidelines_runner /path/to/guidelines_runner
   ```

3. **rust-guidelines MCP Server 已配置**（推荐）或可通过 CLI 调用
   ```json
   {
     "mcpServers": {
       "rust-guidelines": {
         "command": "npx",
         "args": ["-y", "rust-guidelines-mcp-server"]
       }
     }
   }
   ```

4. **目标 Rust 项目路径已确认** — 需要是一个有效的 Cargo 项目（含 `Cargo.toml`）

---

## 资源文件

本 Skill 依赖以下资源文件，所有路径相对于 `skills/rule-audit/` 目录（通过符号链接指向 `full-audit/` 下的实际文件）：

| 文件 | 用途 |
|------|------|
| [`assets/tool_handle.json`](assets/tool_handle.json) | 37 条静态工具路由表，用于查找规范编号对应的 lint codes |
| [`assets/llm_handle.json`](assets/llm_handle.json) | 9 条 LLM 检测路由表，含 `pre_filter` 预过滤配置和 `batch_size` |
| [`assets/output_schema.json`](assets/output_schema.json) | 统一输出 JSON schema 定义 |
| [`refs/rust_coding_guidelines.md`](refs/rust_coding_guidelines.md) | 完整 Rust 编程规范参考文档 |
| [`refs/G.CMT.02.md`](refs/G.CMT.02.md) | 文件头注释应包含版权说明 |
| [`refs/G.FMT.01.md`](refs/G.FMT.01.md) | extern 外部函数应显式指定 ABI 标识 |
| [`refs/G.TYP.BOL.03.md`](refs/G.TYP.BOL.03.md) | && 和 \|\| 右侧不应包含副作用 |
| [`refs/G.TYP.SCT.01.md`](refs/G.TYP.SCT.01.md) | 外部使用的自定义类型宜实现常见 trait |
| [`refs/G.CTF.01.md`](refs/G.CTF.01.md) | 宜优先使用模式匹配而非判断后再取值 |
| [`refs/G.CTF.02.md`](refs/G.CTF.02.md) | Match Guard 中不应使用带副作用的条件表达式 |
| [`refs/G.MAC.DCL.01.md`](refs/G.MAC.DCL.01.md) | 宏匹配规则应遵从匹配范围从小至大 |
| [`refs/G.MAC.PRO.011.md`](refs/G.MAC.PRO.011.md) | 不应通过过程宏将 unsafe 包装为 safe |
| [`refs/G.SAF.MEM.04.md`](refs/G.SAF.MEM.04.md) | 敏感信息使用完毕后应立即清零 |

---

## 支持的规范编号

共 46 条可检测的规范编号，分为 Tool 组和 LLM 组：

### Tool 组（37 条）— 通过 `guidelines_runner` 静态检测

G.NAM.01, G.NAM.02, G.CMT.01, G.CNS.01, G.CNS.02, G.TYP.01, G.TYP.BOL.01, G.TYP.BOL.02,
G.TYP.CHR.01, G.TYP.INT.01, G.TYP.SLC.01, G.TYP.SLC.02, G.TYP.STR.01, G.TYP.PTR.01,
G.CTF.03, G.EXP.01, G.EXP.02, G.FUD.01, G.FUD.02, G.TRA.01, G.TRA.02, G.ERR.01, G.MOD.01,
G.SAF.UNS.01, G.SAF.UNS.02, G.SAF.FFI.01, G.SAF.FFI.02, G.SAF.FFI.03, G.SAF.FFI.04, G.SAF.FFI.05,
G.SAF.ASY.01, G.SAF.MTH.01, G.SAF.MEM.01, G.SAF.MEM.02, G.SAF.MEM.03,
G.MAC.01, G.OTH.01

### LLM 组（9 条）— 通过大模型代码审查

| # | 规范编号 | 标题 | ref 文件 |
|---|---------|------|---------|
| 1 | G.CMT.02 | 文件头注释应包含版权说明 | [`refs/G.CMT.02.md`](refs/G.CMT.02.md) |
| 2 | G.FMT.01 | extern 外部函数应显式指定 ABI 标识 | [`refs/G.FMT.01.md`](refs/G.FMT.01.md) |
| 3 | G.TYP.BOL.03 | && 和 \|\| 右侧不应包含副作用 | [`refs/G.TYP.BOL.03.md`](refs/G.TYP.BOL.03.md) |
| 4 | G.TYP.SCT.01 | 外部使用的自定义类型宜实现常见 trait | [`refs/G.TYP.SCT.01.md`](refs/G.TYP.SCT.01.md) |
| 5 | G.CTF.01 | 宜优先使用模式匹配而非判断后再取值 | [`refs/G.CTF.01.md`](refs/G.CTF.01.md) |
| 6 | G.CTF.02 | Match Guard 中不应使用带副作用的条件表达式 | [`refs/G.CTF.02.md`](refs/G.CTF.02.md) |
| 7 | G.MAC.DCL.01 | 宏匹配规则应遵从匹配范围从小至大 | [`refs/G.MAC.DCL.01.md`](refs/G.MAC.DCL.01.md) |
| 8 | G.MAC.PRO.011 | 不应通过过程宏将 unsafe 包装为 safe | [`refs/G.MAC.PRO.011.md`](refs/G.MAC.PRO.011.md) |
| 9 | G.SAF.MEM.04 | 敏感信息使用完毕后应立即清零 | [`refs/G.SAF.MEM.04.md`](refs/G.SAF.MEM.04.md) |

---

## 工作流

### 步骤 1: 接收用户输入与规则路由

#### 1.1 解析规范编号

接收用户指定的规范编号，支持以下输入格式：

- 单条规则：`G.CMT.02`
- 多条规则（逗号分隔）：`G.NAM.01, G.CTF.01, G.SAF.MEM.04`
- 多条规则（空格分隔）：`G.NAM.01 G.CTF.01 G.SAF.MEM.04`

解析后得到规范编号列表，如 `["G.NAM.01", "G.CTF.01", "G.SAF.MEM.04"]`。

#### 1.2 验证编号有效性

对每个规范编号，检查是否在上述 46 条支持列表中：

- 如果编号有效，继续处理
- 如果编号无效，向用户报告无效编号，并列出支持的编号供参考
- 如果部分有效部分无效，报告无效编号后继续处理有效编号

#### 1.3 路由分类

在 [`assets/tool_handle.json`](assets/tool_handle.json) 和 [`assets/llm_handle.json`](assets/llm_handle.json) 中查找每个规范编号对应的条目，分类为两组：

```
输入: ["G.NAM.01", "G.CTF.01", "G.SAF.MEM.04"]

路由结果:
  tool_group: ["G.NAM.01"]          → 在 tool_handle.json 中找到
  llm_group:  ["G.CTF.01", "G.SAF.MEM.04"]  → 在 llm_handle.json 中找到
```

#### 1.4 确认目标项目路径

- 向用户确认待审计的 Rust 项目根目录（包含 `Cargo.toml` 的目录）
- 记录项目绝对路径，后续用于填充报告的 `projectPath` 字段

---

### 步骤 2: Tool 组检测

> **仅当 tool_group 非空时执行此步骤。**

与 [`full-audit`](../full-audit/SKILL.md) 的 Tool 层检测不同，rule-audit 需要**仅启用用户指定规则对应的 lint codes**，而非全量检测。

#### 2.1 提取目标 lint codes

从 [`assets/tool_handle.json`](assets/tool_handle.json) 中查找 tool_group 中每条规则对应的 `lint_codes`：

```
示例：用户指定 G.NAM.01
查询 tool_handle.json → 找到:
  guideline_code: "G.NAM.01"
  lint_codes: ["non_camel_case_types", "non_snake_case", "non_upper_case_globals"]

合并所有 tool_group 规则的 lint_codes，得到目标 lint 列表。
```

#### 2.2 执行指定 lint 检测

**方式一：MCP Server 调用（优先）**

通过 MCP 调用 `rust-guidelines` 服务的 `check_project` 工具：

```
工具名: check_project
参数: { "project_path": "<目标项目绝对路径>" }
```

> 注意：如果 MCP Server 支持 lint 过滤参数，应传入仅需检测的 lint codes。

**方式二：CLI 回退**

如果 MCP 不可用，使用命令行方式，通过 `-W` 参数仅启用指定的 lint codes：

```bash
cd <目标项目路径> && cargo +guidelines_runner clippy --message-format=json -- \
  -W clippy::non_camel_case_types \
  -W clippy::non_snake_case \
  -W clippy::non_upper_case_globals \
  2>&1
```

**CLI lint 参数构建规则：**

- 对于 `clippy::` 前缀的 lint code，直接使用 `-W clippy::xxx`
- 对于 rustc 原生 lint code（如 `non_snake_case`），使用 `-W xxx`
- 多个 lint code 用多个 `-W` 参数拼接

#### 2.3 解析工具输出

工具输出为 JSON Lines 格式。解析步骤：

1. **逐行解析** JSON 输出，筛选 `reason` 为 `"compiler-message"` 的条目
2. **提取诊断信息**：`level`、`code.code`、`message`、`spans[0].file_name`、`spans[0].line_start`、`spans[0].column_start`、`rendered`
3. **过滤无效条目**：跳过 `level` 为 `"note"` 且无 `code` 的纯提示信息
4. **过滤非目标 lint**：仅保留 lint code 在目标 lint 列表中的诊断条目

#### 2.4 映射 guideline_code

加载 [`assets/tool_handle.json`](assets/tool_handle.json)，构建 lint code → guideline_code 的反查映射表，对每条诊断设置 `guideline_code` 和 `source: "tool"`。

#### 2.5 构建 Tool 诊断列表

将每条诊断转换为统一 schema 格式：

```json
{
  "level": "warning",
  "code": "non_snake_case",
  "guideline_code": "G.NAM.01",
  "source": "tool",
  "message": "variable `myVar` should have a snake case name",
  "file": "src/lib.rs",
  "line": 15,
  "column": 9,
  "rendered": "warning: variable `myVar` should have a snake case name\n  --> src/lib.rs:15:9",
  "suggestions": []
}
```

---

### 步骤 3: LLM 组检测

> **仅当 llm_group 非空时执行此步骤。**

复用 [`full-audit`](../full-audit/SKILL.md) 的 **"预过滤 → 分批审查 → 进度追踪"** 三阶段策略，但仅针对用户指定的 LLM 规则执行。加载 [`assets/llm_handle.json`](assets/llm_handle.json) 获取 llm_group 中每条规则的完整配置。

#### 3.1 文件收集与检查矩阵

扫描项目全部 `.rs` 源文件（排除 `target/` 目录），初始化检查矩阵，仅包含用户指定的 LLM 规则列：

```
示例：用户指定了 G.CTF.01 和 G.SAF.MEM.04

                 G.CTF.01  G.SAF.MEM.04
src/main.rs        ⬜          ⬜
src/lib.rs         ⬜          ⬜
...

⬜ = 待检查  ✅ = 已检查  ➖ = 无需审查（预过滤无匹配）
```

#### 3.2 预过滤

对 llm_group 中的每条规则，按其 `pre_filter` 配置进行预过滤：

| pre_filter 类型 | 适用规则 | 操作 |
|----------------|---------|------|
| `head` | G.CMT.02 | 提取每个文件前 N 行 |
| `grep` | G.FMT.01, G.TYP.BOL.03, G.TYP.SCT.01, G.CTF.01, G.CTF.02, G.MAC.DCL.01, G.SAF.MEM.04 | 用正则搜索匹配文件，提取匹配行及上下文 |
| `crate_type` | G.MAC.PRO.011 | 仅保留 proc-macro crate 的源文件 |

不匹配的文件直接标记为 ➖（无需审查）。详细的预过滤模式参见 [`full-audit`](../full-audit/SKILL.md) 步骤 3.2。

#### 3.3 分批审查

对每条 LLM 规则，将预过滤后的代码片段分批送入 LLM 审查：

- 每批文件数由规则的 `batch_size` 字段控制（5~20 个文件不等）
- 每批**只聚焦一条规则**，附带对应的 `refs/G.XXX.md` 参考文件
- 每批总代码量建议 < 8000 tokens

**单批审查的 Prompt 结构：**

```markdown
[System] 你是 Rust 编程规范审查员。请严格根据以下规范要求审查代码，
只报告确实违反规范的问题，不要报告误报。

[规范参考]
{加载对应的 refs/G.XXX.md 文件内容}

[待审查代码]
--- 文件: src/foo.rs (行 1-50) ---
{预过滤后的代码片段}

--- 文件: src/bar.rs (行 23-67) ---
{预过滤后的代码片段}

[输出要求]
请按以下 JSON 数组格式输出发现的问题（如无问题则输出空数组 []）：
[
  {
    "level": "warning",
    "code": "<guideline_code>",
    "guideline_code": "<guideline_code>",
    "source": "llm",
    "message": "<问题描述>",
    "file": "<文件相对路径>",
    "line": <行号>,
    "column": <列号，不确定时填 1>,
    "suggestions": [{ "replacement": "<修复代码>", "applicability": "HasPlaceholders" }]
  }
]
```

#### 3.4 进度追踪

1. **逐批标记**：每批审查完成后，标记对应 `文件 × 规则` 为 ✅
2. **无匹配标记**：预过滤无匹配的文件标记为 ➖
3. **完成验证**：每条规则的所有文件都标记为 ✅ 或 ➖ 后，该规则才算完成
4. **进度报告**：
   ```
   [进度] G.CTF.01 检测完成: 45/45 文件已覆盖 (12 已检查, 33 无需审查), 发现 2 个问题
   ```

---

### 步骤 4: 结果合并与输出

#### 4.1 合并诊断列表

将步骤 2（Tool 组）和步骤 3（LLM 组）产生的所有诊断条目合并为统一的 `diagnostics` 数组。

#### 4.2 计算 summary 统计

```json
{
  "total": "<diagnostics 总数>",
  "errors": "<error 条目数>",
  "warnings": "<warning 条目数>",
  "tool_diagnostics": "<tool 来源条目数>",
  "llm_diagnostics": "<llm 来源条目数>"
}
```

#### 4.3 汇总 checkedRules

收集本次检测涉及的所有 `guideline_code`（去重），仅包含用户指定的规范编号（与 full-audit 列出全部 46 条不同）。

#### 4.4 生成最终报告

输出符合 [`assets/output_schema.json`](assets/output_schema.json) 的统一 JSON 报告，`checkMode` 设为 `"specific"`。

---

## 使用示例

### 示例 1: 检查单条 Tool 规则

**用户输入：** "请检查项目是否符合 G.NAM.01（统一命名风格）"

**执行流程：**
1. 解析规范编号：`["G.NAM.01"]`
2. 路由查询：在 [`assets/tool_handle.json`](assets/tool_handle.json) 中找到 → tool_group
3. 提取 lint_codes：`["non_camel_case_types", "non_snake_case", "non_upper_case_globals"]`
4. 执行检测：
   ```bash
   cd <项目路径> && cargo +guidelines_runner clippy --message-format=json -- \
     -W non_camel_case_types -W non_snake_case -W non_upper_case_globals 2>&1
   ```
5. 解析输出 → 映射 guideline_code → 生成报告

### 示例 2: 检查单条 LLM 规则

**用户输入：** "请检查项目是否符合 G.CMT.02（文件头版权说明）"

**执行流程：**
1. 解析规范编号：`["G.CMT.02"]`
2. 路由查询：在 [`assets/llm_handle.json`](assets/llm_handle.json) 中找到 → llm_group
3. 获取配置：`pre_filter.type = "head"`, `pre_filter.lines = 10`, `batch_size = 20`
4. 文件收集 → 提取每个文件前 10 行
5. 分批审查：每批 20 个文件头部，加载 [`refs/G.CMT.02.md`](refs/G.CMT.02.md) 作为参考
6. 合并结果 → 生成报告

### 示例 3: 混合检查（Tool + LLM）

**用户输入：** "请检查 G.NAM.01 和 G.CMT.02"

**执行流程：**
1. 解析规范编号：`["G.NAM.01", "G.CMT.02"]`
2. 路由分类：
   - tool_group: `["G.NAM.01"]` → 走 Tool 检测路径
   - llm_group: `["G.CMT.02"]` → 走 LLM 检测路径
3. 并行执行两条路径
4. 合并两组结果 → 生成统一报告

### 示例 4: 批量检查多条 LLM 规则

**用户输入：** "请检查 G.CTF.01, G.CTF.02, G.SAF.MEM.04"

**执行流程：**
1. 解析规范编号：`["G.CTF.01", "G.CTF.02", "G.SAF.MEM.04"]`
2. 路由分类：全部在 llm_group
3. 逐条执行 LLM 检测（各自独立的预过滤 + 分批审查）
4. 合并三条规则的诊断结果 → 生成报告

---

## 输出格式

最终报告为 JSON 格式，`checkMode` 为 `"specific"`，`checkedRules` 仅包含用户指定的规范编号：

```json
{
  "projectPath": "/path/to/rust-project",
  "checkMode": "specific",
  "checkedRules": ["G.NAM.01", "G.CMT.02"],
  "summary": {
    "total": 5,
    "errors": 0,
    "warnings": 5,
    "tool_diagnostics": 3,
    "llm_diagnostics": 2
  },
  "diagnostics": [
    {
      "level": "warning",
      "code": "non_snake_case",
      "guideline_code": "G.NAM.01",
      "source": "tool",
      "message": "variable `myVar` should have a snake case name",
      "file": "src/lib.rs",
      "line": 15,
      "column": 9,
      "rendered": "warning: variable `myVar` should have a snake case name\n  --> src/lib.rs:15:9",
      "suggestions": []
    },
    {
      "level": "warning",
      "code": "G.CMT.02",
      "guideline_code": "G.CMT.02",
      "source": "llm",
      "message": "文件头部缺少版权声明注释",
      "file": "src/main.rs",
      "line": 1,
      "column": 1,
      "suggestions": [
        {
          "replacement": "// Copyright (c) 2024 YourCompany. All rights reserved.\n// SPDX-License-Identifier: MIT",
          "applicability": "HasPlaceholders"
        }
      ]
    }
  ]
}
```

---

## 注意事项

1. **路由查询是核心** — rule-audit 的核心逻辑是根据用户指定的规范编号，在 [`assets/tool_handle.json`](assets/tool_handle.json) 和 [`assets/llm_handle.json`](assets/llm_handle.json) 中查找对应条目，然后分发到不同的处理路径。如果编号在两个路由表中都找不到，应提示用户该编号不在支持范围内。

2. **Tool 组需精确控制 lint** — 与 full-audit 不同，rule-audit 的 Tool 组需要**仅启用用户指定规则对应的 lint codes**，而非全量检测。通过 CLI 的 `-W` 参数或 MCP 的 lint 过滤参数实现精确控制。

3. **LLM 组复用三阶段策略** — LLM 组的检测流程（预过滤 → 分批审查 → 进度追踪）与 [`full-audit`](../full-audit/SKILL.md) 完全一致，但仅针对用户指定的 LLM 规则执行，不需要跑完全部 9 条。

4. **分批审查时保持规则聚焦** — 每批只检查一条规则，附带该规则的 ref 参考文件。不要在一批中混合多条规则，以避免 LLM 注意力分散导致漏检。

5. **LLM 诊断的 `code` 字段** — 对于 LLM 组产生的诊断，`code` 字段直接使用 `guideline_code`（如 `G.CMT.02`），因为没有对应的 lint code。

6. **`checkMode` 必须为 `"specific"`** — 区别于 full-audit 的 `"full"` 模式，rule-audit 的输出报告中 `checkMode` 固定为 `"specific"`。

7. **`checkedRules` 仅含指定编号** — 与 full-audit 列出全部 46 条不同，rule-audit 的 `checkedRules` 仅包含用户本次指定的规范编号。

8. **无效编号处理** — 如果用户指定的编号不在 46 条支持列表中，应明确告知用户，并列出可用的编号供参考。部分有效时，继续处理有效编号。

9. **仅一组有内容时的处理** — 如果路由后 tool_group 为空（用户仅指定了 LLM 规则），则跳过步骤 2；如果 llm_group 为空（用户仅指定了 Tool 规则），则跳过步骤 3。

10. **排除 `target/` 目录** — 扫描 `.rs` 文件时，必须排除 `target/` 目录下的编译产物。

11. **MCP 与 CLI 的选择** — 优先使用 MCP Server 调用 `check_project` 工具。仅当 MCP 不可用时，才回退到 CLI 方式。两种方式的输出格式相同。

12. **报告输出位置** — 最终 JSON 报告可以直接输出到终端，也可以写入文件（如 `guidelines-report.json`），根据用户需求决定。
