## 想法
目前有 53 条规范，我想有一个静态工具 + 大模型辅助检测的工作流程，并且固化为 skills 等，用于对 rust 项目进行完整的规范检查。具体说明如下：

1. 静态工具校验大部分规则：coding-guidelines-ruleset。
2. 无法校验部分使用大模型来完成检查
3. coding-guidelines-ruleset 实际是一个特定改造过的 clippy, 注册为 guidelines_runner，我也有 mcp 
来处理，https://www.npmjs.com/package/rust-guidelines-mcp-server，本地安装：
```json
{
  "mcpServers": {
    "rust-guidelines": {
      "command": "npx",
      "args": ["-y", "rust-guidelines-mcp-server"]
    }
  }
}
```

4. 具体的材料可见当前目录下的 data 目录。
5. 静态检测以及大模型完成项目的检测输出格式，按照如下 schema 完成静态后动态工具的对齐。


### schema 
```json
{
  "projectPath": "f:\\lee_space\\code\\rust-clippy-mcp-server-test",
  "summary": {
    "total": 30,
    "errors": 2,
    "warnings": 28
  },
  "diagnostics": [
    {
      "level": "warning",
      "code": "clippy::collapsible_if",
      "message": "this `if` statement can be collapsed",
      "file": "src\\main.rs",
      "line": 74,
      "column": 5,
      "rendered": "warning: this `if` statement can be collapsed\n  --> src\\main.rs:74:5\n   |\n74 | /     if a {\n75 | |         if b {\n76 | |             println!(\"both true\");\n77 | |         }\n78 | |     }\n   | |_____^\n   |\n   = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#collapsible_if\n   = note: `#[warn(clippy::collapsible_if)]` on by default\nhelp: collapse nested if block\n   |\n74 ~     if a && b {\n75 +         println!(\"both true\");\n76 +     }\n   |\n\n",
      "suggestions": [
        {
          "file": "src\\main.rs",
          "line_start": 74,
          "line_end": 78,
          "column_start": 5,
          "column_end": 6,
          "replacement": "if a && b {\n        println!(\"both true\");\n    }",
          "applicability": "MachineApplicable"
        }
      ]
      }
    ]
}
```


## 结构
skills
+ default.skill(默认全部检测) 
	+ 执行默认的检测过程(对应全部的 tool_handle 表格)
	+ 查看llm_handle 对应的处理方式。补充额外的处理逻辑
	+ 输出对齐 schema
+ specific.skill(仅仅特定规则，比如用户想检查特定的规则)
	+ 查询 tool_handle
	+ 查询 llm_handle 
	+ 获取处理方式，分别处理
	+ 输出对齐 schema
+ assets
	+ tool_handle
	+ llm_handle 
	+ schema  
+ refs
	+ 特定 skill 


## 路由表设计
tool_handle 
| guideline code | title | clippy code | handler |
| G.NAM.01  | 应使用统一的命名风格 | a |  tool 处理 |
|G.NAM.01 | 应使用统一的命名风格 | b |  tool 处理 | 


llm_handle 
| guideline code | title | clippy code | handler |
| G.CMT.02  | 文件头注释应包含版权说明 | none |  link_to_spec_skill |




## tool 的默认 lint 项
```toml
[build]
rustflags = [
    "-W", "clippy::implicit_abi",
    "-W", "clippy::infinite_loop",
    "-W", "clippy::unsafe_block_in_proc_macro",
    "-W", "clippy::untrusted_lib_loading",
    "-W", "clippy::passing_string_to_c_functions",
    "-W", "clippy::extern_without_repr",
    "-W", "clippy::mem_unsafe_functions",
    "-W", "clippy::non_reentrant_functions",
    "-W", "clippy::blocking_op_in_async",
    "-W", "clippy::fallible_memory_allocation",
    "-W", "clippy::null_ptr_dereference",
    "-W", "clippy::dangling_ptr_dereference",
    "-W", "clippy::return_stack_address",
    "-W", "clippy::ptr_double_free",
    "-W", "clippy::invalid_char_range",
    "-W", "clippy::unconstrained_numeric_literal",
]
```
例如：设置 ruleset 工具。rustup toolchain link guidelines_runner F:\lee_space\rim_local_install\install\tools\ruleset\runner 注册


