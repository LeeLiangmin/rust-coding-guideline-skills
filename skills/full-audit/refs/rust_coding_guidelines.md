# Rust 编程规范

> 来源：[旋武社区](https://xuanwu.openatom.cn/articles/rules/coding-guidelines/)
> 最后更新：2026/3/11

---

# Rust 编程规范

旋武社区发表于：2024-05-08


# 1 概述


## 1.1 背景

为了帮助开发者高效安全的使用 Rust 语言进行开发，保证 Rust 代码质量，特制定 Rust 编程规范。


## 1.2 目标与适用范围

**本规范的目标如下：**

- 遵循 Rust 语言特性，提高代码的可读性、可维护性、健壮性和可移植性。
- 提高 Unsafe Rust 代码编写的规范性和安全性。
- 编程规范条款力求系统化、易应用、易检查，帮助开发者提升开发效率。

**适用范围：**

本规范适用于公司使用 Rust 语言编写的自研代码。


## 1.3 总体原则

程序需要在保证功能正确的前提下，满足可读、可维护、安全、可靠、可测试、高效、可移植的特征要求。


## 1.4 条款组织方式

每个条款一般包含标题、级别、描述等组成部分

**规范条款分为原则和规则两个类别：**

- **原则：** 可以评价规则内容制定的好坏并引导规则进行相应的调整（从规范架构和功能上说，指导规 则的制定以及代码/框架/功能的设计）。工具不检测原则类条目。
- **规则：** 是需要遵从或参考的实践。工具会检测规则类条目。

**标题**

概括本条款的要求或建议。

通过标题前的编号标识出条款的类别为原则或规则：其中'P'为单词Principle首字母， 'G'为单词 Guideline的首字母。

- 标识 `P` 为原则。编号方式为 `P.Element.Number` 。
- 标识 `G` 为规则。编号方式为 `G.Element.Number `。
- 当有子目录时，编号方式为 `P.Element.SubElement.Number` 或 `G.Element.SubElement.Number` 。 其中 `Element` 为领域知识中关键元素（本规范中对应的二级目录）的 3 英文字母缩略语。 `Number` 是 从 1 开始递增的两位阿拉伯数字，不足两位时高位补 0。

（术语参考： [SEI CERT C Coding Standard](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) ）

| Element | 解释 | Element | 解释 |
| --- | --- | --- | --- |
| NAM | 命名 (Naming) | CMT | 注释 (Comment) |
| FMT | 格式 (Format) | TYP | 数据类型 (Data Type) |
| CNS | 常量 (Const) | PTR | 指针 (Pointer) |
| EXP | 表达式 (Expression) | CTF | 控制流程 (Control Flow) |
| STR | 字符串 (String) | INT | 整数 (Integer) |
| MOD | 模块 (Module) | MEM | 内存 (Memory) |
| MAC | 宏 (Macro) | FUD | 函数设计 (Function Design) |
| ASY | 异步 (Async) | TRA | Trait |
| ERR | 错误处理 (Error Handle) | UNS | 非安全 (Unsafe Rust) |
| MTH | 多线程 (Multi Threads) | FFI | 外部函数调用接口 ( Foreign Function Interface ) |
| SLC | 切片类型 (Slice) | BOL | 布尔 (Bool) |
| CHR | 字符类型 (Char) | FLT | 浮点数 (Float) |
| SCT | 结构体 (Struct) | SAF | 内存安全 (Safety) |
| DCL | 声明宏 (Declarative) | PRO | 过程宏 (Procedural) |

**级别** 规则类条款分为两个级别：

- 要求：表示产品原则上应该遵从，但可以按照具体产品版本计划和节奏分期实现
- 建议：表示该条款属于优秀的编程实践，有助于进一步消解风险，产品可结合业务情况考虑是否纳 入，但要保证实施一致的代码风格
- 

**描述**

对条款的进一步描述，描述条款的原理，配合正确和错误的代码例子作为示范。有的条款还包含一些规 则不适用的例外场景。

**说明**

对条款中的名词、背景、原理等进行补充说明。


# 2 代码风格


## 2.1 命名


#### P.NAM.01 标识符命名应符合阅读习惯

**【描述】**

标识符的命名要清晰、明了，有明确含义，容易理解。符合英文阅读习惯的命名将明显提高代码可读性。

**【说明】**

一些好的实践包括但不限于：

- 使用正确的英文单词并符合英文语法，不要使用拼音
- 仅使用常见或领域内通用的单词缩写
- 布尔型变量或函数避免使用否定形式


#### G.NAM.01 应使用统一的命名风格

**【级别】** 建议

**【描述】**

应在代码中遵循统一的 Rust 命名风格。

- 通常在 类型、trait、枚举体 等语言项使用大驼峰（ `UpperCamelCase` ） 命名风格，在 变量名、函 数名 等语言项使用蛇形（ `snake_case` ）命名风格。
- 为 Cargo Feature 命名应该避免出现： `no-` 或 `not-` 之类的否定前缀； `use-` 或 `with-` 之类的多余前缀； `-support` 之类的多余后缀。
- 

**【说明】**

下面是汇总信息：

| Item | 规范 |
| --- | --- |
| 包（Crates） | 通常使用 snake_case |
| 模块（Modules） | snake_case |
| 类型（Types） | UpperCamelCase |
| Trait | UpperCamelCase |
| 枚举体 | UpperCamelCase |
| 枚举体成员（Enum variants） | UpperCamelCase |
| 结构体（Structs） | UpperCamelCase |
| 结构体/联合体成员 | snake_case |
| 函数（Functions） | snake_case |
| 方法（Methods） | snake_case |
| 通用构造函数（General constructors） | new 或者 with_more_details |
| 转换构造函数（Conversion constructors） | from_some_other_type |
| 宏（Macros） | snake_case |
| 本地变量（Local variables） | snake_case |
| 静态变量（Statics） | SCREAMING_SNAKE_CASE |
| 常量（Constants） | SCREAMING_SNAKE_CASE |
| 类型参数（Type parameters） | 简明的 UpperCamelCase ，通常使用单个大写字母： T |
| 生存期（Lifetimes） | 简短的 lowercase ，通常使用单个小写字母 'a , 'de , 'src , 尽量保持语义 |
| 特性（Features） | snake_case |

补充解释:

- `UpperCamelCase` : 每个单词的首字母都是大写的，其余部分小写，没有分隔符。由首字母缩写组 成的缩略语和复合词的缩写，算作单个词。比如，应该使用 `Uuid ` 而非 `UUID` ，使用 `Stdin `而不 是 `StdIn` 。
- `snake_case` : 每个单词都是小写的，使用下划线作为分隔符， 如 `is_xid_start` 。
- `SCREAMING_SNAKE_CASE` : 每个单词都是大写的，使用下划线作为分隔符。
- 在 `snake_case` 或者 `SCREAMING_SNAKE_CASE` 情况下，每个词不应该由单个字母组成——除非这 个字母是最后一个词。比如，使用 `btree_map` 而不使用 `b_tree_map` ，使用 `PI_2` 而不使用`PI2` 。
- 通用构造函数的 `with_more_details` 形式是指推荐用户使用如: with_capacity , with_file_name 等函数名。
- 转换构造函数的 `from_some_other_type` 形式是指推荐用户使用如: `from_bytes` , `from_str` , `from_array` 等函数名。

关于包命名：

- 由于历史问题，包名有两种形式 `snake_case` 或 `kebab-case` ，但实际在代码中需要引入包名的时 候， Rust 只能识别 `snake_case` ，也会自动将 `kebab-case` 识别为 `kebab_case` 。所以建议使用 `snake_case` 。
- Crate 的名称通常不应该使用 `-rs` 或者 `-rust` 作为后缀或者前缀。但是有些情况下，比如是其他 语言移植的同名 Rust 实现，则可以使用 `-rs` 后缀来表明这是 Rust 实现的版本。


#### G.NAM.02 类型转换函数命名宜遵循所有权语义

**【级别】** 建议

**【描述】**

Rust 语言依赖所有权来管理内存，所以在命名上遵循所有权语义，可以提升代码可读性和透明性，以便 从代码命名中发现内存使用的代价。

**【说明】**

进行特定类型转换的方法名应该包含以下前缀：

| 名称前缀 | 内存代价 | 所有权 | 方法中self的类型 |
| --- | --- | --- | --- |
| as_ | 无代价 | borrowed -> borrowed | &self 或 &mut self |
| to_ | 代价昂贵 | borrowed ->borrowed borrowed -> owned (非 Copy 类型)  owned -> owned (Copy 类型) | &mut self &self (非 Copy 类型) self (Copy 类型) |
| into_ | 看情况 | owned -> owned (非 Copy 类型) | self |

以 `as_` 和` into_` 作为前缀的类型转换通常是 降低抽象层次，要么是查看背后的数据 (`as`)，要么是解构 (deconstruct) 背后的数据 ( `into` ) 。

相对来说，以 `to_` 作为前缀的类型转换处于同一个抽象层次，但是底层会做更多工作，比如多了内存拷贝等操作。

当一个类型用更高级别的语义 (higher-level semantics) 封装 (wraps) 一个内部类型时，应该使用 `into_inner()` 方法名来取出被封装类型的值。

**【反例】**

```Rust
fn main() {
  struct X();
  static S: &str = "str";

  impl X {
    // 不符合：as_ 命名的方法不应获取 self 的所有权
    fn as_str(self) -> &'static str {
      S
    }
    // 不符合：当返回值不是借用，to_ 命名的方法不应使用 &mut self 作为参数
    fn to_my_string(&mut self) -> String { S.to_string()
      }
    // 不符合：into_ 通常需要获取 self 的所有权
    fn into_my_string(&self) -> String { 
      S.to_string()
    }
  }
}
```

**【正例】**

```Rust
fn main() {
  struct X();
  static S: &str = "str";

  impl X {
    // 符合：as_ 命名的方法只借用 self
    fn as_str(&self) -> &'static str {
    S
    }
    // 符合：返回值不是借用，且 self 对应的类型不支持 Copy 时，to_ 命名的方法则使用 &self 作为参数
    fn to_my_string(&self) -> String { S.to_string()
    }
    // 符合：into_ 命名的方法获取了 self 的所有权
    fn into_my_string(self) -> String { S.to_string()
    }
  }
}
```

其他标准库示例：

- as_

1. [str::as_bytes()](https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes)

用于查看 UTF-8 字节的 `str` 切片，这是无内存代价的（不会产生内存分配）。 传入值是 `&str` 类型，输出值是 `&[u8]` 类型。

- to_

1. [Path::to_str](https://doc.rust-lang.org/stable/std/path/struct.Path.html#method.to_str)

对操作系统路径进行 UTF-8 字节检查，开销昂贵。

虽然输入和输出都是借用，但是这个方法对运行时产生不容忽视的代价。

1. [str::to_lowercase()](https://doc.rust-lang.org/std/primitive.str.html#method.to_lowercase)

生成正确的 Unicode 小写字符，

涉及遍历字符串的字符，可能需要分配内存。

输入值是 ``类型，输出值是 `String` 类型。

1. [f64::to_radians()](https://doc.rust-lang.org/std/primitive.f64.html#method.to_radians)

把浮点数的角度制转换成弧度制。

输入和输出都是 `f64` 。没必要传入 ` &f64` ，因为复制`f64`花销很小。 但是使用 `into_radians` 名称就会具有误导性，因为输入数据没有被消耗。

- into_

1. [String::into_bytes()](https://doc.rust-lang.org/std/string/struct.String.html#method.into_bytes)

从 `String` 提取出背后的 `Vec<u8>` 数据，这是无代价的。 它转移了 `String` 的所有权，然后返回具有所有权的 `Vec<u8>` 。

1. [BufReader::into_inner()](https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner)

转移了 buffered reader 的所有权，取出其背后的 reader ，这是无代价的。 存于缓冲区的数据被丢弃了。

1. [BufWriter::into_inner()](https://doc.rust-lang.org/std/io/struct.BufWriter.html#method.into_inner)

转移了 buffered writer 的所有权，取出其背后的 writer ，这可能以很大的代价刷新所有缓存数据。

如果类型转换方法返回的类型具有 `mut `修饰，那么这个方法的名称应如同返回类型组成部分的顺序那 样，带有 mut。

比如[Vec::as_mut_slice](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.as_mut_slice) 返回 `&mut [T]` 类型，这个方法的功能正如其名称所述，所以这个名称优于 `as_slice_mut` 。

- [Result::as_ref](https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref)
- [RefCell::as_ptr](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr)
- [slice::to_vec](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec)
- [Option::into_iter](https://doc.rust-lang.org/std/option/enum.Option.html#method.into_iter)


## 2.2 注释与格式


#### G.CMT.01 公开的 API 需要文档注释

**【级别】** 建议

**【描述】**

- 公开 API 如需返回 Result 类型的函数，则相应文档中需增加 Error 注释 解释什么场景下会返回什么样的错误类型，便于调用者处理错误。
- 公开 API 如有发生 Panic 的可能，则相应文档中需增加 Panic 注释 解释在什么条件下会发生 Panic ，便于调用者进行预处理。
- 公开 API 中的 Unsafe 函数或 trait ，文档中需添加 Safety 注释和相关说明 解释该函数接口的安全边界，便于调用者安全地使用。

在函数内部的 unsafe 代码块（即 unsafe {...} ）上方，不统一要求，但是建议添加 SAFETY 注 释，提升代码的可维护性。

Safety 文档注释建议包含以下内容（如果适用）：

- 前提条件（例如，参数的有效状态是什么？） 。 失败处理（例如，什么值应该被释放？）
- 成功处理（例如，什么值被创建或消耗？）

**【正例】**

```Rust
use std::io;
// 符合：增加了规范的 Errors 文档注释 /// ...
/// # Errors ///
/// Will return `Err` if `filename` does not exist or the user does not have /// permission to read it.
pub fn read(filename: &str) -> io::Result<String> {
  // ...
}

// 符合：增加了规范的 Panic 注释 /// ...
/// # Panics ///
/// Will panic if y is 0
pub fn divide_by(x: i32, y: i32) -> i32 { 
  if y == 0 {
    panic!("Cannot divide by 0") 
  } else {
    x / y
  } 
}

// 符合：增加了 unsafe 下的 Safety 注释 /// ...
/// # Safety ///
/// The bytes passed in must be valid UTF-8. /// ...
pub const unsafe fn from_utf8_unchecked(v: &[u8]) -> &str {
  // SAFETY: the caller must guarantee that the bytes `v` are valid UTF-8.
  // Also relies on `&str` and `&[u8]` having the same layout.
unsafe { mem::transmute(v) } 
}
```


#### G.CMT.02 文件头注释应包含版权说明

**【级别】** 建议

**【描述】**

文件头注释应首先包含版权说明。

如果文件头注释需要增加其他内容，可以后面补充。

比如：文件功能说明，作者、创建日期、注意事项等等。

编写版权注释时注意事项：

- 文件头注释应该从文件顶头开始。
- 保持统一格式。
- 保持版面工整，若内容过长，超出行宽要求，换行时应注意对齐。
- 首先包含“版权许可” ，然后包含其他可先选内容。
- 不要空有格式，无内容。


#### P.CMT.01 注释跟代码一样重要，应按需注释

**【描述】**

一般应通过清晰的软件架构，良好的符号命名来提高代码可读性；在需要的时候，才辅以注释说明。 注释是为了帮助阅读者快速读懂代码，所以要从读者的角度出发， **按需注释。**

注释内容要简洁、明了、无歧义，信息全面且不冗余。

需要注释的地方没有注释，代码则难以被读懂；而包含无用、重复信息的冗余注释不仅浪费维护成本， 还会弱化真正有用的注释，最终让所有注释都不可信。

**注释跟代码一样重要。**

写注释时要换位思考，用注释去表达此时读者真正需要的信息。在代码的功能、意图层次上进行注释， 即注释解释代码难以表达的意图，不要重复代码信息。

修改代码时，也要保证其相关注释的一致性。只改代码，不改注释是一种不文明行为，破坏了代码与注 释的一致性，让阅读者迷惑、费解，甚至误解。

使用流利的中文或英文进行注释。

为降低沟通成本，应使用团队内最擅长、沟通效率最高的语言写注释；注释语言由开发团队统一决定。


#### P.FMT.01 代码格式应保持统一

**【描述】**

代码应该保持统一的格式，这样可以增强代码可读性和可维护性，利于团队协作。

统一的代码格式应遵循以下规范：

- 缩进：使用四个空格（space）而非制表符（tab），文档代码同等对待。
- 换行：统一大括号换行风格；多行表达式操作时，操作符应该置于行首。
- 行间：如果需要留空，则最多空一行。
- 行宽：不超过 120 个字符（注： rustfmt 缺省宽度（ max_width）为 100，因此需要在 rustfmt 模板中明确配置）。
- 

应该总是使用代码自动格式化工具，比如 rustfmt ，来达成这一目标。

**【说明】**

rustfmt 支持各团队按需定制代码风格。


#### G.FMT.01 extern 外部函数应显式指定 ABI 标识

**【级别】** 要求

**【描述】**

当使用 `extern` 指定外部函数时，应显式指定 [ABI 标识](https://doc.rust-lang.org/reference/items/external-blocks.html#abi)，否则默认为 `C-ABI` 。 显式指定 ABI 标识是 Rust 语言的一种约定俗成，有利于代码健壮性。

**【反例】**

```Rust
use std::os::raw::c_char;
// 不符合：不要省略 C-ABI 指定
extern {
  fn strlen(s: *const c_char) -> i32; 
  }
```

**【正例】**

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


# 3 编程实践


## 3.1 常量


#### G.CNS.01 常量类型不应具有内部可变性

**【级别】** 要求

**【描述】**

内部可变性类型包括： `atomics` , `Cell` , `RefCell` , `UnsafeCell` , `Mutex` , `RwLock` , `OnceLock` 等，对 `const` 变量的修改是无效的，有可能跟程序语义预期结果不符

**【说明】**

所有 atomics 类型可参考: [https://doc.rust-lang.org/std/sync/atomic/index.html#structs](https://doc.rust-lang.org/std/sync/atomic/index.html#structs)

**【反例】**

```Rust
use std::sync::atomic::{AtomicUsize, Ordering::SeqCst};

// 不符合
const CONST_ATOM: AtomicUsize = AtomicUsize::new(12);

CONST_ATOM.store(6, SeqCst); // 对 const 变量值修改实际是无效的 
assert_eq!(CONST_ATOM.load(SeqCst), 12); // 仍为 12
```

**【反例】**

```Rust
use core::cell::Cell;

// 不符合
const VAL: Cell<i32> = Cell::new(100);

VAL.set(1);
assert_eq!(VAL.get(), 100); // 仍为原来的数值 100
```

**【反例】**

```Rust
use std::sync::Mutex;
use std::sync::RwLock;
use std::sync::OnceLock;

// 不符合
const M: Mutex<u8> = Mutex::new(5);
const R: RwLock<u8> = RwLock::new(5);
const O: OnceLock<u8> = OnceLock::new();

fn main() {
  *M.lock().unwrap() = 10; println!("{:?}", M);

  *R.write().unwrap() = 10; println!("{:?}", R);
  O.set(10).unwrap();
  println!("{:?}", O); 
}
```


#### G.CNS.02 宜使用标准库中的预定义常量

**【级别】** 建议

**【描述】**

Rust 中提供了一些特殊常量的定义（例如std::f32::consts），其精确度通常会比开发者自行定义的高， 所以若考虑数值精确度时，宜使用标准库预定义的特殊常量。

**【反例】**

```Rust
let min: i8 = -128;
let max: u16 = 65535;

// 不符合
let x = 3.14_f64;  let y = 1_f64 / x;
```

**【正例】**

```Rust
let min: i8 = i8::MIN;
let max: u16 = u16::MAX;

let x = std::f64::consts::PI;
let y = std::f64::consts::FRAC 1 PI;

let inf = f32::INFINITY;
let neg_inf = f32::NEG_INFINITY;
```


## 3.2 数据类型


#### G.TYP.01 数值字面量要添加明确的类型标识

**【级别】** 要求

**【描述】**

如果数值字面量没有被指定具体类型，那么单靠类型推导，整数类型会被默认绑定为 i32 类型，而浮点 数则默认绑定为 f64 类型。这可能导致某些运行时的意外，比如因为类型预期错误，导致应该在编译期 发现的问题遗留到运行时，比如导致数据计算溢出等问题。

**【反例】**

```Rust
// 期望是 u64 类型，用于统计数据，不考虑溢出问题
let mut count = 0;

while (/* ... */) {
  // 每秒执行一次网络流量统计
  let bytes: i32 = /* get packet size */
  // 不符合： u64 不会溢出，而 i32 可能会溢出。此处本应该编译报错。
  count += bytes; 
}
```

**【反例】**

```Rust
#![warn(clippy::default_numeric_fallback)]
// 不符合
let i = 10; // i32
let f = 1.23; // f64
```

**【正例】**

```Rust
#![warn(clippy::default_numeric_fallback)]
// 符合
let i: u32 = 10;
let f: f32 = 1.23;
// 符合
let i = 10u32;
let f = 1.23f32;
```


#### P.TYP.01 合理选择算术转换方式

**【描述】**

关键字 as 只用于有损转换无法避免时，否则尽可能利用 From::from 或 TryFrom::try_from 实现无 损转换。

**【正例】**

- 考虑是否必须要做有损的算术转换，如非必要，应使用相同类型来避免转换。

```Rust
let a: f64 = 64.0;

let b: f64 = a; // 符合
```

**【正例】**

- 在将数据从低位转为高位时，建议使用 from 转换，代表无损转换：

```Rust
let c: u16 = 512;
 
let p = u64::from(c); // 符合
```

**【正例】**

- 在必须做数据有损的算术转换但不希望数据截断的情况下，使用 try_from 保证数据准确性：

```Rust
let b: isize = 512;

let y: Result<i8, _> = i8::try_from(b); // 符合
```

**【例外】**

- 在必须做数据有损的算术转换但业务场景允许数据截断，或无法通过 from / try_from 显式转换 时，可继续使用类型强转：

```Rust
let a: f64 = 64.0;
let b: i64 = 100000;

let _: f32 = a as f32; let _: i16 = b as i16;
```


#### G.TYP.BOL.01 不宜使用块表达式作为控制表达式

**【级别】** 要求

**【描述】**

使用块表达式作为控制表达式，易引入带副作用的函数调用，导致程序运行逻辑和预期不符，代码的可 读性也同步变差。

Rust中的控制表达式包括：

- if语句
- while语句
- match语句

**【反例】**

```Rust
// 不符合：可读性差，条件表达式容易被误认为是执行分支
if {
  let filename = get_input_from_user();
  let cur_dir = std::env::current_dir().unwrap(); let filepath = cur_dir.join(filename);
  std::fs::OpenOptions::new() 
    .create_new(true)
    .open(filepath) 
    .is_ok()
} {
    // ...
}
```

**【正例】**

```Rust
fn create_new_user_file() -> bool {
  let filename = get_input_from_user();
  let cur_dir = std::env::current_dir().unwrap(); let filepath = cur_dir.join(filename);
  std::fs::OpenOptions::new() 
    .create_new(true)
    .open(filepath) 
    .is_ok()
}

// 符合
if create_new_user_file() {
    // ...
}
```

**【反例】**

```Rust
fn main() {
  // 不符合：使用块表达式
  match {
    println!("side effect"); std::env::var("PATH")
  } {
    Ok(_) => (), 
    _ => (),
  } 
}
```

**【正例】**

```Rust
fn main() {
  // 符合
  match std::env::var("PATH") {
    Ok(_) => (), 
    _ => (),
  } 
}
```


#### G.TYP.BOL.02 不宜将布尔变量和布尔字面量进行比较

**【级别】** 建议

**【描述】**

条件表达式中类似 `x == true` , `x != true` , `x < true` , `x > false` 等是非必要的表达方式，宜直接使用布尔变量, 如 `x` , `!x` 。

**【反例】**

```Rust
// 不符合
if x == true {}  
if y == false {}

assert_eq!("a".is_empty(), false); 
assert_ne!("a".is_empty(), true);
```

**【正例】**

```Rust
// 符合
if x {}  
if !y {}

assert!(!"a".is_empty());
```


#### G.TYP.BOL.03 && 和 || 操作符的右侧操作中不应包含副作用

**【级别】** 建议

**【描述】**

逻辑与（&&）、逻辑或（||）表达式中的右侧操作是否被执行，取决于左操作的求值结果，当左操作的 求值结果可以得出整个逻辑表达式的结果时，不会再计算右操作的结果。

如果右操作包含副作用，则不能确定是否确实发生了副作用。

**【说明】**

副作用是指对执行状态产生影响，包括：修改对象、修改文件等。在某些情况下副作用会给程序带来不 必要的麻烦，降低程序的可读性，其产生的错误也难以查找。

**【反例】**

```Rust
// 不符合
if flag > 0 || fs::remove_file("example.txt").is_ok() {
  // ...
}
```

【正例】

```Rust
// 符合
if flag > 0 {
  // ...
} else {
    let removed = fs::remove_file("example.txt").is_ok(); 
    if removed {
    // ...
  } 
}
```


#### G.TYP.CHR.01 整数到字符的类型转换应注意有效范围

【级别】 建议

【描述】

在 Rust 中，字符类型 `char` 本质上是一个 `Unicode` 码点(标量值) ，4 个字节大小。 `char` 码点的有效 范围为 `[0, 0xD7FF]` 和 `[0xE000, 0x10FFFF] `。

`u8` 类型可以直接转换为 `char` ，而 `u32` 类型转换为 `char` 是可能失败的。

【正例】

```Rust
let c = char::from(97u8); assert_eq!(c, 'a');

let c = char::from_u32(0x110000); assert_eq!(c, Some('❤ '));

let c = char::from_u32(0xDE01); assert_eq!(c, None);
```


#### G.TYP.INT.01 整数类型的算数运算

**【级别】** 建议

**【描述】**

使用整数计算时需结合场景和业务考虑，如发生溢出、回绕或截断时，是否会产生未定义行为或不符合 预期的结果，比如在计算时间或数组索引时发生整数溢出会导致错误结果或 Panic。

在 Rust 标准库中，提供 `add` / `checked_add` / `saturating_add` / `overflowing_add` / `wrapping_add` 等系列方法，可结合场合选择合适的方法：

- `checked_*` 系列函数返回 Option ，一旦发生溢出则返回 None。
- `saturating_*` 系列函数返回类型是整数，如果溢出，则给出该类型可表示范围的“最大/最小”值。
- `wrapping_*` 系列函数则是直接抛弃已经溢出的最高位，将剩下的部分返回。即返回二进制补码结果。
- `overflowing_*` 系列函数返回二进制补码结果以及指示是否发生溢出的布尔值。

**【说明】**

Rust 编译器在编译时默认没有溢出检查（可通过编译参数来引入），但在运行时会有 Rust 内置 `lint (# [deny(arithmetic_overflow)])`来检查，如果有溢出会 Panic。

无符号整数使用时要注意回绕(wraparound) ，不同整数类型转换时需注意截断。

整数溢出（Overflow）：是指在计算机中使用固定位数的二进制来表示整数时，当进行加减乘等运算 时，如果得到的结果超出了该类型的范围，发生溢出现象。例如，如果使用 u8 类型的整数表示范围为 0 - 255 的无符号整数，当进行加法运算时，如果两个加数之和超过了 255，就会发生溢出现象，结果会变 为 0 - 254 范围内的某个值，而不是正确的结果。

整数回绕（Wraparound）：是指当一个整数超出了其类型的取值范围时，其值会被回绕到类型的取值 范围内的最小值或最大值。例如，在使用 u8 类型表示无符号整数时，如果对一个值为 255 的整数进行 加 1 操作，得到的结果会变为 0，因为 0 是 u8 类型的最小值，此时发生了回绕现象。

**【反例】**

```Rust
#![warn(clippy::integer_arithmetic)]

// 不符合
assert_eq!((-5i32).abs(), 5); assert_eq!(100i32+1, 101);

fn test_integer_overflow() {
  // 不符合：这种写法，Rust 编译器不检查，但 Clippy 可以检查到 // debug 模式，发生运行时 panic
  // release 模式，输出 x = 0
  let mut x: u8 = 255; x += 1;
  println!("x={}", x); 
}
```

**【正例】**

```Rust
#![warn(clippy::integer_arithmetic)]

// 符合
assert_eq!((-5i32).checked_abs(), Some(5)); 
assert_eq!(100i32.saturating_add(1), 101);

fn test_integer_overflow() { let x: u8 = 255;
  // 符合：输出 x = 0
  println!("x={}", x.wrapping_add(1));
  // 符合：输出 x = 255
  println!("x={}", x.saturating_add(1)); 
}
```

**【反例】**

```Rust
#![feature(unchecked_math)]
// 不符合：不安全的移位操作
fn main() {
  let a: u8 = 1;
  let a_shl = unsafe { a.unchecked_shl(8) }; 
  println!("{a_shl}"); // 输出：1
}
```

**【正例】**

```Rust
// 符合：使用安全的移位操作
fn main() {
  let a: u8 = 1;
  let a_shl = a.wrapping_shl(8);
  println!("{a_shl}"); // 输出：0 
}
```


#### G.TYP.SLC.01 宜使用迭代器而非下标遍历切片和数组

**【级别】** 建议

**【描述】**

在 for 循环中使用索引下标访问有可能导致边界错误。 遍历切片和数组时，宜使用迭代器来避免索引访问。

**【反例】**

```Rust
let list: &[&str] = &["aa", "bb", "cc"];

// 不符合：人工计算长度选择范围很可能会出错
for i in 0..list.len() { 
  let item = list[i];
  // ...
}
```

【正例】

```Rust
let list: &[&str] = &["aa", "bb", "cc"];

// 符合
for item in list {
  // ...
}
```


#### G.TYP.SLC.02 应保证数组索引在有效范围内

**【级别】** 要求

**【描述】**

RUST 的数组/切片越界访问不会出现内存被越界读写的后果，但会导致 `panic`。使用索引访问数组的时 候需要保证索引值在有效范围。

建议：

- 如果基于单个索引获取数组元素，需提前校验索引有效；或者使用 `get` 方法，越界时返回 None 。
- 如果需要获取一个切片，需要保证切片的起始索引地址在有效范围内，否则会导致 `panic`。

**【反例】**

```Rust
fn main() {
  let arr = [1, 2, 3, 4, 5];
  // 不符合：越界访问
  let _ = arr[5]; 
}
```

**【正例】**

```Rust
fn main() {
  fn get_index() -> usize {
    5
  }

let arr = [1, 2, 3, 4, 5]; let index = get_index();
// 符合：校验索引是否越界
if index >= arr.len() {
    // 错误处理示例，此处产品应根据业务场景，选择合适的处理方式
    panic!("index out of bounds"); 
  }

  let _ = arr[index]; 
}
```


#### G.TYP.SCT.01 外部使用的自定义类型宜实现常见 trait

**【级别】** 建议

**【描述】**

为外部使用的自定义类型 API 实现一些常见的公共 trait 有利于代码持续演进，并且提高可维护性。

建议实现的公共 trait 包括： `Debug` , `Default` , `Clone` , `PartialEq` , `PartialOrd` , `Hash` , `Eq` , `Ord` 。

【正例】

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
pub MyData {	
  value: i32,
}
```


#### G.TYP.STR.01 在实现Display trait时不应调用to_string()方法

**【级别】** 要求

**【描述】**

对某一类型实现 [Display](https://doc.rust-lang.org/std/fmt/trait.Display.html)trait 的同时会自动为其实现 `ToString` trait。此时若调用 `to_string` 的 话，将会造成无限递归以及栈溢出。

【反例】

```Rust
use std::fmt;

struct Structure(i32);
impl fmt::Display for Structure {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result { 
    write!(f, "{}", self.to_string()) // 不符合
  } 
}
```

【正例】

```Rust
use std::fmt;

struct Structure(i32);
impl fmt::Display for Structure {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "{}", self.0) // 符合
  } 
}
```


#### G.TYP.PTR.01 Rc Arc Box 不应相互嵌套使用

**【级别】** 建议

**【描述】**

不合理的智能指针嵌套使用容易造成多级指针引用，导致内存访问效率不佳。 建议使用 `Rc<T>` 和 `Arc<T>` 来替代 `Rc<Box<T>>` 和 `Arc<Box<T>>` 类型。

**【反例】**

```Rust
use std::rc::Rc; 
fn main() {
  let val = Rc::new(Box::new(0i32)); // 不符合 println!("{val:?}");
}
```

**【正例】**

```Rust
use std::rc::Rc; 
fn main() {
  let val = Rc::new(0i32); // 符合 println!("{val:?}");
}
```


## 3.3 控制流程


#### G.CTF.01 宜优先使用模式匹配而非判断后再取值

**【级别】** 建议

**【描述】**

应优先使用模式匹配方式，而不是通过 if 判断值是否相等，这样代码会更简洁。

**【反例】**

```Rust
let opt: Option<_> = ...;

// 不符合
if opt.is_some() {
  let value = opt.unwrap();
  ...
}

// 不符合
let list: &[f32] = ...;

if !list.is_empty() {  
  let first = list[0];
  ...
}
```


#### G.CTF.02 在 Match 分支的 Guard 语句中不要使用带有副作用的条件表达式

**【级别】** 建议

**【描述】**

在 match 分支中， 匹配几次就会执行 Guard 几次。 Guard 的条件表达式不应该携带副作用。

**【说明】**

副作用是指对执行状态产生影响，包括：修改对象、修改文件等。在某些情况下副作用会给程序带来不 必要的麻烦，其产生的错误十分难以查找，并降低程序的可读性。

**【反例】**

```Rust
use std::cell::Cell; 

fn main() {
  let i: Cell<i32> = Cell::new(0); 
  let opt = Some(1);
  // 不符合：代码最终会输出两次 "add 1"
  match opt {
    // 这个条件表达式带有副作用：打印并修改了变量 i 内部的值，因为匹配两次，所以会执行两次
    Some(1) | Some(_) if condition(&i) => {} 
    _ => {}
  }
  assert_eq!(i.get(), 2); 
}

// 带有副作用，匹配时多次执行可能会出现预想不到的逻辑问题
fn condition(i: &Cell<i32>) -> bool { 
  println!("add 1");
  i.set(i.get() + 1); // 如果函数被执行两次，这里也会被设置两次
  false
}
```

**【正例】**

```Rust
use std::cell::Cell; 

fn main() {
  let i: Cell<i32> = Cell::new(0); let opt = Some(1);
  // 符合：代码最终输出一次 "add 1"
  match opt {
    // 注意有副作用的函数被移动到了Guard语句之后调用以作判断
    Some(1) => if condition(&i) {} 
    Some(_) => if condition(&i) 
    {} _ => {}
  }
  assert_eq!(i.get(), 1); 
}

// 带有副作用，匹配时多次执行可能会出现预想不到的逻辑问题
fn condition(i: &Cell<i32>) -> bool { println!("add 1");
  i.set(i.get() + 1); // 如果函数被执行两次，这里也会被设置两次
  false
}
```


#### G.CTF.03 循环或递归必须安全退出

**【级别】** 建议

**【描述】**

在应用程序中，一个重复提供服务的逻辑循环，应当包含相应的退出机制，并且将资源正确释放后安全退出。

循环必须安全退出，递归也必须安全结束。

**【反例】**

```Rust
fn main() {
  let mut counter = 0;

  loop {
    counter += 1;
    // 不符合：循环无退出条件
  } 
}
```

**【正例】**

```Rust
fn main() {
  let mut counter = 0;

  loop {
    counter += 1;
    // 符合：循环包含退出条件
    if should_exit(counter) { 
      break;
    }
  } 
}

fn should_exit(count: u32) -> bool { 
  count >= 5
}
```


#### G.EXP.01 避免复合条件表达式永远为 true 或 false

**【级别】** 建议

**【描述】**

复合条件表达式永远为 `true` 或 `false` 通常是因为代码编写错误。

**【反例】**

```Rust
let x = 2;
// 不符合：该表达式会永远是 false
if (x & 1 == 2) { }

let x = 2;
if (x & 3) < 4 { // 不符合：判断表达式结果永远为 true 
  // do something
}

let y = 0;
if (x < y) && false { // 不符合：判断表达式结果永远为 false 
  // do something
}
```

**【正例】**

```Rust
let x = 2;
// 符合
if x == 2 { }
```


#### G.EXP.02 用括号明确表达式的操作顺序

**【级别】** 建议

**【描述】**

宜使用括号强调表达式的优先级顺序，增加可读性，防止开发者因不熟悉默认的优先级而导致错误。

**【反例】**

```Rust
1 << 2 + 3 // 不符合  
-1i32.abs() // 不符合
```

**【正例】**

```Rust
(1 << 2) + 3  // 符合 
(-1i32).abs() // 符合
```


## 3.4 函数设计


#### G.FUD.01 内存拷贝代价大的参数应按地址传递

**【级别】** 建议

**【描述】**

Rust 中的所有权转移也意味着一次栈内存的拷贝，因此所有非引用和指针类型的参数传递都是值传递， 需要考虑内存拷贝的代价，设计合理的参数类型：

- 对于输入参数，拷贝代价小的类型使用值传递，拷贝代价大的类型传递只读引用。
- 对于返回类型，如果拷贝代价大，宜设计传递可写借用参数并返回它。

**【反例】**

输入参数

```Rust
// 不符合
fn func(val: [i32; 1000]);
```

**【正例】**

输入参数

```Rust
// 符合
fn func(val: &[i32]);
```

**【反例】**

返回类型

```Rust
// 不符合：后续Self类型功能扩展，很可能导致栈内存拷贝代价增大
fn set_property(self, name: &str, value: &str) -> Self;
```

**【正例】**

返回类型

```Rust
// 符合：传递可写借用参数并返回它，适合链式调用的场景
fn set_property(&mut self, name: &str, value: &str) -> &mut Self;
```


#### G.FUD.02 可能出错的函数，应保证用户不可忽略返回值

**【级别】** 建议

**【描述】**

RUST 中有以下方法可以在用户忽略函数返回值的时候出现编译告警：

- 返回值类型是 `Result`
- 函数定义增加 #[must_use] 修饰
- 函数返回的专有数据类型增加 #[must_use] 修饰

**【正例】**

输入参数

```Rust
#[must_use]
fn foo() -> i32;

#[must_use]
struct MyError(i32, i32); 
fn foo() -> MyError;
```


## 3.5 Trait


#### G.TRA.01 在实现相等性、排序和散列类 trait 时，应保证一致性

**【级别】** 要求

**【描述】**

在手动实现 `Hash` 和 `PartialEq/ Eq` 时必须要满足等式 `(k1 == k2) == (hash(k1) == hash(k2))`，即当 `k1` 和 `k2` 相等时， `hash(k1)` 也应该和 `hash(k2)` 相等。

在手动实现 `PartialOrd` 和 `Ord` 时则必须要满足等式 `k1.cmp(&k2) == k1.partial_cmp(&k2).unwrap()` 。

若类型实现了 `Borrow` trait，也应保证上述等式对借用后的类型同样成立，即:

`hash(k1) == hash(k2) 等效于 hash(k1.borrow()) == hash(k2.borrow())` ,

且 `k1.borrow().cmp(&k2.borrow()) 等效于 k1.borrow().partial_cmp(&k2.borrow()).unwrap()` ;

通常情况下，可以通过 `derive` 属性宏来自动实现这些 traits。但相等性，排序和散列在不同应用场景 可能存在不同的定义，又或是因为依赖的某种类型未实现其中一个 trait 而不能使用 `derive` 时，就需 要开发者手动实现以上 traits，这时就要注意保持一致性。

一旦不一致，可能导致出现非预期的函数行为。

**【反例】**

```Rust
// 不符合，雇员(`Employee`)类型包括 uid 及 name，通过宏自动实现的 `Hash` 会同时考虑所有成 员变量，
// 但是手动实现的 `PartialEq` trait 仅考虑到了 uid，这将会导致结果不一致。
// 因为当两个雇员(假设为 a 和 b)的 name 不同，但 uid 相同时 , `a == b` 为 true, 但`hash(a)` 不等于 `hash(b)`

#[derive(Hash, Eq)]
struct Employee { 
  uid: u32,
  name: String,
}

impl PartialEq for Employee {
  fn eq(&self, other: &Self) -> bool {
    self.uid == other.uid
  } 
}
```

【正例】

```Rust
// 符合，都使用 `derive` 自动实现

#[derive(Hash, PartialEq, Eq)] 
struct Employee {
  uid: u32,
  name: String, 
}

或

use std::hash::{Hash, Hasher};

#[derive(Eq)]
struct Employee { 
  uid: u32,
  name: String,
}

// 符合，都使用手动实现的方式且保证一致性， // 与 `hash` 一样都使用 uid 作为唯一条件
impl PartialEq for Employee {
  fn eq(&self, other: &Self) -> bool { 
    self.uid == other.uid
  } 
}

impl Hash for Employee {
  fn hash<H: Hasher>(&self, state: &mut H) { 
    self.uid.hash(state)
  } 
}
```

【反例】

```Rust
#[derive(Ord, PartialEq, Eq)] 
struct Employee {
  uid: u32,
  name: String, 
}

// 不符合 , `Ord` 和 `PartialOrd` 的实现不一致
impl PartialOrd for Employee {
fn partial_cmp(&self, other: &Employee) -> Option<Ordering> { 
    Some(self.uid.cmp(&other.uid))
  } 
}
```

【正例】

```Rust
// 符合 , 都使用 `derive` 自动实现
#[derive(PartialOrd, Ord, PartialEq, Eq)] 
struct Employee {
  uid: u32,
  name: String, 
}
#[derive(PartialEq, Eq)] 
struct Employee {
  uid: u32,
  name: String, 
}

// 符合 , 都使用手动实现的方式且保证一致性
impl PartialOrd for Employee {
fn partial_cmp(&self, other: &Employee) -> Option<Ordering> { 
    Some(self.uid.cmp(&other.uid))
  } 
}

impl Ord for Employee {
  fn cmp(&self, other: &Self) -> Ordering { 
    self.uid.cmp(&other.uid)
  } 
}
```


#### G.TRA.02 宜实现 From 而不是 Into

**【级别】** 要求

**【描述】**

对某一类型实现 [Display](https://doc.rust-lang.org/std/fmt/trait.Display.html)trait 的同时会自动为其实现 `ToString` trait。此时若调用 `to_string` 的 话，将会造成无限递归以及栈溢出。

【反例】

```Rust
use std::fmt;

struct Structure(i32);
impl fmt::Display for Structure {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result { 
    write!(f, "{}", self.to_string()) // 不符合
  } 
}
```

【正例】

```Rust
use std::fmt;

struct Structure(i32);
impl fmt::Display for Structure {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "{}", self.0) // 符合
  } 
}
```


## 3.6 错误处理


#### G.ERR.01 应正确处理返回类型 Option 和 Result

**【级别】** 建议

**【描述】**

对于返回类型为 Option 或 Result 的函数，应使用不会导致 panic 的方法显示处理返回的 None / Err 错误值。

处理方法包括：

- 使用 match 语句
- 使用 unwrap_or / unwrap_or_else / unwrap_or_default 等函数 例外：如下两种场景可直接调用 unwrap / expect ：
- 调用返回 Option / Result 函数之前通过输入参数的提前校验，可以保证返回值一定不会是 None / Err 。
- 业务模块的设计要求对于返回 None / Err 的情况 panic，依赖更高层的可靠性机制检测故障并实现故障恢复。

**【正例】**

```Rust
// ...

pub fn main() {
  // 符合，针对不同情况做出应对措施，有利于定位错误
  let config_content = match fs::read_to_string(CONFIG_PATH) { 
    Ok(content) => content,
    Err(e) => {
      use std::io::ErrorKind; match e.kind() {
        ErrorKind::NotFound => {
          // 处理文件找不到的情况
        }
        ErrorKind::PermissionDenied => {
          // 处理文件权限不够的情况
        }
        _ => {
          // 处理其它错误情况
        }
      }
      return;
    } 
  };

  // ...
}
```


## 3.7 模块


#### G.MOD.01 导入模块中的类型或函数时，应避免直接使用通配符

**【级别】** 建议

**【描述】**

使用通配符导入会污染命名空间，比如导入相同命名的函数或类型。为了避免这种情况，使用导入的类 型或函数时需要带模块名前缀。有个例外，当一个 crate 提供了 prelude 时，可以用 `prelude::*` 。

对于标准库中，很多人都熟知的类型 ，比如 `Arc/ Rc/ Cell/ HashMap` 等 ， 可以导入它们直接使用。

但是对于可能引起困惑的函数，比如 `std::ptr::replace` 和 `std::mem::replace` ，在使用它们的时 候，就必须得带上模块前缀。

使用一些第三方库中定义的类型或函数，也建议带上 crate 或模块前缀。如果太长的话，可以考虑使用 `as` 或 `type` 来定义别名。

以上考虑都是为了增强代码的可读性、可维护性。

**【反例】**

```Rust
use std::sync::Arc; 
use std::ptr::*;

fn main() {
  let foo = Arc::new(vec![1.0, 2.0, 3.0]); // 直接使用 Arc let a = foo.clone();

  let mut rust = vec!['b', 'u', 's', 't'];

  let b = unsafe {
    // 不符合
    // 不知道 `replace` 是来自 `std::ptr` 还是 `std::mem` 模块
    replace(&mut rust[0], 'r') 
  };
}
```

**【正例】**

```Rust
use std::sync::Arc; 
use std::ptr;

fn main() {
let foo = Arc::new(vec![1.0, 2.0, 3.0];
let a = foo.clone();

let mut rust = vec!['b', 'u', 's', 't'];

let b = unsafe {
    // 符合
    // 需要带上 ptr 前缀，因为 `std::mem::replace` 也有同样的效果
    ptr::replace(&mut rust[0] 
  };
}
```


## 3.8 宏


#### G.MAC.DCL.01 编写宏匹配规则时，宜遵从匹配范围从小至大的顺序

**【级别】** 建议

**【描述】**

因为声明宏中，是按规则的编写顺序来匹配的。当第一个规则被匹配到，后面的规则将永远不会匹配 到。

所以，编写声明宏规则时，需要先写匹配范围最小的，最具体的规则，然后逐步编写匹配范围更广泛的规则。

参考 [The Little Book of Rust Macros](https://danielkeep.github.io/tlborm/book/mbe-min-captures-and-expansion-redux.html)。

**【反例】**

```Rust
// 不符合， `$($other:tt)*` 匹配条件在这个宏定义中匹配范围最大却放在了第一位。
macro_rules! match_tokens {
  // 可能会导致调用时代码被错误地匹配到了这一条规则。
  ($($other:tt)*) => {"got something else"}; 
  ($a:tt + $b:tt) => {"got an addition"};
  ($i:ident) => {"got an identifier"}; 
}

// 以下的三种语法： `caravan`, `3 + 6`, `5` 均被匹配为该宏的同一条规则
assert_eq!(
    format!("{},{},{}",
      match_tokens!(caravan), 
      match_tokens!(3 + 6),
      match_tokens!(5)
  ),
  "got something else,got something else,got something else"
);
```

**【正例】**

```Rust
// 符合，匹配范围从小到大
macro_rules! match_tokens {
  ($a:tt + $b:tt) => {"got an addition"}; 
  ($i:ident) => {"got an identifier"};
  ($($other:tt)*) => {"got something else"}; 
}

// 匹配合理，宏将 `caravan` 识别为 identifier, `3 + 6` 识别为 addition, `5` 识别为 something else
assert_eq!(
  format!("{},{},{}",
    match_tokens!(caravan), 
    match_tokens!(3 + 6),
    match_tokens!(5)
  ),
  "got an identifier,got an addition,got something else"
);
```


#### P.MAC.DCL.01 定义和使用声明宏时，应保证卫生性

**【描述】**

- 标识符（包括变量、泛型、生命周期名称）建议作为宏的参数传递，而不是在宏定义时硬编码，这 样可以在提升代码可维护性的同时，让宏的适用范围更广泛。
- 在宏内引用宏外部的 [item](https://doc.rust-lang.org/reference/items.html)，如函数或宏时应使用绝对路径：

1. 当宏定义和宏内引用的 item 在同一个 crate 中定义时，应该使用 $crate 元变量来指代当前 被调用宏的路径。
2. 当宏定义和宏内引用的 item 在不同的 crate 中定义时，用 crate 名来引用。

**【说明】**

声明宏是半卫生宏，变量标识符在宏内外是隔离的，不会产生命名冲突，符合卫生性。但是在宏内定义 的泛型及生命周期标识符的做法是不卫生的。

卫生宏定义请参考 [卫生宏 WiKi](https://zh.wikipedia.org/zh-hans/%E5%8D%AB%E7%94%9F%E5%AE%8F) 。

**【反例】**

```Rust
// 使用宏为带生命周期的类型实现 impl
macro_rules! impl_for { ($name:ty) => {
// 不符合，这里使用的 'a 是宏内部定义
  impl<'a> $name {
      fn get(&self) -> i32 { 
        *self.0
      } 
    }
  }; 
}

struct Foo<'a>(&'a i32);
// 程序能正常编译运行，但宏内外的 'a 被共用，不卫生，也不利于可维护性
impl_for!(Foo<'a>);
// ...

struct Bar<'b>(&'b i32);
// 但如果要用相同的宏为生命周期为 'b 的类型实现 impl 时 , 
// 编译会失败 : use of undeclared lifetime name `'b`
impl_for!(Bar<'b>);
```

**【正例】**

```Rust
// 使用宏为带生命周期的类型实现 FooTrait
macro_rules! impl_for {
  // 符合，这里不直接使用宏内部定义的 'a ，而是通过 $lifetime 从外部传入的， 
  // 从而避免因宏不卫生引发的问题
  ($name:ty, $lifetime:tt) => {
    impl<$lifetime> for $name {
      fn get(&self) -> i32 { 
        *self.0
      } 
    }
  }; 
}

struct Foo<'a>(&'a	i32);
impl_for!(Foo<'a>,	'a); // 这里的 'a 是从宏外部传入到宏内
struct Bar<'b>(&'b	i32);
impl_for!(Bar<'b>,	'b); // 同时，宏的适用范围更加广泛
```

**【反例】**

```Rust
// 宏定义在 crate a 中
#[macro_export]
macro_rules! helped {
  () => { helper!() } // This might lead to an error due to 'helper' not being in scope.
}

#[macro_export]
macro_rules! helper { 
  () => { () }
}

// 在 crate b 中使用 crate a 的两个宏
// 注意： `helper_macro::helper` 并没有导入进来
use helper_macro::helped;

fn unit() {
  // Error! 这个宏会出现问题，因为 helped 宏内部调用的 helper 宏的路径会被编译器认为是在当 前的 crate b 中
  helped!(); 
}
```

**【正例】**

```Rust
// 宏定义在 crate a 中
#[macro_export]
macro_rules! helped {
  () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
  () => { () }
}

// 在crate b 中使用 crate a 的两个宏
// 注意： `helper_macro::helper` 并没有导入进来
use helper_macro::helped;

fn unit() {
  // OK! 这个宏能运行通过，因为 `$crate` 正确地展开成 `helper_macro` 所在的 crate a 的 路径（而不是使用者 crate b 的路径）
  helped!();
}
```


#### P.MAC.PRO.01 定义和使用过程宏时，应保证卫生性

**【描述】**

过程宏在展开时，有时可能会直接引用输入代码中的标识符（变量、函数名等）。如果过程宏引入了与周围代码冲突的标识符，就会导致不卫生的情况。为了避免这种情况，通常需要对输入标识符进行重命名。

要避免不卫生的情况，包括以下常用方法：

a. 确保宏生成代码的符号与外部有差异，如增加`_`前缀。

b. 对库中程序项使用绝对路径.

**【反例】**

```Rust
// crate guide
#[proc_macro]
pub fn my_macro(input: TokenStream) -> TokenStream { 
  let expanded = quote! {
    // 不符合：直接使用外部的变量 x
    x = 999;
    // 不符合: 宏内定义的变量 local 被外部使用
    let local = "hello";

    // ...
  };
  expanded.into() 
}

// crate b
fn main() {
  let mut x = 0;  
  let local = "";

  guide::my_macro!();

  assert_eq!(x, 999);
  assert_eq!(local, "hello"); 
}
```

**【正例】**

```Rust
#[proc_macro]
pub fn my_macro(input: TokenStream) -> TokenStream { 
  let expanded = quote! {
    // 符合：方法 a. 宏内变量增加 `    ` 前缀
    let _x = 0;
    let _local = "hello";

    // ...
  };
  expanded.into() 
}
```

**【正例】**

过程宏生成的代码宜使用绝对路径，防止命名冲突产生意想不到的后果。 可以配合使用 `#![no_implicit_prelude]` 属性来验证标识符是否都使用了绝对路径：

```Rust
#![no_implicit_prelude]

#[derive(MyMacro)] 
struct A;
```

```Rust
// 符合：方法 b. 使用绝对路径
quote!(::std::string::ToString::to_string(a))
```

```Rust
quote! {{
  // 符合：方法 b. 使用绝对路径
  use ::std::string::ToString;
  a.to_string() 
}}
```


#### G.MAC.PRO.011 不应通过过程宏将 unsafe 代码包装为 safe 代码

**【级别】** 要求

**【描述】**

不要通过过程宏将 unsafe 代码包装为safe 代码，以此来规避 Rust 静态分析检查。

**【反例】**

```Rust
// 用 RUST 的过程宏包装 libc 的函数 printf 来打印一个 C 字符串
// 这个宏的输入参数必须是一个有效的 C 字符串指针，整个宏的使用是不安全的， // 但是因为过程宏中包括了 unsafe，导致过程宏看起来像 safe 调用
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

【正例】

```Rust
// 用 RUST 的过程宏包装 libc 的函数 printf 来打印一个 C 字符串
// 这个宏的输入参数必须是一个有效的 C 字符串指针，同时也是调用外部函数 // 必须在 unsafe 代码块中使用
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

【相关讨论】

- [RUSTSEC-2020-0011](https://rustsec.org/advisories/RUSTSEC-2020-0011.html)
- [https://github.com/RustSec/advisory-db/issues/275](https://github.com/RustSec/advisory-db/issues/275)
- [https://github.com/rust-lang/unsafe-code-guidelines/issues/278](https://github.com/rust-lang/unsafe-code-guidelines/issues/278)


# 4 安全实践


## 4.1 Unsafe Rust


#### P.SAF.UNS.01 Unsafe 代码应仅在需要的场景下使用

**【描述】**

- 不要为了逃避编译器安全检查而随便滥用 Unsafe Rust，否则很可能引起未定义行为（UB）。
- 不要为了提升性能而盲目使用 Unsafe Rust。
- 要尽量缩小使用 `unsafe` 块的范围，而不要将多余的代码都包入 `unsafe` 块中。
- 对于 Unsafe 函数，应该显式地标记 `unsafe` ，不要试图通过宏等技巧打破 Unsafe 传播链条。 . 不要通过 Unsafe 将不可变引用/指针手工转换为可变引用/指针。这类场景考虑使用内部可变性容 器 `Cell<T>` 或 `RefCell<T>` 。

**【反例】**

```Rust
// 该函数为滥用 unsafe 来跳过 Rust 借用检查 
// 强行返回本地变量的引用，最终引起未定义行为
fn abuse_unsafe_return_local_ref<'a>() -> &'a String { 
  let s = "hello".to_string();
  let ptr s addr = &s as *const String as usize; 
  unsafe { 
    &*(ptr s addr as *const String) 
  }
}

fn main() {
  let s = abuse_unsafe_return_local_ref(); // error: Undefined Behavior:encountered a dangling reference (use-after-free)
}
```

**【反例】**

```Rust
// 不符合：使用 unsafe 通过不可变借用修改变量
fn main() {
  unsafe fn increase_num(r: &i32) {
  *(r as *const i32 as *mut i32) += 1;
  }

  let num = 0; unsafe {
  increase_num(&num); }
  println!("{num}"); 
}
```


#### G.SAF.UNS.01 禁止将外部可控数据作为加载动态库的参数

**【级别】** 要求

**【描述】**

禁止将从外部获取的数据作为加载动态库的参数，否则可能加载攻击者事先预制的模块。 如果要加载动态库，可以采用如下措施之一：

- 使用白名单机制，确保加载函数的参数不受任何外来数据的影响。 使用数字签名机制保护要加载的动态库，充分保证其完整性。
- 在设备本地加载的动态库通过权限与访问控制措施保证了本身安全性后，通过特定目录自动被程序 加载。
- 在设备本地的配置文件通过权限与访问控制措施保证了本身安全性后，自动加载配置文件中指定的 动态库。

**【反例】**

示例参考： [rust_libloading 例子](https://github.com/nagisa/rust_libloading/blob/master/tests/windows.rs)

```Rust
// 不符合
extern crate libloading; 
use std::io::Read;

fn main() {
  let mut f = std::fs::File::open("tests/nagisa64.txt").unwrap(); 
  let mut name = String::new();
  // 不符合：从不受信任的文件读取用于加载的动态库名称
  f.read_to_string(&mut name).unwrap();

  unsafe {
    let lib = libloading::Library::new(&name); 
    if lib.is_err() {
      println!("can't open {}", name); 
    }
  } 
}
```

**【正例】**

```Rust
extern crate libloading; 
use std::ffi::OsStr;

fn main() -> Result<(), Box<dyn std::error::Error>> {
  fn whitelist_verify<P: AsRef<OsStr>>(lib_path: P) -> Result<bool, String> {
    // 检查传入路径是否在白名单内 // ......
    return Result::Ok(false); 
  }

  let name = "nagisa64.dll";
  if whitelist_verify(name).is_ok() { 
    return;
  }
  unsafe {
    let lib = libloading::Library::new(name); 
    if lib.is_err() {
      println!("can't open {}", name); 
    }
  } 
}
```


## 4.2 FFI


#### P.SAF.FFI.01 在 FFI 边界上传递数据应注意内存安全管理

**【描述】** 在 FFI 边界上传递数据要遵循 “内存由创建方来负责释放” 的基本原则来保证内存的安全管理。

否则可能会遇到以下问题：

- 所有权问题： Rust 使用所有权系统管理内存，而 C 函数通常不了解 Rust 的所有权规则。如果将 Rust 字符串传递给 C 函数，需要确保内存管理的正确性，防止出现悬垂指针或者内存泄漏等问题。可以使用合适的 Rust-C 绑定库或者 FFI（Foreign Function Interface）来处理内存管理问题。
- 生命周期问题： Rust 字符串的生命周期通常是由 Rust 的借用规则确定的。如果需要将字符串传递给 C 函数，并且 C 函数需要长时间持有字符串的引用，需要谨慎处理生命周期的问题，以避免引用无效或悬垂引用的情况。


#### P.SAF.FFI.02 应正确处理 FFI 调用的返回值和输出参数

**【描述】**

在 FFI 调用场景下，应该对函数调用结果的错误异常进行正确处理。错误异常的常见形式有错误码返回值和空指针。

**【反例】**

```Rust
/// 模拟一个外部 C 函数
#[no_mangle]
pub extern "C" fn alloc_huge_space() -> *mut libc::c_int {
  unsafe { 
    libc::malloc(u64::MAX.try_into().unwrap()) as *mut libc::c_int 
  } 
}

fn main() {
  let x = alloc_huge_space();
  let mut v = unsafe {
     std::slice::from_raw_parts_mut(x, 100) 
     }; 
  v[0] = 100;
}
```

**【正例】**

```Rust
/// 模拟一个外部 C 函数
#[no_mangle]
pub extern "C" fn alloc_huge_space() -> *mut libc::c_int {
  unsafe { 
    libc::malloc(u64::MAX.try_into().unwrap()) as *mut libc::c_int 
  } 
}

fn main() {
  let x = alloc_huge_space(); 
  if !x.is_null() {
    let mut v = unsafe {
       std::slice::from_raw_parts_mut(x, 100) 
       };
    v[0] = 100;
  } 
}
```


#### P.SAF.FFI.03 应优先选用 RAII 方式管理外部资源

**【描述】**

FFI 编程中关于资源管理，遵循着 “谁申请谁释放” 的原则。因此在 Rust 中调用 FFI 函数创建的资源，也 应该调用相应的 FFI 函数来释放资源。常见的资源类型包括：堆内存、文件、硬件接口等。

使用 RAII 方式管理外部资源，可以有效避免忘记释放或重复释放问题。

**【说明】**

RAII 是指 resource acquisition is initialization，通过对象的构造函数获取资源，并在对象的析构函数 释放资源。

**【正例】**

```Rust
extern "C" {
  fn new_resource() -> *mut Resource;
  fn process_resource(resource: *mut Resource); 
  fn free_resource(resource: *mut Resource);
}

#[repr(C)]
struct CResource(*mut Resource);

impl CResource {
  fn new() -> Self {
    unsafe { Self(new_resource()) } 
  }
  fn process(&self) {
    unsafe { process_resource(self.0) } 
  }
}

impl Drop for CResource { 
  fn drop(&mut self) {
    unsafe { free_resource(self.0) } 
  }
}

fn main() {
  let mut res = CResource::new(); 
  res.process();
}
```


#### G.SAF.FFI.01 FFI 边界应确保字符串参数正确传递和使用

**【级别】** 要求

**【描述】**

Rust 中字符串的表示采取了与 C 语言不同的方式。 Rust 字符串指针为宽指针，其中包含了字符串的起 始地址指针和字符串长度；而 C 语言中字符串指针为窄指针，仅包含起始地址，且字符串必须以 \0 结 尾。因此，不能直接使用 Rust 的 `String` 和 `&str` 类型与 FFI C 函数进行交互。

Rust标准库提供了两种与 C 语言字符串兼容的字符串类型： `CString` 和 `CStr`。从 Rust 向 C 函数传递 字符串时应当向将字符串转换为 `CString` 类型，再用 `.as_ptr()` 方法获取 C 字符串指针。而在 Rust 中读取 C 函数返回的字符串指针，应该通过 `CStr::from_ptr()` 函数将其转换为 CStr 类型再读取。

`CString` 和 `CStr` 类型也提供了与 `String` 和 `&str` 类型之间的相互转换函数。

**【正例】**

```Rust
use std::ffi::CString;
use std::os::raw::c_char;

extern "C" { fn greet(name: *const c_char); }

fn main() {
  let name = "Rust";
  let name = CString::new(name).unwrap(); 
  unsafe { greet(name.as_ptr()) };
}
```


#### G.SAF.FFI.02 在 FFI 调用的场景下，应确保跨边界数据的内存布局兼容

**【级别】** 要求

**【描述】**

FFI 函数传递的任何数据类型，需要保证内存布局一致。对于结构体、枚举和联合体，应该通过添加 `# [repr(C)]` 属性，来避免字段顺序重排，以保证内存布局的兼容。 在处理网络消息收发场景中，常见 1 字节对齐内存布局方式。在 Rust 中可以使用 `#[repr(packed)]` 表示对应的数据。

**【正例】**

```Rust
#[repr(C)]
struct Foo {
  a: libc::c_short, 
  b: libc::c_int,
  c: libc::c_char, 
  d: libc::c_long,
}

#[link(name = "bar")] extern "C" {
  fn bar(foo: *const Foo); 
}

fn main() {
  let foo = Foo { 
    a: 1234,
    b: 12345, 
    c: 123,
    d: 123456, 
  };
  unsafe {
    bar(&foo as *const Foo); 
  }
}
```

**【正例】**

```Rust
#[repr(packed)]  
#[derive(Debug)] 
struct Foo {
  a: libc::c_char, 
  b: libc::c_int,
  c: libc::c_longlong,
}

extern "C" {
  fn new_foo() -> Foo;
}

let foo = unsafe { new_foo() }; 
println!("{foo:?}");
#pragma pack(1)
typedef struct Foo { 
  char a;
  int b;
  long long c; 
} Foo;
#pragma pack()

struct Foo new_foo() {
  struct Foo foo = {'c', 1234, 123456789}; 
  return foo;
}
```


#### G.SAF.FFI.03 禁止调用 C 不可重入函数

**【级别】** 要求

**【描述】**

在多线程条件下调用外部不可重入函数时，有可能会因竞争条件而导致意外的系统状态和不可预测的结 果，或导致拒绝服务和任意代码执行等内存安全问题。

C 语言标准库中常见的不可重入函数包括：

`getenv()` , `strtok()` , `strerror()` , `asctime()` , `ctime()` , `localtime()` , `gmtime()` , `setlocale()` , `localeconv()` , `syslog()` , g`etlogin()` 等。

开发者能在 Unsafe Rust 下直接使用 `extern "C"` 或 `libc` 库调用这些函数。

一般情况下，开发者可将这些函数替换为其可重入版本，否则需要根据情况解决或预防因竞争而产生的问题。


#### G.SAF.FFI.04 禁止使用外部内存不安全函数

**【级别】** 要求

**【描述】**

C 语言标准函数库中，存在一些不安全函数（见下）。使用不当容易造成缓冲区溢出等安全漏洞。 Rust FFI 编程中禁止调用外部 C 不安全函数。

以下列出了部分内存操作类不安全函数：

- 内存拷贝函数： memcpy , wmemcpy , memmove , wmemmove
- 内存初始化函数： memset
- 字符串拷贝函数： strcpy , wcscpy , strncpy , wcsncpy
- 字符串拼接函数： strcat , wcscat , strncat , wcsncat
- 字符串格式化输出函数： sprintf , swprintf , vsprintf , vswprintf , snprintf , vsnprintf
- 字符串格式化输入函数： scanf , wscanf , vscanf , vwscanf , fscanf , fwscanf , vfscanf , vfwscanf , sscanf , swscanf , vsscanf , vswscanf .
- stdin流输入函数： gets

对于上面这些不安全函数，在 Rust 中，一般有功能相同的安全函数可供使用。比如：

- 对于内存拷贝和初始化，可以使用 `std::mem` 模块提供的函数进行处理。
- 对于字符串拼接和格式化输出，可以使用标准库提供的 `format!` 或 `println!` 等宏函数。
- 对于标准输入流读取，可以使用 `std::io::stdin()` 对象的相关方法来处理。


## 4.3 多线程与异步


#### P.SAF.MTH.01 并发场景下应避免数据竞争和死锁

**【描述】**

Safe Rust 多线程同步锁并发情况下有效地防止数据竞争，但是使用无锁编程时，需要选择合理的内存顺 序才能避免数据竞争。

另外，在 Unsafe Rust 下如果需要手工为一些自定义类型实现 Sync 和 Send auto trait 时，也需要严 格确保该类型在多线程下传递不会出现数据竞争风险。

**【正例】**

- 自旋锁中 Acquire / Release 搭配使用的简易示例:

```Rust
use std::sync::Arc;
use std::sync::atomic::{AtomicBool, Ordering}; 
use std::thread;

fn main() {
  let lock = Arc::new(AtomicBool::new(false)); // 这个原子类型的值用来表示 "是否获取 到锁"

  // ... distribute lock to threads somehow ...

  // 线程 A 尝试通过设置为 true 来获取锁 //
  // Acquire 内存顺序:
  // 当与 load 结合使用时，若 load 的值是由具有 Release (或更强) 排序的 store 操作写入 的，则所有后续操作
  // 都将在该 store 之后进行排序。特别是，所有后续 load 都将看到在 store 之前写入的数据。
  while lock.compare_and_swap(false, true, Ordering::Acquire) { }
  // 在循环外，意味着已经拿到了锁！ // ...访问/操作数据 ...
  // 线程 A 完成了数据操作，释放锁。
  // 此处用 Release 内存顺序可以确保线程 B 在获取锁时能看到线程 A 释放了锁（对内存写入 false）
  //
  // Release 内存顺序：
  // 当与 store 结合使用时，所有先前的操作都会在使用 Acquire（或更强）排序的任何 load 此 值之前排序。
  // 特别是，所有先前的写入对执行 Acquire 此值（或更强）load 的线程都可见。
  lock.store(false, Ordering::Release); 
}
```

【正例】

- Unsafe Rust 中实现 auto trait `Send / Sync` 的示例

Rust futures 库中发现的问题，错误的手工 `Send / Sync` 实现破坏了线程安全保证。受影响的版本 中， `MappedMutexGuard` 的 `Send / Sync` 实现只考虑了 `T` 上的差异，而 `MappedMutexGuard` 则取 消了对 U 的引用。当 MutexGuard::map() 中使用的闭包返回与 `T` 无关的 `U` 时，这可能导致安全 Rust 代码中的数据竞争。

这个问题通过修正 `Send / Sync` 的实现，以及在 `MappedMutexGuard` 类型中添加一个 `PhantomData<&'a mut U> `标记来告诉编译器，这个防护也是在 `U` 之上。

```Rust
// CVE-2020-35905: incorrect uses of Send/Sync on Rust's futures
pub struct MappedMutexGuard<'a, T: ?Sized, U: ?Sized> { 
  mutex: &'a Mutex<T>,
  value: *mut U,
  _marker: PhantomData<&'a mut U>, // + 修复代码 
}

impl<'a, T: ?Sized> MutexGuard<'a, T> {
  pub fn map<U: ?Sized, F>(this: Self, f: F)
    -> MappedMutexGuard<'a, T, U>
    where F: FnOnce(&mut T) -> &mut U { 
      let mutex = this.mutex;
      let value = f(unsafe { &mut *this.mutex.value.get() }); 
        mem::forget(this);
        // MappedMutexGuard { mutex, value }
        MappedMutexGuard { mutex, value, _marker: PhantomData } //  + 修复代码
  } 
}

// unsafe impl<T: ?Sized + Send, U: ?Sized> Send
unsafe impl<T: ?Sized + Send, U: ?Sized + Send> Send // + 修复代码 
for MappedMutexGuard<'_, T, U> {}
//unsafe impl<T: ?Sized + Sync, U: ?Sized> Sync
unsafe impl<T: ?Sized + Sync, U: ?Sized + Sync> Sync // + 修复代码 
for MappedMutexGuard<'_, T, U> {}

// PoC: this safe Rust code allows race on reference counter
* MutexGuard::map(guard, |_ | Box::leak(Box::new(Rc::new(true))));
```

**【反例】**

- 在 Unsafe Rust 中手动为自定义类型实现 Sync 的反例：

```Rust
fn main {
  use std::cell::Cell;
  use std::marker::Sync; 
  use std::sync::Arc;

  struct Counter {
    val: Cell<usize>,
  }

  // 不符合：Counter 在多线程共享不是安全的 // 错误的使用 unsafe impl Sync
  unsafe impl Sync for Counter {}

  impl Counter {
    fn new() -> Counter {
      Counter { val: Cell::new(0) } 
    }

    fn increment(&self) {
      self.val.set(self.val.get() + 1); 
    }

    fn get_count(&self) -> usize {
      self.val.get() }
  }

  let counter = Arc::new(Counter::new()); let mut handles = vec![];

  for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = std::thread::spawn(move || {
      for _ in 0..100 {
        counter.increment();
      } 
    });
    handles.push(handle); 
  }

  for handle in handles {
    handle.join().unwrap();
  }

  println!("Counter value: {}", counter.get_count()); 
}
```

【正例】

- 在 Unsafe Rust 中手动为自定义类型实现 Sync 的正例：

```Rust
fn main() {
  use std::marker::Sync;
  use std::sync::{Arc, Mutex};

  struct Counter {
    count: Mutex<usize>, 
  }

  // 符合：Counter 在多线程共享是安全的
  unsafe impl Sync for Counter {}

  impl Counter {
    fn new() -> Counter { 
      Counter {count: Mutex::new(0), }
    }

    fn increment(&self) {
    let mut count = self.count.lock().unwrap(); *count += 1;
    }

    fn get_count(&self) -> usize {  
      *self.count.lock().unwrap()
    } 
  }

  let counter = Arc::new(Counter::new());
  let mut handles = vec![];

  for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = std::thread::spawn(move || {
      for _ in 0..100 {
        counter.increment();
      } 
    });
    handles.push(handle); 
  }

  for handle in handles {
  handle.join().unwrap();
  }

  println!("Counter value: {}", counter.get_count()); 
}
```

**【反例】**

- 多线程通过 unsafe 修改静态变量

```Rust
// 不符合：数据竞争
static mut SHARED_STATE: u8 = 0;

thread::spawn(move || unsafe { SHARED_STATE = 1 };); 
thread::spawn(move || unsafe { SHARED_STATE = 2 };);
```

**【正例】**

- 多线程通过 unsafe 修改静态变量

```Rust
// 符合
static mut SHARED_STATE: u8 = AtomicU8::new(0);

thread::spawn(move || unsafe { SHARED_STATE.store(1, Ordering::SeqCst); }); 
thread::spawn(move || unsafe { SHARED_STATE.store(2, Ordering::SeqCst); });
```


#### G.SAF.ASY.01 避免在异步处理过程中包含阻塞操作

**【级别】** 建议

**【描述】**

避免在异步编程中使用阻塞操作。

**【反例】**

不要在异步流程中使用阻塞操作函数

```Rust
async fn read_file() {
  std::thread::sleep(std::time::Duration::from_secs(5)); // 不符合 
}
```

**【正例】**

使用异步运行时，如 tokio 提供的非阻塞函数

```Rust
async fn read_file() {
  tokio::time::sleep(std::time::Duration::from_secs(5)).await; // 符合 
  Ok(())
}
```


#### G.SAF.MTH.01 多线程场景下，对于共享的标量类型数据，宜使用对应的原子类型

**【级别】** 建议

**【描述】**

在多线程场景下，对于需要共享的类型数据，如布尔、整型和指针类型，建议使用原子类型，而非使用 互斥锁进行封装。使用原子类型性能更好。

**【反例】**

```Rust
// 不符合
let x = Mutex::new(false);
```

**【正例】**

```Rust
// 符合
let x = AtomicBool::new(false);
```


## 4.4 内存安全


#### G.SAF.MEM.01 应校验内存申请函数的输入参数和返回值

**【级别】** 要求

**【描述】**

涉及内存分配的函数调用，应注意：

- 申请前应校验申请大小

当申请内存大小由程序外部输入时，内存申请前，要求对申请内存大小进行合法性校验，防止申请 0 长度内存，或过多地、非法地申请内存。

- 申请后应校验是否成功

当申请内存的数值过大（可能一次就申请了非常大的超预期的内存；也可能循环中多次申请内 存），可能会造成内存申请失败，返回空指针。

**【反例】**

申请前应校验申请大小

```Rust
// CWE-789
fn get_untrusted_size() -> usize {
  //返回外部函数确定的大小
  unsafe { other_lib_get_size() } 
}

fn main() {
  let size = get_untrusted_size();
  // 不符合：内存分配 size 由外部传入，没有检查其值是否过大
  let buffer: Vec<u8> = Vec::with_capacity(size);
  // receive external data with buffer...
}
```

**【正例】**

申请前应校验申请大小

```Rust
fn get_untrusted_size() -> usize {
  //返回外部函数确定的大小
  unsafe { other_lib_get_size() } 
}

fn main() {
  const BUF_MAX_SIZE: usize = 2_usize.pow(8);
  let size = BUF_MAX_SIZE.min(get_untrusted_size());
  // 符合：给外部传入的内存分配 size 值设定了一个上限
  let buffer: Vec<u8> = Vec::with_capacity(size);
  // receive external data with buffer...
}
```

**【反例】**

申请后应校验是否成功

```Rust
fn func() {
  let malloc_size = || usize::MAX;
  // 不符合：未校验内存分配函数的参数
  let ptr = unsafe { libc::malloc(malloc_size()) };
  // 使用 ptr 进行内存操作 
  // ...
  unsafe { libc::free(ptr) }; 
}
```

**【正例】**

申请后应校验是否成功

```Rust
fn func() {
  let malloc_size = || usize::MAX;
  let ptr = unsafe { libc::malloc(malloc_size()) }; 
  if !ptr.is_null() {
    // 使用 ptr 进行内存操作 
    // ...
    unsafe { libc::free(ptr) }; 
  }
}
```


#### G.SAF.MEM.02 Unsafe 模式下，使用指针及内存地址时应确保有效性

**【级别】** 要求

**【描述】**

在 Unsafe Rust 中，通过裸指针方式申请分配内存应注意：

- 指针解引用前应确保指向的内存已初始化

访问未初始化内存会导致未定义行为。

- FFI函数调用返回的指针应先检查是否为空再使用

解引用空指针会导致程序崩溃。

- 不应将局部变量的栈地址返回到作用域之外

栈内存会在函数返回后被回收，访问会导致未定义行为。

- 指针指向的内存释放后禁止继续访问

指针指向的内存释放后，继续通过指针访问会导致未定义行为。

**【反例】**

禁止使用未初始化的指针

```Rust
fn main() {
  let foo_ptr: *mut Foo = std::ptr::null_mut();
  // 不符合：禁止使用未初始化的指针
  unsafe { println!("{:?}", (*foo_ptr).ptr); } 
}

// 模拟一个外部 c-struct
struct Foo<'a> {
   ptr: &'a i32, 
}
```

**【正例】**

禁止使用未初始化的指针

```Rust
use std::mem::MaybeUninit;

fn main() {
  // 符合：建议的外部数据结构初始化方式
  let foo: Foo = unsafe {
    let mut foo = MaybeUninit::uninit(); 
    initialize(foo.as_mut_ptr());
    foo.assume_init() 
  };
  println!("{:?}", foo.ptr); 
}

// 模拟一个外部 c-struct
struct Foo<'a> { ptr: &'a i32, }

// 模拟一个外部 c 函数
fn initialize(p: *mut Foo) {  
  unsafe { (*p).ptr = &0 };
}
```

**【反例】**

禁止将局部变量的地址返回到作用域之外

```Rust
fn main() {
  let p = return_stack_address();
  unsafe { println!("{}", *p); } 
}

fn return_stack_address() -> *const i32 {
  let value = 123_i32;
  // 不符合：禁止将局部变量的地址返回到作用域之外
  &value as *const i32 
}
```

**【反例】** 禁止释放指针后继续使用

```Rust
const BUF_SIZE: usize = 100;

unsafe {
  let buf = malloc(BUF_SIZE) as *mut libc::c_char;

  // 操作指针 , 写入数据

  free(buf as *mut libc::c_void);

  // 不符合：free 内存后数据还在， `buf` 变成悬挂指针且不为空。 
  // 因此仍然会解引用指针
  if !buf.is_null() {
    println!("{:?}", *buf);
  } 
}
```

**【正例】**

```Rust
const BUF_SIZE: usize = 100;

unsafe {
  let mut buf = malloc(BUF_SIZE) as *mut libc::c_char;

  // 操作指针 , 写入数据

  // 符合：释放内存后将指针设置为 null
  free(buf as *mut c_void);
  buf = std::ptr::null_mut();

  // 符合： `ptr` 为空，判断式不成立所以不会解引用。
  if !buf.is_null() {
    println!("{:?}", *buf);
  } 
}
```


#### G.SAF.MEM.03 Unsafe 模式下，应正确释放申请的内存

**【级别】** 要求

**【描述】**

当动态分配内存不再使用时应予以释放，否则会发生内存泄漏。如果攻击者可以有意触发该漏洞，则内 存资源可能被耗尽，造成拒绝服务。

优先推荐使用 RAII 模式进行内存释放，避免内存泄漏和重复释放的风险。

内存释放后，对应的指针变成悬空指针，建议重置为空指针, 一旦错误访问，可以立即得到反馈，避免 悬空指针的未定义行为（内存可能被其他业务模块重新分配，悬空指针的使用可能无意间修改了其他业务模块的数据）。

**【反例】**

- 双重释放

```Rust
const BUF_SIZE: usize = 100;

unsafe {
  let buf = malloc(BUF_SIZE) as *mut libc::c_char;

  // 操作指针 , 写入数据

  // 不符合：释放内存，但是指针没有设置为空
  free(buf as *mut libc::c_void);

  // ......

  // 不符合：free 内存后，指针不是空，会导致双重释放错误
  free(buf as *mut libc::c_void); 
}
```

【正例】

- 双重释放

```Rust
const BUF_SIZE: usize = 100;

unsafe {
  let mut buf = malloc(BUF_SIZE) as *mut libc::c_char;

  // 操作指针 , 写入数据

  // 符合：释放内存后将指针设置为 null
  if !buf.is_null() {
    free(buf as *mut c_void);
    buf = std::ptr::null_mut();
  }

  // ......

  // 符合： `ptr` 为空，不会造成双重释放错误
  if !buf.is_null() {
    free(buf as *mut c_void);
    buf = std::ptr::null_mut();
  } 
}
```

【反例】

- 忘记释放

```Rust
const BUF_SIZE: usize = 100;

unsafe {
  let buf = malloc(BUF_SIZE) as *mut libc::c_char;

  // 操作指针 , 写入数据

  // 不符合：忘记释放内存
}
```


#### G.SAF.MEM.04 内存中的敏感信息使用完毕后应立即清零

**【级别】** 要求

**【描述】**

内存中的口令、密钥等敏感信息使用完毕后应立即清零，避免被攻击者获取。 为防止内存清零操作被编译优化，推荐使用 `std::ptr::write_volatile` 函数。

**【正例】**

```Rust
struct Buffer {
  password: [u8; 16],
}

impl Buffer {
  fn new() -> Self {
    Self { password: [1; 16] } // 初始化缓冲区
  } 
}

impl Drop for Buffer {
  fn drop(&mut self) { 
    unsafe {
      std::ptr::write_volatile(
        &mut self.password.as_slice(),
        vec![0; self.password.len()].as_slice(), 
      );
    }
  } 
}

fn main() {
  let _buffer = Buffer::new(); // 创建缓冲区实例

  // 当 buffer 离开作用域时，会自动调用其 drop 函数清空内存
}
```


# 5 附录


## 附录A Rustfmt 配置与模板

公司内开发者可参考以下 Rustfmt 模板。 模板内容可以放到 `rustfmt.toml` 或 `.rustfmt.toml` 文件中。使用 `cargo fmt` 执行。 很多选项可直接使用默认值，因此无需配置。以下配置针对使用非默认值的情况。

**【只包含 Stable 的选项】**

```Rust
# 万一你要使用 rustfmt 2.0 就需要指定这个 ·
version = "Two"

# 统一管理宽度设置，但不包含 comment_width
use_small_heuristics="MAX"
# 在match分支中，如果包含了块，也需要加逗号以示分隔
match_block_trailing_comma=true
# 如果项目只在 Unix 平台下跑，可以设置该项为 Unix，表示换行符只依赖 Unix
newline_style="Unix"
# 不要将多个 Derive 宏合并为同一行
merge_derives = false
# 设置行长度
max_width = 120

# 指定 fmt 忽略的目录
ignore = [
  "src/test", 
  "test",
  "docs",
]
```


## 附录B Clippy 配置与模板

有些 Clippy 的 Lint，依赖于一些配置项，如果不想要默认值，可以在 `clippy.toml` 中进行设置。

```
# for `disallowed_method`:
# https://rust-lang.github.io/rust-clippy/master/index.html#disallowed_method
disallowed-methods = []

# 函数参数最长不要超过5个
too-many-arguments-threshold=5
```

**Clippy lint 配置模板**

```
// 参考： https://github.com/serde-rs/serde/blob/master/serde/src/lib.rs
#![allow(unknown_lints, bare_trait_objects, deprecated)]
#![cfg_attr(feature = "cargo-clippy", allow(renamed_and_removed_lints))] #![cfg_attr(feature = "cargo-clippy", deny(clippy, clippy_pedantic))]
// Ignored clippy and clippy_pedantic lints
#![cfg_attr(
  feature = "cargo-clippy", allow(
    // clippy bug: https://github.com/rust-lang/rust-clippy/issues/5704
    unnested_or_patterns,
    // clippy bug: https://github.com/rust-lang/rust-clippy/issues/7768
    semicolon_if_nothing_returned,
    // not available in our oldest supported compiler
    checked_conversions, empty_enum,
    redundant_field_names,
    redundant_static_lifetimes,
    // integer and float ser/de requires these sorts of casts
    cast_possible_truncation, cast_possible_wrap,
    cast_sign_loss,
    // things are often more readable this way
    cast_lossless,
    module_name_repetitions,
    option_if_let_else, single_match_else,  type_complexity,
    use_self,
    zero_prefixed_literal,
    // correctly used
    enum_glob_use,
    let_underscore_drop, map_err_ignore,
    result_unit_err,  wildcard_imports,
    // not practical
    needless_pass_by_value, similar_names,
    too_many_lines,
    // preference
    doc_markdown,
    unseparated_literal_suffix,
    // false positive
    needless_doctest_main,
    // noisy
    missing_errors_doc, must_use_candidate,
  ) 
)]
// Rustc lints.
#![deny(missing_docs, unused_imports)]
```

**Embark Studios 的标准 Lint 配置**

```
// BEGIN - Embark standard lints v5 for Rust 1.55+
// do not change or add/remove here, but one can add exceptions after this section
// for more info see: <https://github.com/EmbarkStudios/rust- ecosystem/issues/59>
#![deny(unsafe_code)] #![warn(
  clippy::all,
  clippy::await_holding_lock, clippy::char_lit_as_u8,
  clippy::checked_conversions, clippy::dbg_macro,
  clippy::debug_assert_with_mut_call, clippy::disallowed_method,
  clippy::disallowed_type, clippy::doc_markdown,
  clippy::empty_enum,
  clippy::enum_glob_use, clippy::exit,
  clippy::expl_impl_clone_on_copy, clippy::explicit_deref_methods,  clippy::explicit_into_iter_loop, clippy::fallible_impl_from,
  clippy::filter_map_next,
  clippy::flat_map_option, clippy::float_cmp_const,
  clippy::fn_params_excessive_bools,
  clippy::from_iter_instead_of_collect,
  clippy::if_let_mutex,
  clippy::implicit_clone,  clippy::imprecise_flops,
  clippy::inefficient_to_string,
  clippy::invalid_upcast_comparisons, clippy::large_digit_groups,
  clippy::large_stack_arrays,
  clippy::large_types_passed_by_value, clippy::let_unit_value,
  clippy::linkedlist,
  clippy::lossy_float_literal, clippy::macro_use_imports,
  clippy::manual_ok_or,
  clippy::map_err_ignore, clippy::map_flatten,
  clippy::map_unwrap_or,
  clippy::match_on_vec_items, clippy::match_same_arms,
  clippy::match_wild_err_arm,
  clippy::match_wildcard_for_single_variants, clippy::mem_forget,
  clippy::mismatched_target_os,
  clippy::missing_enforced_import_renames, clippy::mut_mut,
  clippy::mutex_integer,
  clippy::needless_borrow,
  clippy::needless_continue, clippy::needless_for_each, clippy::option_option,
  clippy::path_buf_push_overwrite, clippy::ptr_as_ptr,
  clippy::rc_mutex,
  clippy::ref_option_ref,
  clippy::rest_pat_in_fully_bound_structs, clippy::same_functions_in_if_condition,  clippy::semicolon_if_nothing_returned,
  clippy::single_match_else,
  clippy::string_add_assign, clippy::string_add,
  clippy::string_lit_as_bytes, clippy::string_to_string,
  clippy::todo,
  clippy::trait_duplication_in_bounds, clippy::unimplemented,
  clippy::unnested_or_patterns, clippy::unused_self,
  clippy::useless_transmute,
  clippy::verbose_file_reads,
  clippy::zero_sized_map_values, future_incompatible,
  nonstandard_style, rust_2018_idioms
)]
// END - Embark standard lints v0.5 for Rust 1.55+ // crate-specific exceptions:
#![allow()]
```

**Clippy 配置的相关问题**

目前 Clippy 不支持配置文件来配置Lint ，目前 像 Embark 公司有两种解决方法：

- 将 lint 放到一个[统一文件](https://github.com/EmbarkStudios/rust-ecosystem/blob/main/lints.rs)中，然后复制粘贴到使用的地方。
- 通过 `.cargo/config.toml` 来配置 `rustflags` ，参考： [lints.toml](https://github.com/EmbarkStudios/rust-ecosystem/blob/main/lints.toml)

Embark 也在跟踪和推动在 Cargo 中支持 Lint 配置的功能，相关 issues：

- [Be able to disable/enable Clippy lints globally](https://github.com/EmbarkStudios/rust-ecosystem/issues/22)
- [Support defining enabled and disabled lints in a configuration file](https://github.com/rust-lang/cargo/issues/5034)
- [[Roadmap] Configuration file for lints](https://github.com/rust-lang/rust-clippy/issues/6625)


## 参考资料

1. [官方｜Rust API 编写指南](https://rust-lang.github.io/api-guidelines/about.html)
2. [官方 | Rust Style Guide](https://github.com/rust-dev-tools/fmt-rfcs/blob/master/guide/guide.md)
3. [Rust's Unsafe Code Guidelines Reference](https://rust-lang.github.io/unsafe-code-guidelines/)
4. [法国国家信息安全局 | Rust 安全（Security）规范](https://anssi-fr.github.io/rust-guide)
5. [Facebook Diem 项目 Rust 编码规范](https://developers.diem.com/docs/core/coding-guidelines/)
6. [Apache Teaclave 安全计算平台 | Rust 开发规范](https://teaclave.apache.org/docs/rust-guildeline/)
7. [PingCAP | 编码风格指南（包括 Rust 和 Go 等）](https://github.com/pingcap/style-guide)
8. [Google Fuchsia 操作系统 Rust 开发指南](https://fuchsia.dev/fuchsia-src/development/languages/rust)
9. [RustAnalyzer 编码风格指南](https://github.com/rust-analyzer/rust-analyzer/blob/master/docs/dev/style.md)
10. [使用 Rust 设计优雅的 API](https://deterministic.space/elegant-apis-in-rust.html)
11. [Rust FFI 指南](https://michael-f-bryan.github.io/rust-ffi-guide/)
12. [大约 478 条 Clippy lint](https://rust-lang.github.io/rust-clippy/master/index.html)
13. [lints in the rustc book](https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html)
14. [Dtolnay 对 crates.io 中 clippy lint 应用统计](https://github.com/dtolnay/noisy-clippy)
