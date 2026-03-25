# G.SAF.MEM.04 内存中的敏感信息使用完毕后应立即清零

## 级别
要求

## 规范描述
内存中的口令、密钥等敏感信息使用完毕后应立即清零，避免被攻击者获取。为防止内存清零操作被编译优化，推荐使用 `std::ptr::write_volatile` 函数。

## 检查要点
- 密码、密钥等敏感数据变量（如命名包含 `password`、`secret`、`key`、`token`、`credential`、`passphrase` 等）是否在使用后进行了清零
- 敏感数据类型是否实现了 `Drop` trait 并在 `drop` 方法中执行清零操作
- 清零操作是否使用了 `std::ptr::write_volatile` 或类似的防优化清零方式（而非普通赋值，普通赋值可能被编译器优化掉）
- 是否使用了 `zeroize` 等专用安全清零库
- 敏感数据的 `Vec`、`String` 等堆分配容器是否在 drop 前清零了内部缓冲区
- 临时存储敏感信息的栈变量是否在作用域结束前清零

## 正例

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

## 检查指令
扫描项目中所有 `.rs` 文件，执行以下检查：

1. 搜索包含敏感信息语义的变量名和字段名，关键词包括：
   - `password`、`passwd`、`pwd`
   - `secret`、`secret_key`
   - `key`、`private_key`、`api_key`、`encryption_key`
   - `token`、`access_token`、`auth_token`
   - `credential`、`passphrase`
   - `pin`、`cvv`、`ssn`
2. 对于每个匹配到的敏感变量/字段，检查：
   - 如果是结构体字段，检查该结构体是否实现了 `Drop` trait。
   - 在 `Drop::drop` 实现中，是否对敏感字段执行了清零操作。
   - 清零操作是否使用了 `std::ptr::write_volatile`、`zeroize` crate 或其他防优化清零方式。
3. 对于局部变量中的敏感数据，检查在变量最后一次使用后、作用域结束前是否有清零操作。
4. 检查是否使用了普通赋值（如 `password = [0; 16]`）进行清零——这可能被编译器优化掉，应标记为"清零操作可能被编译器优化"。
5. 检查项目依赖中是否引入了 `zeroize` 等安全清零库，若已引入则检查敏感类型是否 derive 了 `Zeroize`/`ZeroizeOnDrop`。
6. 汇总所有不符合项，按文件路径和行号列出问题，并说明建议的修复方式。
