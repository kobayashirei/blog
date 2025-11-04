---
title: rust study 5day
published: 2025-11-03
description: 'Rust 切片（Slice）教程'
image: ''
tags: [rust]
category: 'rust'
draft: false 
lang: ''
---


# 🦀 Rust 切片（Slice）教程

## 目录

1. 切片概念
2. 字符串切片（String & str）
3. 数组切片（Array & [T]）
4. 可变切片（&mut [T]）
5. 切片函数参数
6. 切片与借用关系
7. 综合示例
8. 总结

---

## 1️⃣ 切片概念

**切片（Slice）**是对集合中一部分数据的引用。

* 不拥有数据，不会触发移动
* 使用借用语法访问集合中的连续元素
* 支持只读切片（`&[T]`）和可变切片（`&mut [T]`）

```rust
fn main() {
    let arr = [1, 2, 3, 4, 5];
    let slice = &arr[1..4]; // 借用 arr 中索引 1..3 的元素
    println!("{:?}", slice); // [2, 3, 4]
}
```

> `[start..end]` 表示左闭右开区间 `[start, end)`

---

## 2️⃣ 字符串切片（String & str）

Rust 的 `String` 可以被切成 **字符串切片 `&str`**：

```rust
fn main() {
    let s = String::from("Hello Rust");
    let slice = &s[0..5]; // 前五个字符
    println!("{}", slice); // Hello
}
```

### 字符边界注意事项

* Rust 字符串是 UTF-8 编码，不能按字节随意切
* 切片必须落在字符边界上，否则编译错误

```rust
let s = String::from("你好Rust");
// let slice = &s[0..3]; // ❌ 编译错误
let slice = &s[0..6]; // ✅ 正确（每个汉字占 3 字节）
```

---

## 3️⃣ 数组切片（Array & [T]）

数组 `[T; N]` 可以通过切片访问部分元素：

```rust
fn main() {
    let arr = [10, 20, 30, 40, 50];
    let slice = &arr[1..4]; // 索引 1,2,3
    println!("{:?}", slice); // [20, 30, 40]
}
```

* 完整切片：`&arr[..]`
* 从指定索引到末尾：`&arr[2..]`
* 从开头到指定索引：`&arr[..3]`

---

## 4️⃣ 可变切片（&mut [T]）

可修改集合中的元素：

```rust
fn main() {
    let mut arr = [1, 2, 3, 4, 5];
    let slice = &mut arr[1..4]; // 可变切片
    slice[0] += 10;
    slice[2] *= 2;
    println!("{:?}", arr); // [1, 12, 3, 8, 5]
}
```

> 与可变引用类似，同一时间**只能有一个可变切片**，避免数据竞争。

---

## 5️⃣ 切片函数参数

切片非常适合函数参数，可以**不移动原数据**，而是借用访问：

```rust
fn sum(slice: &[i32]) -> i32 {
    slice.iter().sum()
}

fn main() {
    let arr = [1, 2, 3, 4];
    let s = &arr[1..3];
    println!("sum = {}", sum(s)); // 2 + 3 = 5
}
```

可变切片传递允许函数修改原数据：

```rust
fn double(slice: &mut [i32]) {
    for x in slice.iter_mut() {
        *x *= 2;
    }
}

fn main() {
    let mut arr = [1, 2, 3, 4];
    double(&mut arr[1..3]);
    println!("{:?}", arr); // [1, 4, 6, 4]
}
```

---

## 6️⃣ 切片与借用关系

切片本质上是 **引用的一种形式**：

* 切片不会复制数据，只是借用
* 所以切片遵守借用规则：

  * 多个不可变切片可以同时存在
  * 一个可变切片同时只能存在一个

```rust
let mut arr = [1,2,3,4];
let s1 = &arr[0..2];
let s2 = &arr[2..4]; // ✅ 不重叠，可同时存在
let s3 = &mut arr[1..3]; // ❌ 编译错误，与 s1/s2 重叠
```

> Rust 编译器通过静态检查保证安全。

---

## 7️⃣ 综合示例

```rust
fn main() {
    let mut data = [10, 20, 30, 40, 50];

    // 不可变切片
    let s1 = &data[1..4];
    println!("不可变切片 s1 = {:?}", s1);

    // 可变切片
    let s2 = &mut data[2..5];
    for x in s2.iter_mut() {
        *x += 5;
    }
    println!("修改后的数组 = {:?}", data);

    // 字符串切片
    let s = String::from("Rust语言");
    let slice = &s[0..4]; // 前两个字符 "Rust"
    println!("字符串切片: {}", slice);
}
```

输出：

```
不可变切片 s1 = [20, 30, 40]
修改后的数组 = [10, 20, 35, 45, 55]
字符串切片: Rust
```

---

## 8️⃣ 总结

1. 切片是对集合的一部分的 **引用**，不拥有数据
2. 支持 **数组切片** 和 **字符串切片**
3. 可变切片允许修改原数据
4. 切片遵循 Rust **借用规则**，保证内存安全
5. 切片作为函数参数非常高效，避免数据移动

---

💡 **练习建议**：

1. 编写函数，接受数组切片，返回最大值
2. 尝试用可变切片修改数组的部分元素
3. 对字符串使用切片获取指定字符子串，注意 UTF-8 边界
4. 实践切片在函数参数中的借用，提高性能
