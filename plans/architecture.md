# Rust Coding Guidelines Skills 架构设计

> 基于 [idea.md](../idea.md) 的设计构想，构建一套 **静态工具 + LLM 辅助检测** 的 Rust 编程规范检查 Skills 体系。
> 采用统一的 [`coding-guidelines-audit`](../skills/coding-guidelines-audit/SKILL.md) Skill，支持全量/定向两种模式，具备中间产物持久化和断点续审能力。
> 重新设计方案详见 [redesign-plan.md](redesign-plan.md)。

---

## 1. 整体架构

### 1.1 核心理念

将 57 条 Rust 编程规范分为四层处理：

| 层级 | 处理方式 | 条目数 | 说明 |
|------|---------|--------|------|
| **clippy_native** | clippy 原生 lint 静态检测 | 21 条 | 有规范映射，默认启用 |
| **clippy_custom** | 扩展工具自定义 lint 静态检测 | 16 条 | 有规范映射，需 `-W` 启用 |
| **llm** | 大模型代码审查 | 9 条 | 工具无法覆盖的规则 |
| **排除层** | 不检测 | 11 条 | 原则类条目，仅作参考 |

此外，clippy 运行时还会输出大量**无规范映射的默认 lint**（`clippy_default`），这些诊断也会被保留在报告中。

### 1.2 四分类模型

所有检查项按"检查来源"和"是否有规范映射"两个维度分为四类：

| source 分类 | 含义 | 有 guideline_code | 检查方式 | 需要 -W 启用 |
|------------|------|-------------------|---------|-------------|
| `clippy_default` | clippy 默认 lint，无规范映射 | ❌ 留空 | 工具自动检出 | ❌ |
| `clippy_native` | clippy 原生 lint，有规范映射 | ✅ | 工具自动检出 | ❌ |
| `clippy_custom` | 扩展工具自定义 lint，有规范映射 | ✅ | 工具检出，需 -W 启用 | ✅ |
| `llm` | 工具无法检查，LLM 审查 | ✅ | LLM 代码审查 | — |

> `clippy_default` 不在注册表中注册（800+ lint 太多），运行时从 clippy 输出中动态识别。

### 1.3 工作流概览

```mermaid
flowchart TD
    A[用户触发 coding-guidelines-audit] --> B{.guidelines-audit/state.json 存在?}

    B -->|是| B1{有未完成步骤?}
    B1 -->|是| B2[提示用户: 发现未完成审计]
    B2 --> B3{继续 or 重新开始?}
    B3 -->|继续| B4[从中断步骤恢复]
    B3 -->|重新开始| C
    B1 -->|否| C

    B -->|否| C[步骤 1: 模式判断与初始化]

    C --> C1{用户指定了规则编号?}
    C1 -->|否: full 模式| C2[加载全部 46 条规则]
    C1 -->|是: specific 模式| C3[解析编号 - 按 check_type 分组]
    C2 --> D
    C3 --> D

    D[步骤 2: Clippy 检测] --> D1[执行 guidelines_runner]
    D1 --> D2[解析输出]
    D2 --> D3{每条诊断的 lint_code}

    D3 --> D4{在 registry 中查找}
    D4 -->|找到 clippy_native| D5[source=clippy_native + guideline_code]
    D4 -->|找到 clippy_custom| D6[source=clippy_custom + guideline_code]
    D4 -->|未找到| D7[source=clippy_default + guideline_code 留空]

    D5 --> D8[写入 tool-results.json]
    D6 --> D8
    D7 --> D8

    D8 --> E[步骤 3: LLM 检测]
    E --> E1[预过滤 - 分批审查 - 进度追踪]
    E1 --> E2[每批完成后更新 llm-progress.json]

    E2 --> F[步骤 4: 结果合并]
    F --> F1[从 tool-results.json + llm-progress.json 读取]
    F1 --> F2[合并 + 统计 by_source 四分类 + 写入 report.json]
```

---

## 2. 目录结构

### 2.1 Skill 目录

