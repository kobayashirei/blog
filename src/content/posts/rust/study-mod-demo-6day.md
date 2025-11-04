---
title: Rust 模块系统（mod）教程 6day
published: 2025-11-03
description: 'Rust 模块系统（mod）教程 6day'
image: ''
tags: [rust,rust-mod]
category: 'rust'
draft: false 
lang: ''
---

# 🦀 Rust 模块系统（mod）教程

## 目录

1. 模块概念
2. 基本语法
3. 文件与目录结构
4. pub 与私有性
5. 引入模块（use）
6. 嵌套模块
7. 进阶技巧：路径、重导出
8. 综合示例
9. 总结

---

## 1️⃣ 模块概念

Rust 的模块（module）是 **组织代码的单位**：

* 将函数、结构体、枚举、常量等逻辑分组
* 避免命名冲突
* 控制访问范围（公开或私有）

> 模块可以嵌套形成层次结构。

---

## 2️⃣ 基本语法

```rust
mod network {
    pub fn connect() {
        println!("Connecting...");
    }

    fn private_function() {
        println!("这是私有函数");
    }
}

fn main() {
    network::connect();      // ✅ 可访问
    // network::private_function(); // ❌ 编译错误
}
```

* `mod xxx` 定义模块
* 默认模块成员是 **私有的**，使用 `pub` 可公开

---

## 3️⃣ 文件与目录结构

Rust 可以将模块分散到不同文件：

### 文件结构示例

```
src/
├── main.rs
├── network.rs
└── utils/
    ├── mod.rs
    └── math.rs
```

### main.rs

```rust
mod network;  // 引入 network.rs 模块
mod utils;    // 引入 utils/mod.rs 模块

fn main() {
    network::connect();
    utils::math::add(2, 3);
}
```

### network.rs

```rust
pub fn connect() {
    println!("Connecting...");
}
```

### utils/mod.rs

```rust
pub mod math; // 引入 math.rs
```

### utils/math.rs

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

> `mod.rs` 用作目录模块的入口文件（Rust 2018+ 还可以用 `utils.rs` + `utils/` 子模块方式）。

---

## 4️⃣ pub 与私有性

* 默认私有：模块内部可以访问，外部不可访问
* `pub`：公开访问
* `pub(crate)`：同包可访问
* `pub(super)`：父模块可访问
* `pub(self)`：当前模块可访问（冗余，但可用）

```rust
mod outer {
    pub mod inner {
        pub fn hello() { println!("Hello"); }
        fn secret() { println!("Secret"); }
    }
}
fn main() {
    outer::inner::hello(); // ✅ 可访问
    // outer::inner::secret(); // ❌ 编译错误
}
```

---

## 5️⃣ 引入模块（use）

`use` 可以简化模块路径访问：

```rust
mod network {
    pub fn connect() { println!("Connecting..."); }
}

use network::connect;

fn main() {
    connect(); // 不需要写 network::connect
}
```

### 导入全部内容

```rust
use network::*;
```

### 起别名

```rust
use network::connect as net_connect;
fn main() { net_connect(); }
```

---

## 6️⃣ 嵌套模块

模块可以无限嵌套：

```rust
mod app {
    pub mod network {
        pub fn connect() { println!("Connecting..."); }
    }

    pub mod db {
        pub fn query() { println!("Querying..."); }
    }
}

fn main() {
    app::network::connect();
    app::db::query();
}
```

> 使用 `use` 可以简化路径：

```rust
use app::network::connect;
connect();
```

---

## 7️⃣ 进阶技巧

### 1. 重导出（pub use）

```rust
mod network {
    pub fn connect() { println!("Connecting..."); }
}

pub use network::connect; // 外部可以直接调用 connect
```

### 2. 相对路径

```rust
mod outer {
    pub mod inner {
        pub fn hello() { println!("Hello"); }
    }
    pub fn call_inner() {
        super::outer::inner::hello(); // 或使用 crate::outer::inner::hello()
    }
}
```

### 3. 内部私有访问

模块内部可以直接访问同级模块的私有成员。

---

## 8️⃣ 综合示例

项目结构：

```
src/
├── main.rs
├── network.rs
└── utils/
    ├── mod.rs
    └── math.rs
```

### main.rs

```rust
mod network;
mod utils;

use network::connect;
use utils::math::add;

fn main() {
    connect();
    println!("2 + 3 = {}", add(2, 3));
}
```

### network.rs

```rust
pub fn connect() {
    println!("Connecting to server...");
}
```

### utils/mod.rs

```rust
pub mod math;
```

### utils/math.rs

```rust
pub fn add(a: i32, b: i32) -> i32 { a + b }
```

运行：

```
Connecting to server...
2 + 3 = 5
```

---

## 9️⃣ 总结

1. `mod` 用于声明模块，默认私有
2. `pub` 用于公开模块或成员
3. 文件模块：`mod.rs` 或 `mod_name.rs`
4. `use` 用于引入模块并简化路径
5. 模块支持嵌套与重导出
6. Rust 模块系统支持清晰、可维护、可重用的项目结构

---

💡 **练习建议**：

1. 创建一个 `src/services/` 目录，包含 `network.rs` 和 `db.rs` 模块
2. 使用 `use` 在 `main.rs` 调用模块函数
3. 尝试使用 `pub(crate)` 和 `pub(super)` 管理访问范围
4. 尝试重导出模块函数，简化调用路径
