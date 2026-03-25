# Rust Coding Guidelines 实现情况统一分析

> 本报告整合了 Coding Guidelines 全量条目（含 codecheck 扩展），分析每条规范的实现来源：
> - **Clippy 原生**：由上游 rust-clippy 项目原生提供的 lint
> - **Fork 自定义**：由 `J-ZhengLi/rust-clippy` fork 分支自定义开发的 lint
> - **未实现**：目前无工具支持

## 1. 总体统计

| 指标 | 数值 |
|------|------|
| Coding Guidelines 标准条目 | 53 (11 原则 + 42 规则) |
| Codecheck 扩展条目 | 4 |
| **全量条目合计** | **57** |
| Clippy 原生 lint 覆盖 | 21 条 |
| Fork 自定义 lint 覆盖 | 16 条 |
| 未实现（规则类） | 9 条 |
| 未实现（原则类，工具不检测） | 11 条 |

## 2. 全量条目实现情况表

> **实现来源说明**：
> - 🟢 Clippy 原生 = 上游 clippy 已有的 lint
> - 🔵 Fork 自定义 = fork 分支新增的 guidelines lint
> - 🔴 未实现 = 无工具检测支持
> - ⚪ 原则类 = 工具不检测，属指导性条目

### 2.1 规则类条目 (G.*)

