---
title: Rust 错误处理机制教程 8day
published: 2025-11-03
description: ''
image: ''
tags: [rust]
category: 'rust'
draft: false 
lang: ''
---

# 🦀 Rust 错误处理机制教程

## 目录

1. 错误分类
2. panic! 宏（不可恢复错误）
3. Option 类型
4. Result 类型
5. 错误传播（? 运算符）
6. 自定义错误类型
7. 错误链和上下文
8. 综合示例
9. 总结

---

## 1️⃣ 错误分类

Rust 将错误分为两类：

1. **不可恢复错误（Unrecoverable）**

   * 程序无法继续运行
   * 使用 `panic!` 触发
   * 例如数组越界、unwrap None

2. **可恢复错误（Recoverable）**

   * 可以通过程序逻辑处理
   * 使用 `Result<T, E>` 表示
   * 例如文件不存在、网络请求失败

---

## 2️⃣ panic! 宏（不可恢复错误）

```rust
fn main() {
    let v = vec![1, 2, 3];
    // println!("{}", v[10]); // ❌ panic: 索引越界
    panic!("程序遇到致命错误");
}
```

* 会立即中止程序
* 可在测试中用作断言

```rust
fn divide(a: i32, b: i32) -> i32 {
    if b == 0 { panic!("除数不能为0"); }
    a / b
}
```

---

## 3️⃣ Option 类型

用于 **值可能存在或不存在**：

```rust
let value: Option<i32> = Some(10);
let no_value: Option<i32> = None;

// 安全访问
if let Some(v) = value {
    println!("value = {}", v);
}

// 默认值
println!("no_value = {}", no_value.unwrap_or(0));
```

常用方法：

* `is_some() / is_none()`
* `unwrap()`（可能 panic）
* `unwrap_or(default)`
* `map() / and_then()`

---

## 4️⃣ Result 类型

用于 **可恢复错误**：

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 { Err("除数不能为0".to_string()) } else { Ok(a / b) }
}

fn main() {
    match divide(10, 2) {
        Ok(v) => println!("结果 = {}", v),
        Err(e) => println!("错误 = {}", e),
    }
}
```

常用方法：

* `unwrap()` / `unwrap_or()` / `expect()`
* `map()` / `and_then()` / `or_else()`
* `is_ok() / is_err()`

---

## 5️⃣ 错误传播（? 运算符）

快速将错误返回给调用者：

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file(path: &str) -> io::Result<String> {
    let mut file = File::open(path)?;      // 出错直接返回 Err
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

fn main() {
    match read_file("test.txt") {
        Ok(text) => println!("{}", text),
        Err(e) => println!("读取失败: {}", e),
    }
}
```

> `?` 只能在返回 `Result` 或 `Option` 的函数中使用。

---

## 6️⃣ 自定义错误类型

* 使用 `enum` 定义错误类型
* 实现 `std::fmt::Display` 和 `std::error::Error`

```rust
use std::fmt;

#[derive(Debug)]
enum MyError {
    InvalidInput,
    DivideByZero,
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            MyError::InvalidInput => write!(f, "输入无效"),
            MyError::DivideByZero => write!(f, "除数为0"),
        }
    }
}

impl std::error::Error for MyError {}
```

函数返回自定义错误：

```rust
fn divide(a: i32, b: i32) -> Result<i32, MyError> {
    if b == 0 { return Err(MyError::DivideByZero); }
    Ok(a / b)
}
```

---

## 7️⃣ 错误链和上下文

* 使用 `anyhow` 或 `thiserror` crate
* 可保存错误来源和上下文，方便调试

```rust
use anyhow::{Result, Context};
use std::fs::File;

fn read_file(path: &str) -> Result<String> {
    let mut file = File::open(path).context("打开文件失败")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents).context("读取文件失败")?;
    Ok(contents)
}
```

> 错误链可以显示完整调用路径和原因。

---

## 8️⃣ 综合示例

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file(path: &str) -> io::Result<String> {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 { Err("除数不能为0".to_string()) } else { Ok(a / b) }
}

fn main() {
    // Result 错误处理
    match divide(10, 0) {
        Ok(v) => println!("结果 = {}", v),
        Err(e) => println!("错误 = {}", e),
    }

    // 使用 ? 错误传播
    match read_file("test.txt") {
        Ok(text) => println!("文件内容 = {}", text),
        Err(e) => println!("文件读取错误 = {}", e),
    }
}
```

---

## 9️⃣ 总结

1. Rust 错误分为 **不可恢复 (`panic!`)** 和 **可恢复 (`Result`)**
2. **Option** 用于可能缺失的值
3. **Result** 用于可能失败的操作
4. `?` 运算符简化错误传播
5. 自定义错误类型可以实现 Display 与 Error
6. 使用 `anyhow`/`thiserror` 可以方便处理错误链和上下文

---

💡 **练习建议**：

1. 编写函数读取文件并处理不存在文件的情况
2. 使用 Result 链式调用多个可能出错的函数
3. 自定义错误类型，实现更详细的错误信息
4. 结合 if let、match 安全处理 Option 和 Result
