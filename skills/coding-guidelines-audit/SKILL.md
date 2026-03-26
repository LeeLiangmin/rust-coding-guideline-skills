---
name: coding-guidelines-audit
description: "Rust 编程规范审计 - 支持全量检查和定向规则检查，具备断点续审能力"
---

# Rust 编程规范审计

## 概述

本 Skill 指导 AI 对 Rust 项目执行编程规范检查，覆盖 46 条规范规则（37 条 Tool 层 + 9 条 LLM 层）以及标准 clippy lint。

检测采用**双层架构**：
- **Tool 层**：通过 `guidelines_runner` 工具链执行静态 lint 检测
- **LLM 层**：通过大模型代码审查覆盖工具无法检测的规则

**两种模式**（自动判断）：
- **full**：全量检测所有 46 条规则（用户未提供规则编号时）
- **specific**：定向检查用户指定的规则编号（用户提供了规则编号时）

**三种诊断来源**（source）：
- `tool_native`：clippy 原生 lint（含有 guideline_code 映射的和标准 clippy lint）
- `tool_custom`：fork 自定义 lint
- `llm`：LLM 代码审查

中间产物持久化到目标项目的 `.guidelines-audit/` 目录，支持断点续审。

## 资源文件

所有路径相对于 `skills/coding-guidelines-audit/` 目录：

| 文件 | 用途 |
|------|------|
| [`assets/tool_handle.json`](assets/tool_handle.json) | 37 条静态工具路由表，lint_code → guideline_code 映射 |
| [`assets/llm_handle.json`](assets/llm_handle.json) | 9 条 LLM 检测路由表，含 pre_filter 和 batch_size 配置 |
| [`assets/output_schema.json`](assets/output_schema.json) | 统一输出 JSON schema（source 三分类 + by_source 统计） |
| [`refs/`](refs/) 目录 | 9 个规则参考文件（G.CMT.02 ~ G.SAF.MEM.04）+ 完整规范文档 |

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
2. 在 [`assets/tool_handle.json`](assets/tool_handle.json) 和 [`assets/llm_handle.json`](assets/llm_handle.json) 中查找每个编号
3. 分为 `tool_group` 和 `llm_group`
4. 无效编号报告给用户，继续处理有效编号

**full 模式：**
- `tool_group` = 全部 37 条 Tool 规则
- `llm_group` = 全部 9 条 LLM 规则

**通用步骤：**
1. 确认目标项目路径（含 `Cargo.toml`）
2. 验证 guidelines_runner 工具链：`rustup toolchain list | grep guidelines_runner`
3. 检查 MCP Server 可用性，不可用则记录使用 CLI 回退

📝 产物：创建 `.guidelines-audit/` 目录，写入 `state.json`（step1_init = completed）

### 步骤 2: Tool 层检测

> 仅当 tool_group 非空时执行（full 模式下始终执行）

#### 2.1 执行静态检测

**MCP 优先**：调用 rust-guidelines 服务的 `check_project` 工具
```
工具名: check_project
参数: { "project_path": "<目标项目绝对路径>" }
```

**CLI 回退**：
```bash
# full 模式：全量执行
cd <项目路径> && cargo +guidelines_runner clippy --message-format=json 2>&1

# specific 模式：通过 -W 参数仅启用指定 lint codes
cd <项目路径> && cargo +guidelines_runner clippy --message-format=json -- \
  -W clippy::wrong_self_convention \
  -W non_snake_case \
  2>&1
```

#### 2.2 输出解析

逐行解析 JSON 输出：
1. 筛选 `reason` 为 `"compiler-message"` 的条目
2. 提取：`level`, `code.code`, `message`, `spans[0].file_name`, `spans[0].line_start`, `spans[0].column_start`, `rendered`
3. 过滤：跳过 `level` 为 `"note"` 且无 `code` 的纯提示

#### 2.3 guideline_code 和 source 映射

加载 [`assets/tool_handle.json`](assets/tool_handle.json)，构建 `lint_code → {guideline_code, source}` 映射表：

```
对每条诊断的 code 字段:
  在映射表中找到 → guideline_code = 映射值
                    source = clippy_native → "tool_native" / fork_custom → "tool_custom"
  未在映射表中   → guideline_code = ""（留空）
                    source = "tool_native"
```

⚠️ 所有诊断都保留，包括未映射的标准 clippy lint（guideline_code 留空，source = tool_native）

📝 产物：写入 `.guidelines-audit/tool-results.json`，更新 `state.json`（step2_tool = completed）

### 步骤 3: LLM 层检测

> 仅当 llm_group 非空时执行（full 模式下始终执行全部 9 条）

加载 [`assets/llm_handle.json`](assets/llm_handle.json) 获取规则配置。

#### 3.1 文件收集

扫描项目全部 `.rs` 源文件（排除 `target/` 目录）：
```bash
# Windows
dir /s /b <项目路径>\*.rs | findstr /v "\\target\\"
# Linux/Mac
find <项目路径> -name "*.rs" -not -path "*/target/*"
```

#### 3.2 预过滤

对每条 LLM 规则，按 [`assets/llm_handle.json`](assets/llm_handle.json) 中的 `pre_filter` 配置执行预过滤：

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
  "suggestions": [{"replacement": "<修复代码>", "applicability": "HasPlaceholders"}]
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
   - 从 `.guidelines-audit/tool-results.json` 读取 Tool 层诊断
   - 从 `.guidelines-audit/llm-progress.json` 读取 LLM 层诊断（汇总各规则的 diagnostics）

2. **合并**为统一 `diagnostics` 数组

3. **计算 summary**：
   ```json
   {
     "total": "<diagnostics 总数>",
     "errors": "<level 为 error 的数量>",
     "warnings": "<level 为 warning 的数量>",
     "by_source": { "tool_native": "N", "tool_custom": "N", "llm": "N" }
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
3. **Tool 层容错**：Tool 层失败时记录错误并继续 LLM 层检测
4. **产物驱动**：最终报告从中间产物文件合并生成，不依赖 LLM 记忆
5. **`.guidelines-audit/` 目录应加入 `.gitignore`**
