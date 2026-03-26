# G.CTF.01 宜优先使用模式匹配而非判断后再取值

## 级别
建议

## 规范描述
应优先使用模式匹配方式，而不是通过 if 判断值是否相等，这样代码会更简洁。

## 检查要点
- 是否存在 `is_some()` 后跟 `unwrap()` 的模式（应使用 `if let Some(x) = ...` 替代）
- 是否存在 `is_none()` 后跟 `unwrap()` 或 `expect()` 的模式
- 是否存在 `is_ok()` 后跟 `unwrap()` 的模式（应使用 `if let Ok(x) = ...` 替代）
- 是否存在 `is_err()` 后跟 `unwrap_err()` 的模式
- 是否存在 `!list.is_empty()` 后跟 `list[0]` 索引取值的模式（应使用 `if let [first, ..] = list` 替代）
- 是否存在先判断 `Option`/`Result` 状态再取值的两步操作模式

## 反例

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

## 检查指令
扫描项目中所有 `.rs` 文件，执行以下检查：

1. 搜索 `is_some()` 调用，检查其后续代码块中是否对同一变量调用了 `unwrap()`。若存在此模式，建议使用 `if let Some(value) = variable` 替代。
2. 搜索 `is_none()` 调用，检查是否在 else 分支中对同一变量调用了 `unwrap()`。
3. 搜索 `is_ok()` 调用，检查其后续代码块中是否对同一变量调用了 `unwrap()`。若存在此模式，建议使用 `if let Ok(value) = variable` 替代。
4. 搜索 `is_err()` 调用，检查其后续代码块中是否对同一变量调用了 `unwrap_err()`。
5. 搜索 `!xxx.is_empty()` 判断，检查其后续代码块中是否通过索引（如 `xxx[0]`）取值。若存在此模式，建议使用切片模式匹配（如 `if let [first, ..] = xxx`）替代。
6. 对于每个匹配到的模式，标记为"可使用模式匹配替代判断后取值"，并给出建议的替代写法。
7. 汇总所有不符合项，按文件路径和行号列出问题。