| 编号 | 标题 | 实现来源 | Codecheck 检查代码 | Clippy/Fork Lint |
|------|------|---------|-------------------|-----------------|
| G.NAM.01 | 应使用统一的命名风格 | 🟢 Clippy 原生 | `non_camel_case_tyes`, `non_snake_case`, `non_upper_case_globals` | rustc 内置 lint |
| G.NAM.02 | 类型转换函数命名宜遵循所有权语义 | 🟢 Clippy 原生 | `wrong_self_convention` | `clippy::wrong_self_convention` |
| G.CMT.01 | 公开的 API 需要文档注释 | 🟢 Clippy 原生 | `missing_errors_doc`, `missing_panics_doc`, `missing_safety_doc`, `undocumented_unsafe_blocks` | `clippy::missing_errors_doc` 等 |
| G.CMT.02 | 文件头注释应包含版权说明 | 🔴 未实现 | — | — |
| G.FMT.01 | extern 外部函数应显式指定 ABI 标识 | 🔴 未实现 | — | — |
| G.CNS.01 | 常量类型不应具有内部可变性 | 🟢 Clippy 原生 | `borrow_interior_mutable_const`, `clippy::declare_interior_mutable_const` | `clippy::borrow_interior_mutable_const` 等 |
| G.CNS.02 | 宜使用标准库中的预定义常量 | 🟢 Clippy 原生 | `approx_constant` | `clippy::approx_constant` |
| G.TYP.01 | 数值字面量要添加明确的类型标识 | 🔵 Fork 自定义 | `unconstrained_numeric_literal` | `clippy::unconstrained_numeric_literal` |
| G.TYP.BOL.01 | 不宜使用块表达式作为控制表达式 | 🟢 Clippy 原生 | `clippy::blocks_in_conditions` | `clippy::blocks_in_conditions` |
| G.TYP.BOL.02 | 不宜将布尔变量和布尔字面量进行比较 | 🟢 Clippy 原生 | `bool_assert_comparison`, `bool_comparison` | `clippy::bool_assert_comparison` 等 |
| G.TYP.BOL.03 | && 和 \|\| 操作符的右侧操作中不应包含副作用 | 🔴 未实现 | — | — |
| G.TYP.CHR.01 | 整数到字符的类型转换应注意有效范围 | 🔵 Fork 自定义 | `invalid_char_range` | `clippy::invalid_char_range` |
| G.TYP.INT.01 | 整数类型的算数运算 | 🟢 Clippy 原生 | `arithmetic_side_effects` | `clippy::arithmetic_side_effects` |
| G.TYP.SLC.01 | 宜使用迭代器而非下标遍历切片和数组 | 🟢 Clippy 原生 | `needless_range_loop` | `clippy::needless_range_loop` |
| G.TYP.SLC.02 | 应保证数组索引在有效范围内 | 🟢 Clippy 原生 | `unconditional_panic` | rustc 内置 lint |
| G.TYP.SCT.01 | 外部使用的自定义类型宜实现常见 trait | 🔴 未实现 | — | — |
| G.TYP.STR.01 | 在实现 Display trait 时不应调用 to_string() 方法 | 🟢 Clippy 原生 | `clippy::recursive_format_impl` | `clippy::recursive_format_impl` |
| G.TYP.PTR.01 | Rc Arc Box 不应相互嵌套使用 | 🟢 Clippy 原生 | `redundant_allocation` | `clippy::redundant_allocation` |
| G.CTF.01 | 宜优先使用模式匹配而非判断后再取值 | 🔴 未实现 | — | — |
| G.CTF.02 | 在 Match 分支的 Guard 语句中不要使用带有副作用的条件表达式 | 🔴 未实现 | — | — |
| G.CTF.03 | 循环或递归必须安全退出 | 🔵 Fork 自定义 | `infinite_loop` | `clippy::infinite_loop` |
| G.EXP.01 | 避免复合条件表达式永远为 true 或 false | 🟢 Clippy 原生 | `bad_bit_mask` | `clippy::bad_bit_mask` |
| G.EXP.02 | 用括号明确表达式的操作顺序 | 🟢 Clippy 原生 | `precedence` | `clippy::precedence` |
| G.FUD.01 | 内存拷贝代价大的参数应按地址传递 | 🟢 Clippy 原生 | `large_types_passed_by_value` | `clippy::large_types_passed_by_value` |
| G.FUD.02 | 可能出错的函数，应保证用户不可忽略返回值 | 🟢 Clippy 原生 | `unused_must_use` | rustc 内置 lint |
| G.TRA.01 | 在实现相等性、排序和散列类 trait 时，应保证一致性 | 🟢 Clippy 原生 | `derive_ord_xor_partial_ord` | `clippy::derive_ord_xor_partial_ord` |
| G.TRA.02 | 宜实现 From 而不是 Into | 🟢 Clippy 原生 | `from_over_into` | `clippy::from_over_into` |
| G.ERR.01 | 应正确处理返回类型 Option 和 Result | 🟢 Clippy 原生 | `unwrap_used` | `clippy::unwrap_used` |
| G.MOD.01 | 导入模块中的类型或函数时，应避免直接使用通配符 | 🟢 Clippy 原生 | `wildcard_imports` | `clippy::wildcard_imports` |
| G.MAC.DCL.01 | 编写宏匹配规则时，宜遵从匹配范围从小至大的顺序 | 🔴 未实现 | — | — |
| G.MAC.PRO.011 | 不应通过过程宏将 unsafe 代码包装为 safe 代码 | 🔴 未实现 | — | — |
| G.SAF.UNS.01 | 禁止将外部可控数据作为加载动态库的参数 | 🔵 Fork 自定义 | `untrusted_lib_loading` | `clippy::untrusted_lib_loading` |
| G.SAF.FFI.01 | FFI 边界应确保字符串参数正确传递和使用 | 🔵 Fork 自定义 | `passing_string_to_c_functions` | `clippy::passing_string_to_c_functions` |
| G.SAF.FFI.02 | 在 FFI 调用的场景下，应确保跨边界数据的内存布局兼容 | 🔵 Fork 自定义 | `extern_without_repr` | `clippy::extern_without_repr` |
| G.SAF.FFI.03 | 禁止调用 C 不可重入函数 | 🔵 Fork 自定义 | `non_reentrant_functions` | `clippy::non_reentrant_functions` |
| G.SAF.FFI.04 | 禁止使用外部内存不安全函数 | 🔵 Fork 自定义 | `mem_unsafe_functions` | `clippy::mem_unsafe_functions` |
| G.SAF.ASY.01 | 避免在异步处理过程中包含阻塞操作 | 🔵 Fork 自定义 | `blocking_op_in_aysnc` | `clippy::blocking_op_in_async` |
| G.SAF.MTH.01 | 多线程场景下，对于共享的标量类型数据，宜使用对应的原子类型 | 🟢 Clippy 原生 | `mutex_atomic`, `mutex_integer` | `clippy::mutex_atomic` 等 |
| G.SAF.MEM.01 | 应校验内存申请函数的输入参数和返回值 | 🔵 Fork 自定义 | `fallible_memory_allocation` | `clippy::fallible_memory_allocation` |
| G.SAF.MEM.02 | Unsafe 模式下，使用指针及内存地址时应确保有效性 | 🔵 Fork 自定义 | `dangling_ptr_dereference` | `clippy::dangling_ptr_dereference`, `clippy::null_ptr_dereference`, `clippy::return_stack_address` |
| G.SAF.MEM.03 | Unsafe 模式下，应正确释放申请的内存 | 🔵 Fork 自定义 | `ptr_double_free` | `clippy::ptr_double_free` |
| G.SAF.MEM.04 | 内存中的敏感信息使用完毕后应立即清零 | 🔴 未实现 | — | — |

### 2.2 Codecheck 扩展规则（不在标准 Coding Guidelines 中）

