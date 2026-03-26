---
name: coding-guidelines-audit
description: "Rust 编程规范审计 - 支持全量检查和定向规则检查，具备断点续审能力"
---

# Rust 编程规范审计

## 概述

本 Skill 指导 AI 对 Rust 项目执行编程规范检查，覆盖 46 条规范规则（37 条工具层 + 9 条 LLM 层）以及所有 clippy 默认 lint。

检测采用**四分类架构**，按检查来源和规范映射关系分类：

| source 分类 | 含义 | 有 guideline_code | 检查方式 | 需要 -W 启用 |
|------------|------|-------------------|---------|-------------|
| `clippy_default` | clippy 默认 lint，无规范映射 | ❌ 留空 | 工具自动检出 | ❌ |
| `clippy_native` | clippy 原生 lint，有规范映射 | ✅ | 工具自动检出 | ❌ |
| `clippy_custom` | 扩展工具自定义 lint，有规范映射 | ✅ | 工具检出，需 -W 启用 | ✅ |
| `llm` | 工具无法检查，LLM 审查 | ✅ | LLM 代码审查 | — |

**两种模式**（自动判断）：
- **full**：全量检测所有规则 + 所有 clippy 默认 lint（用户未提供规则编号时）
- **specific**：定向检查用户指定的规则编号（用户提供了规则编号时）

**默认检查范围**（full 模式）：
1. clippy 默认启用的所有 lint（`clippy_default` + `clippy_native`）
2. clippy 扩展自定义 lint（`clippy_custom`，通过 `-W` 参数启用）
3. LLM 代码审查（`llm`，覆盖工具无法检测的 9 条规则）

中间产物持久化到目标项目的 `.guidelines-audit/` 目录，支持断点续审。

## 资源文件

所有路径相对于 `skills/coding-guidelines-audit/` 目录：

| 文件 | 用途 |
|------|------|
| [`assets/guidelines_registry.json`](assets/guidelines_registry.json) | 46 条规范统一注册表，含 check_type 和 lint_codes 映射 |
| [`assets/llm_check_config.json`](assets/llm_check_config.json) | 9 条 LLM 检查配置，含 pre_filter 和 batch_size |
| [`assets/output_schema.json`](assets/output_schema.json) | 统一输出 JSON schema（source 四分类 + by_source 统计） |
| [`refs/`](refs/) 目录 | 9 个 LLM 规则参考文件 + 完整规范文档 |

## 前置条件

1. **guidelines_runner 工具链已注册**
   ```bash
   rustup toolchain list | grep guidelines_runner
   # 如未注册：rustup toolchain link guidelines_runner /path/to/guidelines_runner
   ```

2. **rust-guidelines MCP Server 已配置**（推荐，CLI 可回退）
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

3. **目标 Rust 项目路径已确认**（含 `Cargo.toml` 的有效 Cargo 项目）

## 工作流

### 步骤 0: 断点检测

检查目标项目中 `.guidelines-audit/state.json` 是否存在：

- **不存在** → 进入步骤 1（全新审计）
- **存在且所有步骤已 completed** → 询问用户是否重新审计
- **存在且有未完成步骤**（steps 中有非 completed 状态）：
  1. 向用户展示上次审计的进度摘要（模式、已完成步骤、中断位置）
  2. 询问：继续上次审计 or 重新开始？
  3. 如果继续 → 跳转到对应的未完成步骤
  4. 如果重新开始 → 清空 `.guidelines-audit/` 目录，进入步骤 1

⚠️ 断点续审时，步骤 3 从 `llm-progress.json` 恢复，跳过已完成的规则和文件

### 步骤 1: 模式判断与初始化

**模式判断逻辑：**
- 用户提供了规则编号（如 G.NAM.01, G.CMT.02）→ **specific** 模式
- 用户未提供规则编号 → **full** 模式

**specific 模式额外步骤：**
1. 解析规则编号（支持逗号或空格分隔）
2. 在 [`assets/guidelines_registry.json`](assets/guidelines_registry.json) 中查找每个编号
3. 按 `check_type` 分组：
   - `clippy_native` / `clippy_custom` → 加入 `tool_group`
   - `llm` → 加入 `llm_group`
4. 无效编号报告给用户，继续处理有效编号

**full 模式：**
- `tool_group` = 全部 37 条工具规则（21 条 `clippy_native` + 16 条 `clippy_custom`）
- `llm_group` = 全部 9 条 LLM 规则
- 同时包含所有 clippy 默认 lint（`clippy_default`）

**通用步骤：**
1. 确认目标项目路径（含 `Cargo.toml`）
2. 验证 guidelines_runner 工具链：`rustup toolchain list | grep guidelines_runner`
3. 检查 MCP Server 可用性，不可用则记录使用 CLI 回退

📝 产物：创建 `.guidelines-audit/` 目录，写入 `state.json`（step1_init = completed）

### 步骤 2: Clippy 检测（工具层）

> 仅当 tool_group 非空时执行（full 模式下始终执行）

#### 2.1 执行静态检测