```
guideline-skills/
├── skills/
│   └── coding-guidelines-audit/          # 统一的审计 Skill
│       ├── SKILL.md                      # Skill 定义
│       ├── assets/
│       │   ├── guidelines_registry.json  # 46 条规范统一注册表
│       │   ├── llm_check_config.json     # 9 条 LLM 检查配置
│       │   └── output_schema.json        # 统一输出 schema（四分类）
│       └── refs/
│           ├── rust_coding_guidelines.md # 完整 Rust 编程规范文档
│           ├── G.CMT.02.md              # 文件头注释应包含版权说明
│           ├── G.CTF.01.md              # 宜优先使用模式匹配
│           ├── G.CTF.02.md              # Match Guard 不应有副作用
│           ├── G.FMT.01.md              # extern 应显式指定 ABI
│           ├── G.MAC.DCL.01.md          # 宏匹配范围从小至大
│           ├── G.MAC.PRO.011.md         # 过程宏不应包装 unsafe
│           ├── G.SAF.MEM.04.md          # 敏感信息使用后清零
│           ├── G.TYP.BOL.03.md          # && || 右侧不应有副作用
│           └── G.TYP.SCT.01.md          # 自定义类型宜实现常见 trait
├── plans/
│   ├── architecture.md                  # 本文档
│   ├── redesign-plan.md                 # 重新设计方案
│   └── data/                            # 参考数据
└── idea.md                              # 原始构想
```

### 2.2 运行时工作目录

审计过程中在**目标项目**中生成 `.guidelines-audit/` 目录，用于中间产物持久化：

```
<目标项目>/
└── .guidelines-audit/
    ├── state.json            # 审计总状态（断点续审核心）
    ├── tool-results.json     # 工具层检测结果（步骤 2 产物）
    ├── llm-progress.json     # LLM 层检查进度和中间结果（步骤 3 产物）
    └── report.json           # 最终审计报告（步骤 4 产物）
```

> ⚠️ `.guidelines-audit/` 目录应加入目标项目的 `.gitignore`。

---

## 3. 路由表设计

### 3.1 guidelines_registry.json — 统一规范注册表

