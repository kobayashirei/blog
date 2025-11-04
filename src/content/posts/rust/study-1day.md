---
title: rust 1day
published: 2025-11-02
description: 'rust 1day'
image: ''
tags: [rust]
category: 'rust'
draft: false 
lang: ''
---

# Rust 变量与可变性示例项目

目录结构：

```
rust_vars_demo/
└── src/
    └── main.rs
```

`Cargo.toml`：

```toml
[package]
name = "rust_vars_demo"
version = "0.1.0"
edition = "2021"
```

---

`src/main.rs`：

```rust
// 常量示例
const PI: f64 = 3.1415926;

fn main() {
    println!("--- Rust 变量与可变性示例 ---");

    // 1️⃣ 不可变变量（默认）
    let x = 5;
    println!("不可变变量 x = {}", x);
    // x = 10; // ❌ 会报错

    // 2️⃣ 可变变量
    let mut y = 10;
    println!("可变变量 y 初始值 = {}", y);
    y = 15;
    println!("可变变量 y 修改后 = {}", y);

    // 3️⃣ Shadowing（遮蔽）
    let z = 20;
    let z = z + 5; // z 被遮蔽为新变量
    println!("Shadowing 后 z = {}", z);

    {
        let z = z * 2; // 内层作用域遮蔽
        println!("内层作用域 z = {}", z);
    }

    println!("外层作用域 z = {}", z);

    // 4️⃣ 常量
    println!("常量 PI = {}", PI);

    // 5️⃣ 函数参数的可变性
    let mut value = 50;
    println!("函数前 value = {}", value);
    add_ten(&mut value);
    println!("函数后 value = {}", value);
}

// 函数演示可变参数（使用可变引用）
fn add_ten(val: &mut i32) {
    *val += 10;
    println!("函数内修改 val = {}", val);
}
```

---

## ✅ 运行示例

1. 创建项目：

```bash
cargo new rust_vars_demo
cd rust_vars_demo
```

2. 替换 `src/main.rs` 内容为上面的代码

3. 运行：

```bash
cargo run
```

输出示例：

```
--- Rust 变量与可变性示例 ---
不可变变量 x = 5
可变变量 y 初始值 = 10
可变变量 y 修改后 = 15
Shadowing 后 z = 25
内层作用域 z = 50
外层作用域 z = 25
常量 PI = 3.1415926
函数前 value = 50
函数内修改 val = 60
函数后 value = 60
```

---

这个示例覆盖了：

* 默认不可变变量
* 可变变量 `mut`
* Shadowing（遮蔽）
* 常量 `const`
* 函数可变参数（通过可变引用 `&mut`）