**MCP 优先**：调用 rust-guidelines 服务的 `check_project` 工具
```
工具名: check_project
参数: { "project_path": "<目标项目绝对路径>" }
```

**CLI 回退**：

⚠️ `clippy_custom` 类规则（guidelines_runner 自定义 lint）默认未启用，必须通过 `-W` 参数显式开启。

```bash
# full 模式：全量执行（需显式启用所有 clippy_custom lint）
# 从 guidelines_registry.json 中提取所有 check_type="clippy_custom" 的 lint_codes，
# 拼接为 -W 参数列表
cd <项目路径> && cargo +guidelines_runner clippy --message-format=json -- \
  -W clippy::unconstrained_numeric_literal \
  -W clippy::invalid_char_range \
  -W clippy::infinite_loop \
  -W clippy::untrusted_lib_loading \
  -W clippy::passing_string_to_c_functions \
  -W clippy::extern_without_repr \
  -W clippy::non_reentrant_functions \
  -W clippy::mem_unsafe_functions \
  -W clippy::blocking_op_in_async \
  -W clippy::fallible_memory_allocation \
  -W clippy::dangling_ptr_dereference \
  -W clippy::null_ptr_dereference \
  -W clippy::return_stack_address \
  -W clippy::ptr_double_free \
  -W clippy::unsafe_block_in_proc_macro \
  -W clippy::hard_coded_ip \
  -W clippy::implicit_abi \
  -W clippy::mut_from_ref \
  2>&1

# specific 模式：仅启用 tool_group 中的 lint codes
# clippy_native 类无需 -W（默认启用），clippy_custom 类需要 -W
cd <项目路径> && cargo +guidelines_runner clippy --message-format=json -- \
  -W clippy::<custom_lint_1> \
  -W clippy::<custom_lint_2> \
  2>&1
```

> 💡 **动态生成 -W 参数**：实际执行时，应从 [`assets/guidelines_registry.json`](assets/guidelines_registry.json) 中读取所有 `check_type="clippy_custom"` 规则的 `lint_codes`，动态拼接 `-W` 参数列表，而非硬编码。上方示例仅供参考。

#### 2.2 输出解析

逐行解析 JSON 输出：
1. 筛选 `reason` 为 `"compiler-message"` 的条目
2. 提取基础字段：`level`, `code.code`, `message`, `spans[0].file_name`, `spans[0].line_start`, `spans[0].column_start`, `rendered`
3. **提取 suggestions**（关键）：clippy 的修复建议分布在两个位置，需要都检查：
   - **主 spans**：遍历 `message.spans[]`，若 `suggested_replacement` 非 null，提取：
     ```json
     {
       "file": "spans[].file_name",
       "line_start": "spans[].line_start",
       "line_end": "spans[].line_end",
       "column_start": "spans[].column_start",
       "column_end": "spans[].column_end",
       "replacement": "spans[].suggested_replacement",
       "applicability": "spans[].suggestion_applicability"
     }
     ```
   - **子诊断 children**：遍历 `message.children[]`，对每个 child 的 `spans[]` 同样检查 `suggested_replacement` 非 null 的条目，按相同格式提取
   - 将所有提取到的 suggestion 合并为 `suggestions` 数组
   - ⚠️ 如果 `suggestions` 为空数组，仍保留该字段为 `[]`，不要省略
4. 过滤：跳过 `level` 为 `"note"` 且无 `code` 的纯提示

#### 2.3 source 分类与 guideline_code 映射

加载 [`assets/guidelines_registry.json`](assets/guidelines_registry.json)，构建 `lint_code → {guideline_code, check_type}` 反向映射表：

```
对每条诊断的 code 字段:
  在映射表中找到且 check_type = clippy_native →
    source = "clippy_native", guideline_code = 映射值
  在映射表中找到且 check_type = clippy_custom →
    source = "clippy_custom", guideline_code = 映射值
  未在映射表中 →
    source = "clippy_default", guideline_code = ""（留空）
```

⚠️ **所有诊断都保留**：
- `clippy_native`：有规范映射的 clippy 原生 lint
- `clippy_custom`：有规范映射的扩展自定义 lint
- `clippy_default`：无规范映射的 clippy 默认 lint（guideline_code 留空）

> **specific 模式特殊处理**：仅保留 `tool_group` 中规范对应的 lint 诊断，丢弃 `clippy_default` 类诊断（因为用户只关心指定的规范）。

📝 产物：写入 `.guidelines-audit/tool-results.json`，更新 `state.json`（step2_tool = completed）

### 步骤 3: LLM 层检测

> 仅当 llm_group 非空时执行（full 模式下始终执行全部 9 条）

加载 [`assets/guidelines_registry.json`](assets/guidelines_registry.json) 筛选 `check_type = "llm"` 的规则，然后从 [`assets/llm_check_config.json`](assets/llm_check_config.json) 获取对应的检查配置。

#### 3.1 文件收集

扫描项目全部 `.rs` 源文件（排除 `target/` 目录）：

