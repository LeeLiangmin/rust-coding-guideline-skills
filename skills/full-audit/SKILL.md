---
name: full-audit
description: "Rust 编程规范全量审计 - 对项目执行 46 条规范的完整检查（37 条静态工具 + 9 条 LLM 辅助）"
---

# Rust 编程规范全量审计 (Full Audit)

## 概述

本 Skill 指导 AI 助手对 Rust 项目执行**完整的编程规范检查**，覆盖 46 条规则类条目（37 条 Tool 层 + 9 条 LLM 层）。检测采用"静态工具 + LLM 辅助"双层架构：

- **Tool 层**（37 条）：通过 `guidelines_runner` 工具链执行全量静态 lint 检测
- **LLM 层**（9 条）：通过大模型代码审查覆盖工具无法检测的规则

最终输出符合统一 schema 的 JSON 审计报告，`checkMode` 标记为 `"full"`。

---

## 前置条件

执行本 Skill 前，请确认以下环境和工具已就绪：

1. **`guidelines_runner` 工具链已注册**
   ```bash
   # 验证工具链是否可用
   rustup toolchain list | grep guidelines_runner
   # 如未注册，需先执行（路径根据实际安装位置调整）：
   rustup toolchain link guidelines_runner /path/to/guidelines_runner
   ```

2. **rust-guidelines MCP Server 已配置**（推荐）或可通过 CLI 调用
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

3. **目标 Rust 项目路径已确认** — 需要是一个有效的 Cargo 项目（含 `Cargo.toml`）

---

## 资源文件

本 Skill 依赖以下资源文件，所有路径相对于 `skills/full-audit/` 目录：

| 文件 | 用途 |
|------|------|
| [`assets/tool_handle.json`](assets/tool_handle.json) | 37 条静态工具路由表，用于将 lint code 映射回 guideline_code |
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

## 工作流

### 步骤 1: 环境准备

1. **确认目标项目路径**
   - 向用户确认待审计的 Rust 项目根目录（包含 `Cargo.toml` 的目录）
   - 记录项目绝对路径，后续用于填充报告的 `projectPath` 字段

2. **验证工具链**
   ```bash
   rustup toolchain list | grep guidelines_runner
   ```
   - 如果输出中包含 `guidelines_runner`，工具链可用，继续下一步
   - 如果未找到，提示用户先注册工具链：
     ```bash
     rustup toolchain link guidelines_runner /path/to/guidelines_runner
     ```

3. **检查 MCP Server 可用性**
   - 尝试通过 MCP 调用 `rust-guidelines` 服务的 `check_project` 工具
   - 如果 MCP 不可用，记录需使用 CLI 回退方案

---

### 步骤 2: Tool 层检测（37 条规则）

> **核心原则：** `guidelines_runner` 会自动执行所有已注册的 lint 规则，无需查询 `tool_handle.json` 来控制检测范围。`tool_handle.json` 仅在结果处理阶段用于 lint code → guideline_code 的映射。

#### 2.1 执行静态检测

**方式一：MCP Server 调用（优先）**

通过 MCP 调用 `rust-guidelines` 服务的 `check_project` 工具：

```
工具名: check_project
参数: { "project_path": "<目标项目绝对路径>" }
```

**方式二：CLI 回退**

如果 MCP 不可用，使用命令行方式：

```bash
cd <目标项目路径> && cargo +guidelines_runner clippy --message-format=json 2>&1
```

#### 2.2 解析工具输出

工具输出为 JSON Lines 格式（每行一个 JSON 对象）。解析步骤：

1. **逐行解析** JSON 输出，筛选 `reason` 为 `"compiler-message"` 的条目
2. **提取诊断信息**，从每条 `message` 中获取：
   - `level` — 诊断级别（`error` / `warning` / `note`）
   - `code.code` — lint 代码（如 `clippy::wrong_self_convention`）
   - `message` — 诊断消息文本
   - `spans[0].file_name` — 文件路径
   - `spans[0].line_start` — 起始行号
   - `spans[0].column_start` — 起始列号
   - `rendered` — 格式化的完整诊断输出
3. **过滤无效条目**：跳过 `level` 为 `"note"` 且无 `code` 的纯提示信息

#### 2.3 映射 guideline_code

加载 [`assets/tool_handle.json`](assets/tool_handle.json)，构建 lint code → guideline_code 的反查映射表：

