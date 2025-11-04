---
title: rust study 4day
published: 2025-11-03
description: '深入讲解 Rust 的 **借用（Borrowing）与引用（References）**，这是理解所有权系统、避免数据移动、实现安全访问的核心概念。'
image: ''
tags: [rust]
category: 'rust'
draft: false 
lang: ''
---

# 🦀 Rust 借用（References & Borrowing）教程

## 目录

1. 借用概念
2. 不可变引用（&T）
3. 可变引用（&mut T）
4. 引用规则
5. 函数中的借用
6. 生命周期与引用
7. 综合示例
8. 总结

---

## 1️⃣ 借用概念

**借用（Borrowing）**是指：

* 不获取数据的所有权，而是“借用”原数据的访问权
* 借用可以**读取**或**修改**数据（取决于是不可变引用还是可变引用）
* 借用不会触发所有权移动，所以原变量在借用结束后依然有效

```rust
fn main() {
    let s = String::from("Hello");
    let r = &s; // 借用 s，r 是引用
    println!("r = {}", r);
    println!("s = {}", s); // s 仍然可用
}
```

> 借用分为两类：不可变借用和可变借用。

---

## 2️⃣ 不可变引用（&T）

* 语法：`&变量名`
* 特性：

  1. 只能读取数据
  2. 可以同时有多个不可变引用

```rust
fn main() {
    let s = String::from("Rust");

    let r1 = &s;
    let r2 = &s;

    println!("r1 = {}, r2 = {}", r1, r2);
    println!("s = {}", s);
}
```

输出：

```
r1 = Rust, r2 = Rust
s = Rust
```

> 多个不可变引用不会导致数据竞争。

---

## 3️⃣ 可变引用（&mut T）

* 语法：`&mut 变量名`
* 特性：

  1. 可以修改原数据
  2. **同一时间只能有一个可变引用**

```rust
fn main() {
    let mut s = String::from("Hello");

    let r = &mut s;
    r.push_str(", world");

    println!("{}", r); // Hello, world
}
```

> 如果在可变引用存在时尝试创建不可变引用或另一个可变引用，编译器会报错。

```rust
let mut s = String::from("Hello");
let r1 = &s;
let r2 = &mut s; // ❌ 编译错误：不能同时有不可变和可变引用
```

---

## 4️⃣ 引用规则（Borrowing Rules）

Rust 借用规则保证数据安全：

1. 任意时刻可以有多个不可变引用（读取）
2. 或者一个可变引用（写入）
3. 不可同时存在可变引用和不可变引用
4. 引用必须始终有效（避免悬垂指针）

---

## 5️⃣ 函数中的借用

借用使函数能在 **不获取所有权** 的情况下访问或修改数据。

### 不可变引用作为参数

```rust
fn print_length(s: &String) {
    println!("长度 = {}", s.len());
}

fn main() {
    let s = String::from("Rustacean");
    print_length(&s); // 借用，不移动
    println!("原字符串仍可用: {}", s);
}
```

### 可变引用作为参数

```rust
fn add_exclamation(s: &mut String) {
    s.push_str("!");
}

fn main() {
    let mut s = String::from("Hello");
    add_exclamation(&mut s);
    println!("{}", s); // Hello!
}
```

---

## 6️⃣ 生命周期与引用

* 引用的有效期必须 ≤ 被引用数据的生命周期
* Rust 编译器通过 **生命周期分析** 检查引用安全性

```rust
fn invalid_reference() -> &String {
    let s = String::from("Hello");
    &s // ❌ s 会在函数结束被释放
}
```

> 正确做法：返回数据所有权或返回已有引用（函数参数引用）。

---

## 7️⃣ 综合示例

```rust
fn main() {
    let mut s = String::from("Rust");

    // 不可变引用
    let r1 = &s;
    let r2 = &s;
    println!("r1={}, r2={}", r1, r2);

    // r1, r2 使用完毕后，可创建可变引用
    let r3 = &mut s;
    r3.push_str("acean");
    println!("{}", r3);

    // 函数借用示例
    print_length(&s);
    add_exclamation(&mut s);
    println!("最终字符串: {}", s);
}

fn print_length(s: &String) { println!("长度: {}", s.len()); }
fn add_exclamation(s: &mut String) { s.push('!'); }
```

输出：

```
r1=Rust, r2=Rust
Rustacean
长度: 9
最终字符串: Rustacean!
```

---

## 8️⃣ 总结

1. 借用允许函数或变量访问数据而 **不转移所有权**
2. 不可变引用（`&T`）可同时存在多个，只读
3. 可变引用（`&mut T`）一次只能有一个，可修改
4. 借用规则保证内存安全和避免数据竞争
5. 生命周期与引用密切相关，Rust 编译器会检查引用有效性

---

💡 **练习建议**：

1. 编写函数接受多个不可变引用，打印内容
2. 编写函数接受可变引用，修改数据
3. 尝试同时创建可变引用和不可变引用，观察编译器报错
4. 练习在函数返回值中安全传递引用
