---
title: 初学rust
published: 2025-09-09
description: '初学rust笔记'
image: ''
tags: [rust,init-rust,init-study-rust,study-rust]
category: 'rust'
draft: false 
lang: 'zh'
---

# 📚 Rust 初学文档教程（入门 → 进阶 → 实战）

## 1. 入门准备

### 1.1 安装 Rust

* 安装 Rust：

  ```bash
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

  （Windows 用户建议用 [官方 MSI 安装器](https://www.rust-lang.org/zh-CN/tools/install)）

* 常用命令：

  ```bash
  rustc --version   # 查看版本
  cargo --version   # 查看版本（包管理/构建工具）
  rustup update     # 更新 Rust
  ```

### 1.2 开发环境

* 推荐 IDE：**VSCode + rust-analyzer 插件**
* 格式化工具：`rustfmt`
* 代码检查：`clippy`

---

## 2. 基础语法

（建议直接使用 Cargo 创建小项目练习）

### 2.1 Hello World

```bash
cargo new hello_rust
cd hello_rust
cargo run
```

### 2.2 基础语法要点

* 变量与常量（`let`、`mut`、`const`）
* 数据类型（整数、浮点数、布尔、字符、字符串、数组、元组）
* 所有权（Rust 最核心概念）
* 借用与引用（`&T`、`&mut T`）
* Slice 切片
* 控制流（`if`、`loop`、`while`、`for`）

---

## 3. 核心概念

### 3.1 所有权与生命周期

* 所有权规则
* 移动（move）
* 克隆（clone）
* 借用（borrow）
* 生命周期（`'a`）

### 3.2 函数与模块

* 函数定义、参数与返回值
* 模块 system（`mod`、`pub`）
* 包与 crate（`lib.rs`、`main.rs`）
* Cargo.toml 配置依赖

### 3.3 结构体 & 枚举

* `struct` 定义、方法实现（`impl`）
* `enum` 与 `match`
* 模式匹配与解构

### 3.4 错误处理

* `Option<T>` 与 `Result<T, E>`
* `?` 运算符简化错误传播
* 自定义错误类型

---

## 4. 中级进阶

### 4.1 泛型与 Trait

* 泛型函数、结构体
* Trait 定义与实现
* `derive` 自动实现（如 `Debug`, `Clone`, `PartialEq`）

### 4.2 集合与迭代器

* `Vec`, `HashMap`, `HashSet`
* 迭代器（`iter`, `map`, `filter`, `collect`）
* 自定义迭代器

### 4.3 智能指针

* `Box<T>`
* `Rc<T>` 与 `Arc<T>`
* `RefCell<T>` 与 `Mutex<T>`

### 4.4 并发编程

* 线程（`std::thread`）
* 消息传递（`mpsc` 通道）
* 多线程共享（`Arc<Mutex<T>>`）
* async/await 异步编程

---

## 5. 高级特性

* 生命周期标注
* 高级 Trait（如 `Send`, `Sync`）
* 宏（`macro_rules!`）
* unsafe Rust（裸指针、FFI、内联汇编）
* Zero-cost abstraction 原则

---

## 6. 工程化

### 6.1 Cargo 高级用法

* Cargo profiles（debug/release）
* Cargo workspace
* Cargo features

### 6.2 测试与文档

* 单元测试（`#[test]`）
* 集成测试（`tests` 文件夹）
* 文档注释（`///`）
* `cargo doc --open`

### 6.3 常用工具链

* `serde`：序列化/反序列化 JSON
* `tokio`：异步运行时
* `reqwest`：HTTP 客户端
* `axum` / `actix-web`：Web 框架
* `diesel` / `sqlx`：数据库 ORM
* `tracing`：日志与监控

---

## 7. 实战项目建议

1. **CLI 工具**（命令行解析 → `clap`/`structopt`）
2. **Web 服务**（`axum`/`actix-web` + `sqlx` + `serde`）
3. **异步爬虫**（`reqwest` + `tokio` + `scraper`）
4. **分布式服务**（学习 `tonic` gRPC）
5. **操作系统内核**（Rust for OS Dev，深入系统编程）

---

## 8. 学习资源

* 📖 《Rust 程序设计语言》（The Book） → [https://doc.rust-lang.org/book/](https://doc.rust-lang.org/book/)
* 📖 《Rust 标准库文档》 → [https://doc.rust-lang.org/std/](https://doc.rust-lang.org/std/)
* 📖 《Rust By Example》 → [https://doc.rust-lang.org/rust-by-example/](https://doc.rust-lang.org/rust-by-example/)
* 🏗️ 实战练习网站 → [https://exercism.org/tracks/rust](https://exercism.org/tracks/rust)
* 🦀 Rust 社区 → [https://users.rust-lang.org/](https://users.rust-lang.org/)

---

⚡ **推荐学习顺序：**

```
安装环境 → 基础语法 → 所有权/借用 → 结构体/枚举 → 错误处理 → 泛型/Trait → 集合/迭代器 
→ 智能指针 → 并发/异步 → 工程化工具 → 实战项目
```

---

要不要我帮你做一个 **循序渐进的学习计划表（按周安排学习内容 + 小练习）**，这样你每天跟着走就能扎实入门 Rust？