```
映射逻辑：
对于 tool_handle.json 中的每条规则 rule:
  对于 rule.lint_codes 中的每个 lint_code:
    映射表[lint_code] = rule.guideline_code
```

对每条解析出的诊断条目：
- 用诊断的 `code` 字段在映射表中查找对应的 `guideline_code`
- 设置 `source` 为 `"tool"`
- 如果 lint code 不在映射表中（可能是非规范相关的通用 lint），`guideline_code` 留空

#### 2.4 构建 Tool 诊断列表

将每条诊断转换为统一 schema 格式：

```json
{
  "level": "warning",
  "code": "clippy::wrong_self_convention",
  "guideline_code": "G.NAM.02",
  "source": "tool",
  "message": "methods called `to_*` usually take self by reference",
  "file": "src/lib.rs",
  "line": 42,
  "column": 5,
  "rendered": "warning: methods called `to_*` usually take...",
  "suggestions": []
}
```

---

### 步骤 3: LLM 层检测（9 条规则）

> **核心策略：** 采用 **"预过滤 → 分批审查 → 进度追踪"** 三阶段策略，确保全覆盖且高效。

加载 [`assets/llm_handle.json`](assets/llm_handle.json) 获取 9 条 LLM 规则的完整配置。

#### 3.1 文件收集

1. **扫描项目**获取全部 `.rs` 源文件列表：
   ```bash
   find <项目路径> -name "*.rs" -not -path "*/target/*"
   ```
   > Windows 环境下可使用：`dir /s /b <项目路径>\*.rs | findstr /v "\\target\\"`

2. **初始化检查矩阵** — 维护 `文件 × 规则` 的检查状态：

   ```
                 G.CMT.02  G.FMT.01  G.TYP.BOL.03  G.TYP.SCT.01  G.CTF.01  G.CTF.02  G.MAC.DCL.01  G.MAC.PRO.011  G.SAF.MEM.04
   src/main.rs      ⬜        ⬜          ⬜             ⬜           ⬜        ⬜          ⬜             ⬜              ⬜
   src/lib.rs       ⬜        ⬜          ⬜             ⬜           ⬜        ⬜          ⬜             ⬜              ⬜
   ...
   
   ⬜ = 待检查  ✅ = 已检查  ➖ = 无需审查（预过滤无匹配）
   ```

#### 3.2 预过滤

对每条 LLM 规则，按其 `pre_filter` 配置进行预过滤，缩小待审查范围：

##### `head` 类型预过滤

适用规则：**G.CMT.02**（文件头版权检查）

```
操作：提取每个 .rs 文件的前 N 行（由 pre_filter.lines 指定）
结果：所有文件都需要检查，但每个文件只送入头部片段
示例：pre_filter.lines = 10 → 提取每个文件前 10 行
```

##### `grep` 类型预过滤

适用规则：**G.FMT.01**, **G.TYP.BOL.03**, **G.TYP.SCT.01**, **G.CTF.01**, **G.CTF.02**, **G.MAC.DCL.01**, **G.SAF.MEM.04**

```
操作：用 pre_filter.patterns 中的正则表达式搜索文件
结果：
  - 匹配的文件：提取匹配行及上下文（pre_filter.context_lines 行）
  - 不匹配的文件：标记为 ➖（无需审查）
```

各规则的预过滤模式：

| 规则 | 搜索模式 | 上下文行数 | 预期过滤率 |
|------|---------|-----------|-----------|
| G.FMT.01 | `extern\s+fn`, `extern\s*\{` | 5 | ~95% |
| G.TYP.BOL.03 | `&&`, `\|\|` | 10 | ~80% |
| G.TYP.SCT.01 | `pub\s+struct\s+`, `pub\s+enum\s+` | 20 | ~85% |
| G.CTF.01 | `is_some()`, `is_none()`, `.unwrap()`, `is_empty()` | 10 | ~85% |
| G.CTF.02 | `match\s+` | 30 | ~90% |
| G.MAC.DCL.01 | `macro_rules!` | 50 | ~95% |
| G.SAF.MEM.04 | `password`, `secret`, `key`, `token`, `credential`, `private_key`, `passphrase` | 20 | ~90% |

##### `crate_type` 类型预过滤

适用规则：**G.MAC.PRO.011**（过程宏 unsafe 包装检测）

```
操作：检查项目中各 crate 的 Cargo.toml，筛选 proc-macro 类型的 crate
结果：
  - proc-macro crate 的源文件：需要审查
  - 非 proc-macro crate 的源文件：标记为 ➖（无需审查）
判断方式：Cargo.toml 中包含 [lib] 段且 proc-macro = true
```

