# G.CTF.02 Match Guard 中不应使用带副作用的条件表达式

## 级别
建议

## 规范描述
在 match 分支中，匹配几次就会执行 Guard 几次。Guard 的条件表达式不应该携带副作用。

## 说明
副作用是指对执行状态产生影响，包括：修改对象、修改文件等。在某些情况下副作用会给程序带来不必要的麻烦，其产生的错误十分难以查找，并降低程序的可读性。

## 检查要点
- match 分支 guard（`if` 条件）中是否包含修改状态的操作（如 `Cell::set`、`RefCell::borrow_mut` 等）
- match 分支 guard 中是否包含 IO 操作（如 `println!`、`write!`、文件操作等）
- match 分支 guard 中是否包含函数调用且该函数可能有副作用
- 使用 `|` 连接多个模式时，guard 是否会被多次执行导致副作用被重复触发
- guard 中调用的函数是否为纯函数（无副作用）

## 反例

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

## 正例

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

## 检查指令
扫描项目中所有 `.rs` 文件，执行以下检查：

1. 定位所有 `match` 表达式。
2. 对于每个 match 分支，检查是否包含 guard 条件（即 `模式 if 条件 => ...` 形式）。
3. 对于包含 guard 的分支，分析 guard 条件表达式：
   - 检查是否包含函数调用（特别是可能有副作用的函数）。
   - 检查是否包含 `println!`、`eprintln!`、`write!` 等宏调用。
   - 检查是否包含 `Cell::set`、`RefCell::borrow_mut`、赋值操作等状态修改。
   - 检查是否包含文件操作、网络操作等 IO 副作用。
4. 特别关注使用 `|` 连接多个模式的分支（如 `A | B if guard`），因为 guard 会对每个匹配的模式执行一次，副作用会被放大。
5. 对于纯比较操作（如 `== `、`>`、`<`、`.is_empty()` 等），不标记为问题。
6. 汇总所有不符合项，按文件路径和行号列出问题，并说明可能的副作用类型。
