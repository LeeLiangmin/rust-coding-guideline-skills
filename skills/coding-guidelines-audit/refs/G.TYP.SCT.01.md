# G.TYP.SCT.01 外部使用的自定义类型宜实现常见 trait

## 级别
建议

## 规范描述
为外部使用的自定义类型 API 实现一些常见的公共 trait 有利于代码持续演进，并且提高可维护性。

建议实现的公共 trait 包括： `Debug` , `Default` , `Clone` , `PartialEq` , `PartialOrd` , `Hash` , `Eq` , `Ord` 。

## 检查要点
- `pub struct` 是否实现了 `Debug` trait（通过 `#[derive(Debug)]` 或手动实现）
- `pub enum` 是否实现了 `Debug` trait（通过 `#[derive(Debug)]` 或手动实现）
- `pub struct` / `pub enum` 是否实现了 `Clone` trait
- `pub struct` / `pub enum` 是否实现了 `PartialEq` trait
- `pub struct` / `pub enum` 是否实现了 `Default` trait（对于 struct 类型）
- 如果类型的所有字段都支持 `Eq`、`Hash`、`Ord`，是否也实现了这些 trait
- 是否存在仅 derive 了部分常见 trait 而遗漏了其他适用 trait 的情况

## 正例

```Rust
use std::{path::PathBuf, time::Duration};

#[derive(Clone, Debug, Default, PartialEq)]
pub struct MyConfiguration {
  output: Option<PathBuf>,
  search_path: Vec<PathBuf>, 
  timeout: Duration,
  check: bool, 
}

#[derive(Clone, Debug, Default, PartialEq, Eq, PartialOrd, Ord, Hash)]
pub struct MyData {
  value: i32,
}
```

## 检查指令
扫描项目中所有 `.rs` 文件，执行以下检查：

1. 搜索所有 `pub struct` 和 `pub enum` 定义（包括 `pub(crate)` 等可见性修饰符的变体）。
2. 对于每个公开类型定义，检查其上方的 `#[derive(...)]` 属性：
   - 是否包含 `Debug`：若缺少，标记为"公开类型缺少 Debug trait 实现"。
   - 是否包含 `Clone`：若缺少，标记为"公开类型缺少 Clone trait 实现"。
   - 是否包含 `PartialEq`：若缺少，标记为"公开类型缺少 PartialEq trait 实现"。
3. 对于 `pub struct` 类型，额外检查是否实现了 `Default`。
4. 同时搜索手动 `impl Debug for`、`impl Clone for` 等实现，作为 derive 的补充判断。
5. 对于包含所有字段均为基本类型或已实现 `Eq`/`Hash`/`Ord` 的类型，建议也实现这些 trait。
6. 汇总所有不符合项，按文件路径和行号列出缺少的 trait。