[`assets/guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) 是所有 46 条规范条目的**唯一注册中心**。

**字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `guideline_code` | string | 规范编号，如 `G.NAM.01`（唯一标识） |
| `title` | string | 规范标题 |
| `level` | string | 级别：`requirement` / `suggestion` |
| `check_type` | string | 检查方式：`clippy_native` / `clippy_custom` / `llm` |
| `lint_codes` | string[] | 对应的 clippy/rustc lint 代码列表（`llm` 类型为空数组） |

> **注意**：`clippy_default` 不在此表中注册，因为它们没有规范映射，是运行时从 clippy 输出中动态识别的。

### 3.2 llm_check_config.json — LLM 检查配置

[`assets/llm_check_config.json`](../skills/coding-guidelines-audit/assets/llm_check_config.json) 仅包含 `check_type = "llm"` 的规则的检查参数。

**字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `guideline_code` | string | 规范编号（引用 registry 中的条目） |
| `ref_file` | string | 参考规范文件的相对路径 |
| `difficulty` | string | 检测难度：`low` / `medium` / `high` |
| `check_strategy` | string | 检查策略标识 |
| `check_description` | string | 给 LLM 的检查指令描述 |
| `audit_focus` | string | 一句话审查聚焦点 |
| `pre_filter` | object | 预过滤配置（`type` + 类型特定参数） |
| `batch_size` | integer | 每批审查的文件数 |

### 3.3 两表关系

```mermaid
erDiagram
    GUIDELINES_REGISTRY {
        string guideline_code PK
        string title
        string level
        string check_type
        array lint_codes
    }
    LLM_CHECK_CONFIG {
        string guideline_code FK
        string ref_file
        string difficulty
        string check_strategy
        string check_description
        string audit_focus
        object pre_filter
        int batch_size
    }
    GUIDELINES_REGISTRY ||--o| LLM_CHECK_CONFIG : "check_type=llm 时关联"
```

> `llm_check_config.json` 中的 `guideline_code` 必须在 `guidelines_registry.json` 中存在且 `check_type = "llm"`。这是一个**引用关系**，不是重复定义。

**`pre_filter` 类型说明：**

| type | 说明 | 参数 |
|------|------|------|
| `head` | 提取文件头部 N 行 | `lines`: 行数 |
| `grep` | 正则搜索匹配文件 | `patterns`: 正则列表, `context_lines`: 上下文行数 |
| `crate_type` | 按 crate 类型过滤 | `types`: crate 类型列表 |

---

## 4. 统一输出 Schema

所有检测结果统一输出为 JSON 格式（[`assets/output_schema.json`](../skills/coding-guidelines-audit/assets/output_schema.json)）。

### 4.1 顶层结构

| 字段 | 类型 | 说明 |
|------|------|------|
| `projectPath` | string | 被检测项目的绝对路径 |
| `checkMode` | `"full"` \| `"specific"` | 检测模式 |
| `checkedRules` | string[] | 本次检测涉及的规范编号列表 |
| `summary` | object | 统计摘要 |
| `diagnostics` | array | 诊断条目数组 |

### 4.2 summary 结构

```json
{
  "total": 38,
  "errors": 2,
  "warnings": 36,
  "by_source": {
    "clippy_default": 15,
    "clippy_native": 8,
    "clippy_custom": 10,
    "llm": 5
  }
}
```

使用 `by_source` 对象按四种诊断来源分类统计。

### 4.3 diagnostics 条目

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `level` | `"error"` \| `"warning"` \| `"note"` | ✅ | 诊断级别 |
| `code` | string | ✅ | lint 代码，如 `clippy::collapsible_if` |
| `guideline_code` | string | | 对应的规范编号，如 `G.NAM.01`（`clippy_default` 留空） |
| `source` | `"clippy_default"` \| `"clippy_native"` \| `"clippy_custom"` \| `"llm"` | ✅ | 诊断来源四分类 |
| `message` | string | ✅ | 诊断消息 |
| `file` | string | ✅ | 文件相对路径 |
| `line` | integer | ✅ | 行号 |
| `column` | integer | | 列号 |
| `rendered` | string | | 格式化的完整诊断输出（主要用于 clippy 来源） |
| `suggestions` | array | | 修复建议 |

---

## 5. Skill 工作流设计

统一的 [`coding-guidelines-audit`](../skills/coding-guidelines-audit/SKILL.md) Skill 通过自动判断模式（full/specific）执行审计，工作流分为 5 个步骤。

### 步骤 0: 断点检测

检查目标项目中 `.guidelines-audit/state.json` 是否存在：
- **不存在** → 全新审计，进入步骤 1
- **存在且全部 completed** → 询问用户是否重新审计
- **存在且有未完成步骤** → 展示进度摘要，询问继续或重新开始；继续则从中断步骤恢复

### 步骤 1: 模式判断与初始化

**模式判断**：用户提供规则编号 → `specific` 模式；未提供 → `full` 模式。

- **specific 模式**：解析编号，在 [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) 中查找，按 `check_type` 分为 `tool_group`（clippy_native + clippy_custom）和 `llm_group`
- **full 模式**：`tool_group` = 全部 37 条工具规则，`llm_group` = 全部 9 条 LLM 规则，同时包含所有 clippy 默认 lint
- 验证 `guidelines_runner` 工具链和 MCP Server 可用性

📝 产物：创建 `.guidelines-audit/state.json`

### 步骤 2: Clippy 检测（工具层）

1. **执行检测**：MCP 优先（`check_project`），CLI 回退（`cargo +guidelines_runner clippy --message-format=json`）
   - `clippy_custom` 类 lint 需通过 `-W` 参数显式启用
2. **输出解析**：筛选 `compiler-message`，提取诊断信息和 suggestions
3. **source 四分类映射**：查 [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json)：
   - 找到且 `check_type = clippy_native` → `source = "clippy_native"` + `guideline_code`
   - 找到且 `check_type = clippy_custom` → `source = "clippy_custom"` + `guideline_code`
   - 未找到 → `source = "clippy_default"` + `guideline_code` 留空

📝 产物：写入 `.guidelines-audit/tool-results.json`，更新 `state.json`

### 步骤 3: LLM 层检测

采用 **"预过滤 → 分批审查 → 进度追踪"** 三阶段策略：

1. **预过滤**：按 [`llm_check_config.json`](../skills/coding-guidelines-audit/assets/llm_check_config.json) 中的 `pre_filter` 配置缩小审查范围
2. **分批审查**：每批单规则 + 对应 [`refs/G.XXX.md`](../skills/coding-guidelines-audit/refs/) 参考文件，`batch_size` 由配置控制
3. **进度追踪**：维护 `文件 × 规则` 检查矩阵，确保全覆盖

```mermaid
flowchart TD
    A[从 registry 筛选 check_type=llm] --> A1[加载 llm_check_config.json]
    A1 --> B[扫描项目 .rs 文件]
    B --> C[初始化检查矩阵]
    C --> D{还有未完成的规则?}
    D -->|否| E[LLM 层检测完成]
    D -->|是| F[取下一条规则]
    F --> G[按 pre_filter 预过滤]
    G --> H{还有待检查的文件?}
    H -->|否| I[该规则完成]
    H -->|是| J[取一批文件]
    J --> K[加载 refs/G.XXX.md + 构建 prompt]
    K --> L[LLM 审查 - 解析输出]
    L --> M[更新 llm-progress.json]
    M --> H
    I --> D
```

📝 产物：每批完成后更新 `.guidelines-audit/llm-progress.json`，全部完成后更新 `state.json`

⚠️ 断点续审时从 `llm-progress.json` 恢复，跳过已完成的规则和文件

### 步骤 4: 结果合并与输出

1. 从 `tool-results.json` 和 `llm-progress.json` 读取诊断
2. 合并为统一 `diagnostics` 数组
3. 计算 `summary`（含 `by_source` 四分类统计）
4. 写入最终报告

📝 产物：写入 `.guidelines-audit/report.json`，更新 `state.json`

⚠️ 最终报告从中间产物文件合并生成，不依赖 LLM 记忆

---

## 6. refs 参考文件设计

[`coding-guidelines-audit/refs/`](../skills/coding-guidelines-audit/refs/) 目录包含两类参考文件：

### 6.1 完整规范文档

[`refs/rust_coding_guidelines.md`](../skills/coding-guidelines-audit/refs/rust_coding_guidelines.md) — 完整的 Rust 编程规范文档，作为 LLM 审查时的通用参考上下文。

### 6.2 LLM 规则专项参考文件

每个 LLM 检测规则对应一个 `.md` 参考文件，结构统一：

```markdown
# G.XXX.XX 规则标题

## 级别
requirement / suggestion

## 规范描述
...

## 检查要点
- 要点 1
- 要点 2

## 正例
...代码示例...

## 反例
...代码示例...

## 检查指令
给 LLM 的具体审查指令...
```

---

## 7. MCP Server 集成

### 7.1 配置

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

### 7.2 在 Skill 中的使用

统一的 `coding-guidelines-audit` Skill 按以下优先级调用工具：

1. **MCP 优先**：调用 `check_project` 工具，传入项目路径
2. **CLI 回退**：`cargo +guidelines_runner clippy --message-format=json`

两种方式输出格式相同（JSON），Skill 工作流无需区分处理。

---

## 8. 规则覆盖总览

| 分类 | 条目数 | 处理方式 | 数据来源 |
|------|--------|---------|---------|
| 🟢 clippy_native | 21 | clippy 原生 lint 静态检测 | [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) |
| 🔵 clippy_custom | 16 | 扩展工具自定义 lint 静态检测 | [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) |
| 🔴 llm | 9 | LLM 代码审查 | [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) + [`llm_check_config.json`](../skills/coding-guidelines-audit/assets/llm_check_config.json) |
| ⚪ 原则类条目 | 11 | 不检测 | 无 |
| 🟡 clippy_default | 800+ | clippy 默认 lint（无规范映射） | 运行时动态识别 |
| **规范合计** | **57** | — | — |

---

## 9. 维护指南

### 9.1 日常维护事项

- **新增 clippy_native 规则**：在 [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) 中添加条目，`check_type` 设为 `clippy_native`
- **新增 clippy_custom 规则**：在 [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) 中添加条目，`check_type` 设为 `clippy_custom`
- **新增 LLM 规则**：
  1. 在 [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) 中添加条目，`check_type` 设为 `llm`
  2. 在 [`llm_check_config.json`](../skills/coding-guidelines-audit/assets/llm_check_config.json) 中添加检查配置
  3. 创建对应的 `refs/G.XXX.md` 参考文件
- **修改输出格式**：更新 [`output_schema.json`](../skills/coding-guidelines-audit/assets/output_schema.json)，同步更新 SKILL.md 中的相关描述

---

## 10. 诊断来源（source）四分类设计

### 10.1 分类体系

| source 值 | 含义 | 来源 | guideline_code |
|-----------|------|------|----------------|
| `clippy_default` | clippy 默认 lint，无规范映射 | clippy 输出 | 留空 |
| `clippy_native` | clippy 原生 lint，有规范映射 | clippy 输出 | 有映射 |
| `clippy_custom` | 扩展工具自定义 lint，有规范映射 | clippy 输出（需 -W 启用） | 有映射 |
| `llm` | LLM 代码审查 | AI 审查结果 | 有映射 |

### 10.2 映射逻辑

对 clippy 输出的每条诊断：

1. 用 `lint_code` 在 [`guidelines_registry.json`](../skills/coding-guidelines-audit/assets/guidelines_registry.json) 中查找
2. **找到且 check_type = clippy_native**：
   - `guideline_code` = 匹配规则的 `guideline_code`
   - `source` = `"clippy_native"`
3. **找到且 check_type = clippy_custom**：
   - `guideline_code` = 匹配规则的 `guideline_code`
   - `source` = `"clippy_custom"`
4. **未找到**（标准 clippy lint，不在注册表中）：
   - `guideline_code` = `""`（留空）
   - `source` = `"clippy_default"`

### 10.3 设计考量

- 四分类使报告消费者能精确区分：无规范映射的 clippy 默认检测、有规范映射的 clippy 原生检测、扩展自定义检测和 LLM 检测
- `by_source` 统计对象按四类分别计数，具备良好的扩展性
- 未映射的标准 clippy lint 统一归入 `clippy_default`，保证不丢失任何诊断
- `clippy_default` 不在注册表中注册，避免维护 800+ lint 条目的负担

---

## 11. 中间产物持久化设计

### 11.1 设计原则

| 原则 | 说明 |
|------|------|
| **每步写入** | 每个主要步骤完成后立即将结果写入文件 |
| **断点续审** | AI 重新加载时可检测并恢复上次进度 |
| **增量更新** | LLM 层每批审查完成后更新进度文件 |
| **最终合并** | 最终报告从中间产物文件合并生成，不依赖 LLM 记忆 |

### 11.2 state.json — 审计总状态

```json
{
  "version": "2.0",
  "mode": "full",
  "projectPath": "/path/to/project",
  "startedAt": "2024-01-01T00:00:00Z",
  "updatedAt": "2024-01-01T00:15:00Z",
  "steps": {
    "step1_init": "completed",
    "step2_tool": "completed",
    "step3_llm": "in_progress",
    "step4_merge": "pending"
  },
  "checkedRules": ["G.NAM.01", "G.NAM.02", "G.CMT.02"],
  "config": {
    "specificRules": [],
    "toolGroup": ["G.NAM.01", "G.NAM.02"],
    "llmGroup": ["G.CMT.02"]
  }
}
```

步骤状态枚举：`pending` → `in_progress` → `completed` | `failed`

### 11.3 tool-results.json — 工具层检测结果

步骤 2 完成后写入，包含已解析、已映射的全部工具诊断：

```json
{
  "completedAt": "2024-01-01T00:05:00Z",
  "rawDiagnosticsCount": 25,
  "diagnostics": [
    {
      "level": "warning",
      "code": "clippy::wrong_self_convention",
      "guideline_code": "G.NAM.02",
      "source": "clippy_native",
      "message": "methods called to_* usually take self by reference",
      "file": "src/lib.rs",
      "line": 42,
      "column": 5,
      "rendered": "...",
      "suggestions": []
    },
    {
      "level": "warning",
      "code": "clippy::collapsible_if",
      "guideline_code": "",
      "source": "clippy_default",
      "message": "this if statement can be collapsed",
      "file": "src/main.rs",
      "line": 74,
      "column": 5,
      "rendered": "...",
      "suggestions": []
    }
  ]
}
```

### 11.4 llm-progress.json — LLM 层检查进度

步骤 3 过程中**持续更新**，每批审查完成后追加结果：

```json
{
  "updatedAt": "2024-01-01T00:15:00Z",
  "files": ["src/main.rs", "src/lib.rs", "src/utils.rs"],
  "rules": {
    "G.CMT.02": {
      "status": "completed",
      "fileStatus": {
        "src/main.rs": "checked",
        "src/lib.rs": "checked",
        "src/utils.rs": "skipped"
      },
      "diagnostics": [{ "...": "..." }]
    },
    "G.CTF.01": {
      "status": "in_progress",
      "fileStatus": {
        "src/main.rs": "checked",
        "src/lib.rs": "pending"
      },
      "diagnostics": []
    }
  }
}
```

`fileStatus` 枚举：`pending` → `checked` | `skipped`（预过滤无匹配）

### 11.5 report.json — 最终审计报告

步骤 4 产物，结构遵循 [`output_schema.json`](../skills/coding-guidelines-audit/assets/output_schema.json) 定义。

### 11.6 断点续审工作流

```mermaid
flowchart TD
    A[AI 加载 coding-guidelines-audit skill] --> B{state.json 存在?}

    B -->|否| C[全新审计: 从步骤 1 开始]
    B -->|是| D[读取 state.json]

    D --> E{有未完成的步骤?}
    E -->|否| F[上次审计已完成，询问是否重新审计]
    E -->|是| G[提示用户: 发现上次未完成的审计]

    G --> H{用户选择?}
    H -->|继续| I[从中断步骤恢复]
    H -->|重新开始| C

    I --> J{中断在哪个步骤?}
    J -->|step2_tool| K[重新执行 Clippy 检测]
    J -->|step3_llm| L[读取 llm-progress.json, 跳过已完成规则/文件]
    J -->|step4_merge| M[读取中间产物, 直接合并]
```
