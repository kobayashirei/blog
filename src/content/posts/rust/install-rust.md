---
title: Rust 安装教程（完整版）
published: 2025-11-02
description: 'Rust 安装教程（完整版）'
image: ''
tags: [install,rust]
category: 'rust'
draft: false 
lang: ''
---


# Rust 安装教程（完整版）

## 1️⃣ 安装 Rust

Rust 官方推荐使用 **rustup** 来管理 Rust 版本，它可以同时管理稳定版、beta、nightly，且支持多平台。

### Linux / macOS

打开终端，执行：

```bash
# 下载并安装 rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

安装过程中会提示：

* 默认安装 stable 版本（推荐）
* 自动配置 `PATH`（如果选择 yes，会修改 `.bashrc` 或 `.zshrc`）

安装完成后，刷新环境变量：

```bash
source $HOME/.cargo/env
```

### Windows

1. 下载官方安装器：[https://rustup.rs/](https://rustup.rs/)
2. 双击运行 `rustup-init.exe`，选择默认安装即可
3. 安装完成后重启命令行或 PowerShell

---

## 2️⃣ 验证安装

```bash
# 查看 Rust 版本
rustc --version

# 查看 Cargo 版本
cargo --version

# 查看 rustup 版本
rustup --version
```

> 示例输出：

```
rustc 1.78.0 (stable)
cargo 1.78.0 (stable)
rustup 1.30.0
```

---

## 3️⃣ Rust 常用组件

Rust 官方提供一些常用组件，可以通过 rustup 安装：

```bash
# 安装 Clippy（代码检查工具）
rustup component add clippy

# 安装 Rustfmt（代码格式化工具）
rustup component add rustfmt

# 安装 Rust source（便于 IDE 智能提示）
rustup component add rust-src
```

---

## 4️⃣ 安装 IDE / 编辑器支持

### 1. Visual Studio Code

* 安装 VS Code
* 安装扩展：

  * Rust Analyzer (推荐)
  * CodeLLDB（调试）
* 配置 Rust 工具链路径（通常 rustup 已自动配置）

### 2. IntelliJ IDEA / CLion

* 安装 Rust 插件（官方 Rust plugin）
* 配置 rustup 工具链

---

## 5️⃣ 配置多版本 Rust（可选）

```bash
# 查看可用工具链
rustup show

# 安装 nightly 版本
rustup install nightly

# 切换到 nightly
rustup default nightly

# 针对单个项目使用 nightly
cd my_project
rustup override set nightly
```

---

## 6️⃣ 创建第一个 Rust 项目

```bash
# 创建新项目
cargo new hello_world
cd hello_world

# 编译并运行
cargo run
```

输出：

```
Hello, world!
```

---

## 7️⃣ 更新 Rust

Rust 更新比较频繁，推荐使用 rustup 定期更新：

```bash
# 更新 rustup 本身
rustup self update

# 更新工具链
rustup update
```

---

## 8️⃣ 卸载 Rust

如果需要卸载 Rust：

```bash
rustup self uninstall
```

这会删除 Rust、Cargo、rustup 以及相关组件。

---

💡 **小技巧**：

* Cargo 是 Rust 的包管理工具，同时管理依赖、构建和运行。
* 推荐在 Linux/macOS 上使用 rustup 安装，Windows 可以通过官方安装器。
* 对于开发 Rust Web / CLI / WASM 项目，建议同时安装 nightly 和 stable，方便使用最新特性。