**优先使用工具**：通过 `list_files` 等工具递归列出项目目录下所有 `.rs` 文件，过滤掉 `target/` 目录下的文件。

**CLI 回退**（按平台选择）：
```bash
# Windows (PowerShell)
Get-ChildItem -Path <项目路径> -Filter *.rs -Recurse | Where-Object { $_.FullName -notmatch '\\target\\' } | Select-Object -ExpandProperty FullName

# Linux/Mac
find <项目路径> -name "*.rs" -not -path "*/target/*"
```

#### 3.2 预过滤

对每条 LLM 规则，按 [`assets/llm_check_config.json`](assets/llm_check_config.json) 中的 `pre_filter` 配置执行预过滤：

| pre_filter 类型 | 操作 | 适用规则示例 |
|----------------|------|-------------|
| `head` | 提取每个文件前 N 行 | G.CMT.02（前 10 行） |
| `grep` | 用正则搜索匹配文件，提取匹配行及上下文 | G.FMT.01, G.CTF.01 等 |
| `crate_type` | 仅保留指定 crate 类型的源文件 | G.MAC.PRO.011（proc-macro） |

- 无匹配的文件标记为 `skipped`
- `grep` 类型：用 `pre_filter.patterns` 搜索，提取匹配行 ± `context_lines` 行上下文
- `crate_type` 类型：检查 `Cargo.toml` 中 `[lib]` 段的 `proc-macro = true`

#### 3.3 分批审查

对每条规则，将预过滤后的代码片段分批送入 LLM：
- 每批文件数由规则的 `batch_size` 字段控制
- ⚠️ 每批只聚焦一条规则，不混合多条规则
- 每批附带对应的 [`refs/G.XXX.md`](refs/) 参考文件

**Prompt 模板：**

```
[System] 你是 Rust 编程规范审查员。请严格根据以下规范要求审查代码，只报告确实违反规范的问题，不要报告误报。

[规范参考]
{加载对应的 refs/G.XXX.md 文件内容}

[待审查代码]
--- 文件: {file_path} (行 {start}-{end}) ---
{代码片段}

[输出要求]
按以下 JSON 数组格式输出（无问题则输出 []）：
[{
  "level": "warning",
  "code": "{guideline_code}",
  "guideline_code": "{guideline_code}",
  "source": "llm",
  "message": "<问题描述>",
  "file": "<文件路径>",
  "line": <行号>,
  "column": <列号>,
  "suggestions": [{"file": "<文件路径>", "line_start": <起始行>, "line_end": <结束行>, "column_start": <起始列>, "column_end": <结束列>, "replacement": "<修复代码>", "applicability": "HasPlaceholders"}]
}]
```

#### 3.4 进度追踪

每批审查完成后：
1. 更新 `.guidelines-audit/llm-progress.json`，标记该批文件为 `checked`
2. 每条规则所有文件都标记为 `checked` 或 `skipped` 后，该规则才算完成
3. 输出进度报告：
   ```
   [进度] G.CMT.02 检测完成: 45/45 文件已覆盖 (40 checked, 5 skipped), 发现 3 个问题
   ```

📝 产物：每批完成后更新 `llm-progress.json`，全部完成后更新 `state.json`（step3_llm = completed）

⚠️ 断点续审时从 `llm-progress.json` 恢复，跳过已完成的规则和文件

### 步骤 4: 结果合并与输出

1. **读取中间产物**：
   - 从 `.guidelines-audit/tool-results.json` 读取工具层诊断
   - 从 `.guidelines-audit/llm-progress.json` 读取 LLM 层诊断（汇总各规则的 diagnostics）

2. **合并**为统一 `diagnostics` 数组

3. **计算 summary**：
   ```json
   {
     "total": "<diagnostics 总数>",
     "errors": "<level 为 error 的数量>",
     "warnings": "<level 为 warning 的数量>",
     "by_source": {
       "clippy_default": "N",
       "clippy_native": "N",
       "clippy_custom": "N",
       "llm": "N"
     }
   }
   ```

4. **汇总 checkedRules**：
   - full 模式：全部 46 条规范编号
   - specific 模式：仅用户指定的编号

5. **设置 checkMode**：`"full"` 或 `"specific"`

📝 产物：写入 `.guidelines-audit/report.json`，更新 `state.json`（step4_merge = completed）

⚠️ 最终报告必须从中间产物文件合并生成，不依赖 LLM 记忆

输出报告文件路径给用户：`<项目路径>/.guidelines-audit/report.json`

## 关键约束

1. **排除 target/**：扫描 `.rs` 文件时必须排除 `target/` 目录下的编译产物
2. **MCP 优先，CLI 回退**：两种方式输出格式相同，优先使用 MCP Server
3. **工具层容错**：工具层失败时记录错误并继续 LLM 层检测
4. **产物驱动**：最终报告从中间产物文件合并生成，不依赖 LLM 记忆
5. **`.guidelines-audit/` 目录应加入 `.gitignore`**
6. **四分类一致性**：所有诊断的 `source` 字段必须使用四分类枚举值之一
