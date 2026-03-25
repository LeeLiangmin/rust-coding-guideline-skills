# G.MAC.DCL.01 编写宏匹配规则时，宜遵从匹配范围从小至大的顺序

## 级别
建议

## 规范描述
因为声明宏中，是按规则的编写顺序来匹配的。当第一个规则被匹配到，后面的规则将永远不会匹配到。

所以，编写声明宏规则时，需要先写匹配范围最小的，最具体的规则，然后逐步编写匹配范围更广泛的规则。

## 说明
参考 [The Little Book of Rust Macros](https://danielkeep.github.io/tlborm/book/mbe-min-captures-and-expansion-redux.html)。

片段说明符（fragment specifier）的匹配范围从小到大大致为：
- `lit`（字面量）< `ident`（标识符）< `expr`（表达式）< `tt`（token tree）< `item` < `$($other:tt)*`（任意 token 序列）

## 检查要点
- `macro_rules!` 中多个匹配分支的片段说明符是否从小到大排列
- 是否存在匹配范围大的规则排在匹配范围小的规则前面，导致后续规则永远无法匹配
- 使用 `$($other:tt)*` 等通配匹配是否放在了最后一个分支
- 具体的字面量模式（如 `($a:tt + $b:tt)`）是否排在通用模式（如 `($i:ident)`）之前
- 是否存在因规则顺序不当导致的"死规则"（永远不会被匹配到的规则）

## 反例

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

## 正例

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

## 检查指令
扫描项目中所有 `.rs` 文件，执行以下检查：

1. 定位所有 `macro_rules!` 定义。
2. 对于每个宏定义，提取其所有匹配分支的模式部分。
3. 分析每个分支模式中使用的片段说明符（fragment specifier），评估其匹配范围大小：
   - 具体字面量模式（如固定 token 序列）范围最小。
   - `$x:lit` < `$x:ident` < `$x:path` < `$x:expr` < `$x:ty` < `$x:tt`。
   - `$($x:tt)*` 或 `$($x:tt)+` 等重复模式范围最大。
4. 检查分支顺序是否从匹配范围小到大排列：
   - 若发现范围大的分支排在范围小的分支前面，标记为"宏匹配规则顺序不当，可能导致后续规则无法匹配"。
5. 特别检查 `$($x:tt)*` 是否作为最后一个分支（兜底规则）。
6. 汇总所有不符合项，按文件路径和行号列出问题，并说明建议的规则排列顺序。
