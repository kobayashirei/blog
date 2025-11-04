---
title: Rust Cargo 使用指南
published: 2025-11-02
description: 'Rust Cargo 使用指南'
image: ''
tags: [rust,cargo]
category: 'rust'
draft: false 
lang: ''
---

# Rust Cargo 使用指南

`Cargo` 是 Rust 官方的 **包管理器和构建工具**，负责：

* 管理依赖（Crates）
* 构建项目（`build`、`run`）
* 发布库或应用（`publish`）
* 管理工作区（Workspace）

---

## 1️⃣ 创建项目

### 新建一个二进制项目（可执行程序）

```bash
cargo new myapp
cd myapp
```

生成目录结构：

```
myapp/
├── Cargo.toml   # 项目信息和依赖
└── src/
    └── main.rs  # 入口文件
```

### 新建库项目

```bash
cargo new mylib --lib
```

生成：

```
mylib/
├── Cargo.toml
└── src/
    └── lib.rs
```

---

## 2️⃣ 构建项目

### 编译项目

```bash
cargo build       # Debug 模式，默认
cargo build --release  # Release 模式，优化编译
```

编译后的可执行文件：

* Debug：`target/debug/myapp`
* Release：`target/release/myapp`

### 运行项目

```bash
cargo run
```

相当于：

```bash
cargo build
./target/debug/myapp
```

### 检查代码

```bash
cargo check
```

* 只做语法检查，不生成可执行文件
* 编译速度快，用于开发阶段

---

## 3️⃣ 管理依赖

### 添加依赖

编辑 `Cargo.toml`：

```toml
[dependencies]
regex = "1.11.0"   # 添加 regex 库
```

或者通过命令添加：

```bash
cargo add regex
```

### 更新依赖

```bash
cargo update
```

### 删除依赖

```bash
cargo remove regex
```

---

## 4️⃣ 运行测试

### 创建测试

在 `src/lib.rs` 或 `src/main.rs` 添加：

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

### 运行测试

```bash
cargo test
```

---

## 5️⃣ 发布库（Crate）

1. 登录 crates.io：

```bash
cargo login <API_TOKEN>
```

2. 发布：

```bash
cargo publish
```

---

## 6️⃣ 工作区（Workspace）

适用于多包项目（比如一个仓库里多个 crate）：

**目录结构：**

```
myworkspace/
├── Cargo.toml      # workspace 配置
├── crate1/
│   └── Cargo.toml
└── crate2/
    └── Cargo.toml
```

**workspace Cargo.toml 示例：**

```toml
[workspace]
members = ["crate1", "crate2"]
```

然后可以统一编译：

```bash
cargo build
cargo run -p crate1
```

---

## 7️⃣ 常用 Cargo 命令总结

| 命令                      | 功能         |
| ----------------------- | ---------- |
| `cargo new <name>`      | 创建新项目      |
| `cargo build`           | 编译项目       |
| `cargo build --release` | 编译 Release |
| `cargo run`             | 编译并运行      |
| `cargo check`           | 快速检查代码     |
| `cargo test`            | 运行测试       |
| `cargo update`          | 更新依赖       |
| `cargo add <crate>`     | 添加依赖       |
| `cargo remove <crate>`  | 移除依赖       |
| `cargo publish`         | 发布 crate   |
| `cargo doc --open`      | 生成并打开文档    |

---

## 8️⃣ 小技巧

1. **依赖版本写法**

   ```toml
   regex = "1.11"     # 1.11.x 最新版本
   regex = "^1.11.0"  # >=1.11.0, <2.0
   ```

2. **查看依赖树**

```bash
cargo tree
```

3. **快速运行带参数**

```bash
cargo run -- arg1 arg2
```
