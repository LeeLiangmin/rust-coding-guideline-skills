# G.FMT.01 extern 外部函数应显式指定 ABI 标识

## 级别
要求

## 规范描述
当使用 `extern` 指定外部函数时，应显式指定 [ABI 标识](https://doc.rust-lang.org/reference/items/external-blocks.html#abi)，否则默认为 `C-ABI` 。显式指定 ABI 标识是 Rust 语言的一种约定俗成，有利于代码健壮性。

## 检查要点
- `extern fn` 声明是否显式指定了 ABI（如 `extern "C" fn`）
- `extern` 块（`extern { ... }`）是否显式指定了 ABI（如 `extern "C" { ... }`）
- 是否存在省略 ABI 标识的 `extern fn` 或 `extern { }` 声明
- ABI 标识是否使用了合法的值（如 `"C"`、`"Rust"`、`"system"` 等）

## 反例

```Rust
use std::os::raw::c_char;
// 不符合：不要省略 C-ABI 指定
extern {
  fn strlen(s: *const c_char) -> i32; 
  }
```

## 正例

```Rust
use std::os::raw::c_char;
// 符合
extern "C" {
  fn strlen(s: *const c_char) -> i32; }

// 符合
extern "Rust" {  
  type MyType;
  fn f(&self) -> usize;
}
```

## 检查指令
扫描项目中所有 `.rs` 文件，执行以下检查：

1. 使用正则表达式搜索所有 `extern` 关键字的使用位置。
2. 对于 `extern fn` 形式的函数声明，检查 `extern` 和 `fn` 之间是否有 ABI 字符串字面量（如 `"C"`、`"Rust"` 等）。若 `extern` 后直接跟 `fn`（即 `extern fn`），则标记为"extern fn 缺少显式 ABI 标识"。
3. 对于 `extern { ... }` 形式的外部块声明，检查 `extern` 和 `{` 之间是否有 ABI 字符串字面量。若 `extern` 后直接跟 `{`（即 `extern {`），则标记为"extern 块缺少显式 ABI 标识"。
4. 排除注释和字符串字面量中的 `extern` 关键字。
5. 汇总所有不符合项，按文件路径和行号列出问题。