| 编号 | 标题 | 实现来源 | Codecheck 检查代码 | Clippy/Fork Lint |
|------|------|---------|-------------------|-----------------|
| G.MAC.01 | 不应该通过宏将 unsafe 代码伪装为 safe 代码 | 🔵 Fork 自定义 | `unsafe_block_in_proc_macro` | `clippy::unsafe_block_in_proc_macro` |
| G.OTH.01 | 禁止硬编码 IP 地址 | 🟢 Clippy 原生 | `hard_coded_ip` | — |
| G.SAF.FFI.05 | extern 块应显式标注 ABI | 🔵 Fork 自定义 | `implic_abi` | `clippy::implicit_abi` |
| G.SAF.UNS.02 | 禁止从不可变引用创建可变引用 | 🟢 Clippy 原生 | `mut_from_ref` | `clippy::mut_from_ref` |

### 2.3 原则类条目 (P.*) — 工具不检测

| 编号 | 标题 | 状态 |
|------|------|------|
| P.NAM.01 | 标识符命名应符合阅读习惯 | ⚪ 原则类，工具不检测 |
| P.CMT.01 | 注释跟代码一样重要，应按需注释 | ⚪ 原则类，工具不检测 |
| P.FMT.01 | 代码格式应保持统一 | ⚪ 原则类，工具不检测 |
| P.TYP.01 | 合理选择算术转换方式 | ⚪ 原则类，工具不检测 |
| P.MAC.DCL.01 | 定义和使用声明宏时，应保证卫生性 | ⚪ 原则类，工具不检测 |
| P.MAC.PRO.01 | 定义和使用过程宏时，应保证卫生性 | ⚪ 原则类，工具不检测 |
| P.SAF.UNS.01 | Unsafe 代码应仅在需要的场景下使用 | ⚪ 原则类，工具不检测 |
| P.SAF.FFI.01 | 在 FFI 边界上传递数据应注意内存安全管理 | ⚪ 原则类，工具不检测 |
| P.SAF.FFI.02 | 应正确处理 FFI 调用的返回值和输出参数 | ⚪ 原则类，工具不检测 |
| P.SAF.FFI.03 | 应优先选用 RAII 方式管理外部资源 | ⚪ 原则类，工具不检测 |
| P.SAF.MTH.01 | 并发场景下应避免数据竞争和死锁 | ⚪ 原则类，工具不检测 |

## 3. 实现来源分布汇总

### 3.1 规则类 (G.*) 实现来源分布

| 实现来源 | 条目数 | 占规则类比例 |
|---------|--------|------------|
| 🟢 Clippy 原生 | 21 | 45.7% |
| 🔵 Fork 自定义 | 16 | 34.8% |
| 🔴 未实现 | 9 | 19.6% |
| **规则类合计** | **46** | **100%** |

> 注：规则类 46 条 = 标准 42 条 + codecheck 扩展 4 条

### 3.2 全量条目实现来源分布

| 实现来源 | 条目数 | 占全量比例 |
|---------|--------|----------|
| 🟢 Clippy 原生 | 21 | 36.8% |
| 🔵 Fork 自定义 | 16 | 28.1% |
| 🔴 未实现（规则类） | 9 | 15.8% |
| ⚪ 原则类（不检测） | 11 | 19.3% |
| **全量合计** | **57** | **100%** |

### 3.3 待实现的 9 条规则

| 编号 | 标题 | 难度评估 |
|------|------|---------|
| G.CMT.02 | 文件头注释应包含版权说明 | 中等（需解析文件头注释格式） |
| G.FMT.01 | extern 外部函数应显式指定 ABI 标识 | 低（与 G.SAF.FFI.05 `implicit_abi` 功能相近） |
| G.CTF.01 | 宜优先使用模式匹配而非判断后再取值 | 高（需识别 if-let 可替代的模式） |
| G.CTF.02 | Match Guard 中不应使用带副作用的条件表达式 | 高（需分析副作用） |
| G.TYP.BOL.03 | && 和 \|\| 右侧不应包含副作用 | 高（需分析副作用） |
| G.TYP.SCT.01 | 外部使用的自定义类型宜实现常见 trait | 中等（需判断"外部使用"和"常见 trait"） |
| G.MAC.DCL.01 | 编写宏匹配规则时，宜遵从匹配范围从小至大的顺序 | 高（需分析宏匹配范围） |
| G.MAC.PRO.011 | 不应通过过程宏将 unsafe 代码包装为 safe 代码 | 中等（与 G.MAC.01 相近，但针对标准 Guidelines 编号） |
| G.SAF.MEM.04 | 内存中的敏感信息使用完毕后应立即清零 | 高（需识别"敏感信息"语义） |
