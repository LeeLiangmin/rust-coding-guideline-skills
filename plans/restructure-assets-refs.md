# 重构计划：将 assets 和 refs 移入 skill 子目录（已完成）

## 背景

当前 `skills/` 目录下直接存放了 `assets/` 和 `refs/` 两个共享资源目录，导致 `skills/` 根目录不够干净。需要将它们移入各自的 skill 子目录中，保持 skill 的自包含性。

## 当前结构

```
skills/
├── assets/                          ← 共享资源，直接在 skills/ 下
│   ├── tool_handle.json
│   ├── llm_handle.json
│   └── output_schema.json
├── refs/                            ← 共享参考文件，直接在 skills/ 下
│   ├── rust_coding_guidelines.md
│   ├── G.CMT.02.md
│   ├── G.FMT.01.md
│   ├── G.TYP.BOL.03.md
│   ├── G.TYP.SCT.01.md
│   ├── G.CTF.01.md
│   ├── G.CTF.02.md
│   ├── G.MAC.DCL.01.md
│   ├── G.MAC.PRO.011.md
│   └── G.SAF.MEM.04.md
├── full-audit/
│   └── SKILL.md                     ← 引用 ../assets/ 和 ../refs/
└── rule-audit/
    └── SKILL.md                     ← 引用 ../assets/ 和 ../refs/
```

## 目标结构

```
skills/
├── full-audit/
│   ├── SKILL.md                     ← 引用 assets/ 和 refs/（同级目录）
│   ├── assets/                      ← 实际文件存放位置
│   │   ├── tool_handle.json
│   │   ├── llm_handle.json
│   │   └── output_schema.json
│   └── refs/                        ← 实际文件存放位置
│       ├── rust_coding_guidelines.md
│       ├── G.CMT.02.md
│       ├── G.FMT.01.md
│       ├── G.TYP.BOL.03.md
│       ├── G.TYP.SCT.01.md
│       ├── G.CTF.01.md
│       ├── G.CTF.02.md
│       ├── G.MAC.DCL.01.md
│       ├── G.MAC.PRO.011.md
│       └── G.SAF.MEM.04.md
└── rule-audit/
    ├── SKILL.md                     ← 引用 assets/ 和 refs/（通过符号链接）
    ├── assets -> ../full-audit/assets   ← 符号链接
    └── refs -> ../full-audit/refs       ← 符号链接
```

## 迁移步骤

### 步骤 1: 移动实际文件

将 `skills/assets/` 和 `skills/refs/` 移动到 `skills/full-audit/` 下：

```cmd
move skills\assets skills\full-audit\assets
move skills\refs skills\full-audit\refs
```

### 步骤 2: 创建目录链接

在 `skills/rule-audit/` 下创建指向 `full-audit` 对应目录的链接：

**Windows（NTFS Junction，无需管理员权限）：**

```cmd
cd skills\rule-audit
mklink /J assets ..\full-audit\assets
mklink /J refs ..\full-audit\refs
```

**Linux/macOS（符号链接）：**

```bash
cd skills/rule-audit
ln -s ../full-audit/assets assets
ln -s ../full-audit/refs refs
```

> 注意：Windows 上使用 NTFS Junction（`/J`），不需要管理员权限或开发者模式。Linux/macOS 上使用标准符号链接。Git 跨平台 clone 时，Linux 端会自动还原为符号链接。

### 步骤 3: 更新 full-audit/SKILL.md 路径引用

所有 `../assets/` → `assets/`，所有 `../refs/` → `refs/`。

**涉及的引用变更：**

| 原路径 | 新路径 |
|--------|--------|
| `(../assets/tool_handle.json)` | `(assets/tool_handle.json)` |
| `(../assets/llm_handle.json)` | `(assets/llm_handle.json)` |
| `(../assets/output_schema.json)` | `(assets/output_schema.json)` |
| `(../refs/rust_coding_guidelines.md)` | `(refs/rust_coding_guidelines.md)` |
| `(../refs/G.CMT.02.md)` | `(refs/G.CMT.02.md)` |
| `(../refs/G.FMT.01.md)` | `(refs/G.FMT.01.md)` |
| `(../refs/G.TYP.BOL.03.md)` | `(refs/G.TYP.BOL.03.md)` |
| `(../refs/G.TYP.SCT.01.md)` | `(refs/G.TYP.SCT.01.md)` |
| `(../refs/G.CTF.01.md)` | `(refs/G.CTF.01.md)` |
| `(../refs/G.CTF.02.md)` | `(refs/G.CTF.02.md)` |
| `(../refs/G.MAC.DCL.01.md)` | `(refs/G.MAC.DCL.01.md)` |
| `(../refs/G.MAC.PRO.011.md)` | `(refs/G.MAC.PRO.011.md)` |
| `(../refs/G.SAF.MEM.04.md)` | `(refs/G.SAF.MEM.04.md)` |

### 步骤 4: 更新 rule-audit/SKILL.md 路径引用

同样将所有 `../assets/` → `assets/`，`../refs/` → `refs/`。由于 `rule-audit/` 下有符号链接，`assets/` 和 `refs/` 会透明地解析到 `full-audit/` 下的实际文件。

变更内容与步骤 3 完全一致。

### 步骤 5: llm_handle.json 中的 ref_file 路径

`llm_handle.json` 中的 `ref_file` 字段当前值为 `refs/G.XXX.md`。由于该文件移动到 `full-audit/assets/` 下后，`refs/` 目录与 `assets/` 目录同级（都在 `full-audit/` 下），`ref_file` 路径保持不变即可正确解析。

**无需修改 JSON 文件内容。**

### 步骤 6: 更新 plans/architecture.md

更新其中的目录结构描述，将 `assets/` 和 `refs/` 从 `skills/` 根级别移到 `full-audit/` 下，并说明 `rule-audit/` 使用符号链接。

同时更新文档中引用 `skills/assets/` 和 `skills/refs/` 的路径描述。

### 步骤 7: 验证

1. 验证符号链接可正常读取文件
2. 验证 SKILL.md 中的 markdown 链接可正确跳转
3. 确认 `skills/` 根目录下只剩 `full-audit/` 和 `rule-audit/` 两个子目录

## 影响范围

| 文件 | 变更类型 |
|------|---------|
| `skills/assets/*` | 移动到 `skills/full-audit/assets/` |
| `skills/refs/*` | 移动到 `skills/full-audit/refs/` |
| `skills/rule-audit/assets` | 新建符号链接 → `../full-audit/assets` |
| `skills/rule-audit/refs` | 新建符号链接 → `../full-audit/refs` |
| `skills/full-audit/SKILL.md` | 更新路径引用 |
| `skills/rule-audit/SKILL.md` | 更新路径引用 |
| `plans/architecture.md` | 更新目录结构描述 |
