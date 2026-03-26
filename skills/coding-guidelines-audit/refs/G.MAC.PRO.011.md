# G.MAC.PRO.011 不应通过过程宏将 unsafe 代码包装为 safe 代码

## 级别
要求

## 规范描述
不要通过过程宏将 unsafe 代码包装为 safe 代码，以此来规避 Rust 静态分析检查。

## 说明
过程宏生成的代码如果在内部使用了 `unsafe` 块，但对外暴露为 safe 接口，会导致调用者在不知情的情况下执行不安全代码，绕过了 Rust 的安全保证机制。这违反了 Rust 的安全设计原则：unsafe 代码的使用应当是显式的、可审计的。

相关讨论：
- [RUSTSEC-2020-0011](https://rustsec.org/advisories/RUSTSEC-2020-0011.html)
- [https://github.com/RustSec/advisory-db/issues/275](https://github.com/RustSec/advisory-db/issues/275)
- [https://github.com/rust-lang/unsafe-code-guidelines/issues/278](https://github.com/rust-lang/unsafe-code-guidelines/issues/278)

## 检查要点
- 过程宏生成的代码中是否将 `unsafe` 块包装在 safe 函数内而未标记 `unsafe`
- `#[proc_macro]`、`#[proc_macro_derive]`、`#[proc_macro_attribute]` 标注的函数中，`quote!` 生成的代码是否包含 `unsafe` 块
- 生成的 `unsafe` 代码是否要求调用者在 `unsafe` 上下文中使用
- 过程宏是否将外部 FFI 调用包装为 safe 接口

## 反例

```Rust
// 用 RUST 的过程宏包装 libc 的函数 printf 来打印一个 C 字符串
// 这个宏的输入参数必须是一个有效的 C 字符串指针，整个宏的使用是不安全的，
// 但是因为过程宏中包括了 unsafe，导致过程宏看起来像 safe 调用
//
// rprintf!(b"hello\0" as *const u8);

#[proc_macro]
pub fn rprintf(input: TokenStream) -> TokenStream {
  let expr = parse_macro_input!(input as syn::Expr);
  // 这里包括 unsafe 代码，这样整个宏可以在 safe 代码中使用
  quote!({
    unsafe {
      printf(b"%s\n\0" as *const _ as *const u8, #expr); 
    }
  }).into() 
}
```

## 正例

```Rust
// 用 RUST 的过程宏包装 libc 的函数 printf 来打印一个 C 字符串
// 这个宏的输入参数必须是一个有效的 C 字符串指针，同时也是调用外部函数
// 必须在 unsafe 代码块中使用
//
// unsafe { rprintf!(b"hello\0" as *const u8) };

#[proc_macro]
pub fn rprintf(input: TokenStream) -> TokenStream {
  let expr = parse_macro_input!(input as syn::Expr);
  // 这里不能包括 unsafe 代码，而是要求整个宏在 unsafe 代码块中使用
  quote!({
    printf(b"%s\n\0" as *const _ as *const u8, #expr); 
  }).into()
}
```

## 检查指令
扫描项目中所有 proc-macro crate 的 `.rs` 文件（通常在 `Cargo.toml` 中标记为 `[lib] proc-macro = true` 的 crate），执行以下检查：

1. 定位所有标注了 `#[proc_macro]`、`#[proc_macro_derive]`、`#[proc_macro_attribute]` 的函数。
2. 在这些函数体内，搜索 `quote!`、`quote_spanned!` 等代码生成宏的调用。
3. 分析生成的代码模板中是否包含 `unsafe` 块：
   - 检查 `quote!` 内部是否有 `unsafe { ... }` 代码块。
   - 检查是否将 FFI 函数调用（如 `extern "C"` 函数）包装在 `unsafe` 块中。
4. 如果生成的代码包含 `unsafe` 块，检查该宏的使用方式：
   - 宏展开后的代码是否要求调用者在 `unsafe` 上下文中使用。
   - 若宏内部包含 `unsafe` 但调用者无需 `unsafe`，则标记为"过程宏将 unsafe 代码包装为 safe 接口"。
5. 同时检查宏的文档注释中是否有 `# Safety` 部分说明安全要求。
6. 汇总所有不符合项，按文件路径和行号列出问题。