#### 3.3 分批审查

对每条 LLM 规则，将预过滤后的待审查文件/代码片段分批送入 LLM：

**分批策略：**

- 每批文件数由规则的 `batch_size` 字段控制（5~20 个文件不等）
- 每批**只聚焦一条规则**，避免多规则混淆
- 每批总代码量控制在合理范围内（建议 < 8000 tokens 代码内容）
- 如果单个文件代码片段过长，可进一步拆分

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
    "code": "<规则的 guideline_code>",
    "guideline_code": "<规则的 guideline_code>",
    "source": "llm",
    "message": "<问题描述>",
    "file": "<文件相对路径>",
    "line": <行号>,
    "column": <列号，不确定时填 1>,
    "suggestions": [
      {
        "replacement": "<建议的修复代码>",
        "applicability": "HasPlaceholders"
      }
    ]
  }
]
```

**9 条规则的审查要点：**

| # | 规则 | ref 文件 | batch_size | 审查聚焦点 |
|---|------|---------|------------|-----------|
| 1 | G.CMT.02 | [`refs/G.CMT.02.md`](refs/G.CMT.02.md) | 20 | 文件头部是否包含版权声明注释 |
| 2 | G.FMT.01 | [`refs/G.FMT.01.md`](refs/G.FMT.01.md) | 10 | extern 函数是否显式指定 ABI |
| 3 | G.TYP.BOL.03 | [`refs/G.TYP.BOL.03.md`](refs/G.TYP.BOL.03.md) | 8 | && 和 \|\| 右侧是否有副作用 |
| 4 | G.TYP.SCT.01 | [`refs/G.TYP.SCT.01.md`](refs/G.TYP.SCT.01.md) | 8 | pub struct/enum 是否实现常见 trait |
| 5 | G.CTF.01 | [`refs/G.CTF.01.md`](refs/G.CTF.01.md) | 8 | 是否可用模式匹配替代判断+取值 |
| 6 | G.CTF.02 | [`refs/G.CTF.02.md`](refs/G.CTF.02.md) | 5 | match guard 中是否有副作用 |
| 7 | G.MAC.DCL.01 | [`refs/G.MAC.DCL.01.md`](refs/G.MAC.DCL.01.md) | 5 | macro_rules! 匹配分支顺序 |
| 8 | G.MAC.PRO.011 | [`refs/G.MAC.PRO.011.md`](refs/G.MAC.PRO.011.md) | 5 | 过程宏是否将 unsafe 包装为 safe |
| 9 | G.SAF.MEM.04 | [`refs/G.SAF.MEM.04.md`](refs/G.SAF.MEM.04.md) | 5 | 敏感信息是否使用后清零 |

#### 3.4 进度追踪

为确保全覆盖，严格执行以下进度追踪机制：

1. **逐批标记**：每批审查完成后，将该批涉及的 `文件 × 规则` 组合标记为 ✅（已检查）
2. **无匹配标记**：预过滤阶段无匹配内容的文件，直接标记为 ➖（无需审查）
3. **完成验证**：每条规则的所有相关文件都标记为 ✅ 或 ➖ 后，该规则才算检测完成
4. **全局完成**：所有 9 条规则都检测完成后，LLM 层检测才算结束

**进度报告格式（每条规则完成后输出）：**

```
[进度] G.CMT.02 检测完成: 45/45 文件已覆盖 (40 已检查, 5 无需审查), 发现 3 个问题
[进度] G.FMT.01 检测完成: 45/45 文件已覆盖 (8 已检查, 37 无需审查), 发现 1 个问题
...
```

---

### 步骤 4: 结果合并与输出

#### 4.1 合并诊断列表

将步骤 2（Tool 层）和步骤 3（LLM 层）产生的所有诊断条目合并为一个统一的 `diagnostics` 数组。

#### 4.2 计算 summary 统计

```json
{
  "total": "<diagnostics 数组总长度>",
  "errors": "<level 为 error 的条目数>",
  "warnings": "<level 为 warning 的条目数>",
  "tool_diagnostics": "<source 为 tool 的条目数>",
  "llm_diagnostics": "<source 为 llm 的条目数>"
}
```

#### 4.3 汇总 checkedRules

收集本次检测涉及的所有 `guideline_code`（去重），包括：
- Tool 层：从 [`assets/tool_handle.json`](assets/tool_handle.json) 中提取全部 37 条规则的 `guideline_code`
- LLM 层：从 [`assets/llm_handle.json`](assets/llm_handle.json) 中提取全部 9 条规则的 `guideline_code`

#### 4.4 生成最终报告

输出符合 [`assets/output_schema.json`](assets/output_schema.json) 的统一 JSON 报告。

---

## 输出格式

最终报告为 JSON 格式，结构如下：

```json
{
  "projectPath": "/path/to/rust-project",
  "checkMode": "full",
  "checkedRules": [
    "G.NAM.01", "G.NAM.02", "G.CMT.01", "G.CMT.02",
    "G.CNS.01", "G.CNS.02", "G.TYP.BOL.01", "G.TYP.BOL.02",
    "G.TYP.BOL.03", "G.TYP.INT.01", "G.TYP.SLC.01", "G.TYP.SLC.02",
    "G.TYP.STR.01", "G.TYP.PTR.01", "G.TYP.SCT.01", "G.TYP.01",
    "G.TYP.CHR.01", "G.EXP.01", "G.EXP.02", "G.FUD.01", "G.FUD.02",
    "G.FMT.01", "G.TRA.01", "G.TRA.02", "G.ERR.01", "G.MOD.01",
    "G.CTF.01", "G.CTF.02", "G.CTF.03", "G.MAC.01", "G.MAC.DCL.01",
    "G.MAC.PRO.011", "G.SAF.MTH.01", "G.SAF.UNS.01", "G.SAF.UNS.02",
    "G.SAF.FFI.01", "G.SAF.FFI.02", "G.SAF.FFI.03", "G.SAF.FFI.04",
    "G.SAF.FFI.05", "G.SAF.ASY.01", "G.SAF.MEM.01", "G.SAF.MEM.02",
    "G.SAF.MEM.03", "G.SAF.MEM.04", "G.OTH.01"
  ],
  "summary": {
    "total": 15,
    "errors": 3,
    "warnings": 12,
    "tool_diagnostics": 11,
    "llm_diagnostics": 4
  },
  "diagnostics": [
    {
      "level": "warning",
      "code": "clippy::wrong_self_convention",
      "guideline_code": "G.NAM.02",
      "source": "tool",
      "message": "methods called `to_*` usually take self by reference",
      "file": "src/lib.rs",
      "line": 42,
      "column": 5,
      "rendered": "warning: methods called `to_*` usually take self by reference\n  --> src/lib.rs:42:5",
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

1. **Tool 层检测是全量执行的** — `guidelines_runner` 会自动运行所有已注册的 lint，不需要逐条指定。`tool_handle.json` 仅用于结果阶段的 lint code → guideline_code 映射，不用于控制检测范围。

2. **LLM 层检测必须确保全覆盖** — 通过检查矩阵和进度追踪机制，确保每个 `.rs` 文件在每条 LLM 规则下都被处理（检查或标记为无需审查）。不要遗漏任何文件。

3. **预过滤是效率优化手段** — 预过滤的目的是减少送入 LLM 的无关代码量，但不应因此遗漏文件。对于 `head` 类型的预过滤（如 G.CMT.02），所有文件都需要检查。

4. **分批审查时保持规则聚焦** — 每批只检查一条规则，附带该规则的 ref 参考文件。不要在一批中混合多条规则，以避免 LLM 注意力分散导致漏检。

5. **LLM 诊断的 `code` 字段** — 对于 LLM 层产生的诊断，`code` 字段直接使用 `guideline_code`（如 `G.CMT.02`），因为没有对应的 lint code。

6. **处理大型项目** — 如果项目文件数量很大（> 100 个 `.rs` 文件），可以适当增加 `batch_size` 或在预过滤阶段更积极地过滤，但必须确保全覆盖。

7. **错误处理** — 如果 Tool 层检测失败（如工具链未安装），应记录错误并继续执行 LLM 层检测，最终报告中注明 Tool 层检测未完成。

8. **排除 `target/` 目录** — 扫描 `.rs` 文件时，必须排除 `target/` 目录下的编译产物。

9. **MCP 与 CLI 的选择** — 优先使用 MCP Server 调用 `check_project` 工具。仅当 MCP 不可用时，才回退到 CLI 方式 `cargo +guidelines_runner clippy --message-format=json`。两种方式的输出格式相同。

10. **报告输出位置** — 最终 JSON 报告可以直接输出到终端，也可以写入文件（如 `guidelines-report.json`），根据用户需求决定。
