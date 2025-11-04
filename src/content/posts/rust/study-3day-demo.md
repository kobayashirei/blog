---
title: study 3day demo
published: 2025-11-03
description: 'Rust 函数参数与返回值中的所有权 Demo'
image: ''
tags: [rust]
category: 'rust'
draft: false 
lang: ''
---

# 🦀 Rust 函数参数与返回值中的所有权

## 目录

1. 函数参数的所有权规则
2. 堆数据的移动（Move）
3. 克隆（Clone）
4. 引用（Borrowing）
5. 函数返回值的所有权
6. 综合示例
7. 总结

---

## 1️⃣ 函数参数的所有权规则

Rust 函数传参默认会 **转移（Move）所有权**：

```rust
fn take_ownership(s: String) {
    println!("我拥有: {}", s);
} // s 离开作用域，自动释放堆内存

fn main() {
    let s = String::from("Rust");
    take_ownership(s);
    // println!("{}", s); // ❌ s 已被移动
}
```

> 栈类型（如 `i32`）传参时会 **拷贝**，不影响原变量：

```rust
fn add_one(x: i32) -> i32 { x + 1 }

fn main() {
    let a = 5;
    let b = add_one(a);
    println!("a = {}, b = {}", a, b); // a 仍然可用
}
```

---

## 2️⃣ 堆数据的移动（Move）

传递堆数据（`String`, `Vec` 等）时默认发生 **移动**：

```rust
fn append_world(s: String) -> String {
    let mut s = s;
    s.push_str(" world");
    s
}

fn main() {
    let s1 = String::from("Hello");
    let s2 = append_world(s1); // s1 移动到函数
    // println!("{}", s1); // ❌ s1 已失效
    println!("{}", s2); // Hello world
}
```

> 注意：如果函数不返回值，堆内存会在函数结束时被释放。

---

## 3️⃣ 克隆（Clone）传参

如果想保留原变量，可 **显式克隆**：

```rust
fn append_exclamation(mut s: String) -> String {
    s.push('!');
    s
}

fn main() {
    let s1 = String::from("Hi");
    let s2 = append_exclamation(s1.clone()); // 克隆一份
    println!("s1 = {}, s2 = {}", s1, s2);
}
```

输出：

```
s1 = Hi, s2 = Hi!
```

---

## 4️⃣ 引用（Borrowing）

可以通过引用借用堆数据，不移动所有权：

```rust
fn print_length(s: &String) {
    println!("长度是 {}", s.len());
}

fn main() {
    let s = String::from("Rustacean");
    print_length(&s); // 借用，不转移所有权
    println!("原字符串依然可用: {}", s);
}
```

> 可变引用允许函数修改数据：

```rust
fn add_suffix(s: &mut String) {
    s.push_str("!!!");
}

fn main() {
    let mut s = String::from("Hello");
    add_suffix(&mut s);
    println!("{}", s); // Hello!!!
}
```

⚠️ 注意：同一时间不能有多个可变引用。

---

## 5️⃣ 函数返回值的所有权

函数可以 **返回堆数据**，将所有权传给调用者：

```rust
fn give_ownership() -> String {
    let s = String::from("Ownership");
    s // 返回所有权
}

fn main() {
    let s1 = give_ownership(); // s1 获得所有权
    println!("{}", s1);
}
```

### 返回引用（Borrowing）需谨慎

```rust
fn invalid_ref() -> &String {
    let s = String::from("Hi");
    &s // ❌ 编译错误，s 会在函数结束释放
}
```

> 函数返回引用必须保证引用有效（通常返回参数引用或全局数据）。

---

## 6️⃣ 综合示例

```rust
fn main() {
    let mut s = String::from("Hello");

    // 借用不可变引用
    print_length(&s);

    // 借用可变引用
    add_suffix(&mut s);

    // 移动到函数并返回
    let s2 = append_world(s);
    println!("{}", s2);

    // 克隆保留原数据
    let s3 = append_exclamation(s2.clone());
    println!("s2 = {}, s3 = {}", s2, s3);
}

// 函数定义
fn print_length(s: &String) { println!("长度: {}", s.len()); }
fn add_suffix(s: &mut String) { s.push_str("!!!"); }
fn append_world(s: String) -> String {
    let mut s = s;
    s.push_str(" world");
    s
}
fn append_exclamation(mut s: String) -> String { s.push('!'); s }
```

---

## 7️⃣ 总结

1. **栈数据**传递默认拷贝，不影响原变量。
2. **堆数据**传递默认移动（Move）。
3. **克隆**用于显式复制堆数据。
4. **借用引用**避免移动，同时保持原变量有效：

   * 不可变引用：允许读取，不能修改
   * 可变引用：允许修改，借用规则严格
5. **函数返回值**可以转移所有权给调用者
6. 返回引用必须保证引用有效，防止悬垂引用

---

💡 **练习建议**：

1. 尝试写一个函数接受 `Vec<String>` 并返回其中最长的字符串，考虑移动和引用
2. 修改函数，让它可以在不移动原 `Vec` 的情况下返回最大长度
3. 尝试使用克隆与借用结合，提高内存安全性
