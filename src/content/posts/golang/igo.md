---
title: Igo - Cross-Platform Go Build Tool
published: 2025-09-08
description: 'Igo 是一个跨平台 Go 构建与打包工具，支持 Windows、macOS、Linux 等多平台交叉编译与分发。'
image: ''
tags: [go, golang, build-tool, cross-platform, cli]
category: 'golang'
draft: false 
lang: 'zh'
---






# 🚀 Igo - Cross-Platform Go Build Tool

`Igo` 是一个强大的 **跨平台 Go 构建与打包命令行工具**，在 Go 原生 `go build` 的基础上，提供了更高级的能力：  
- 一次配置，多平台打包  
- 自动生成 `zip`、`tar`、`tar.gz` 等分发包  
- 进度可视化  
- 灵活的 YAML 配置  

非常适合需要 **跨平台交付二进制** 的 Go 项目。

---

## ✨ 特性

- **跨平台支持**：Windows、macOS、Linux 一键构建  
- **灵活配置**：基于 YAML，支持变量替换  
- **多种打包格式**：ZIP、TAR、TAR.GZ  
- **Go 原生命令扩展**：集成 run、build 等命令  
- **可定制**：自定义变量、编译规则  
- **进度可视化**：长任务不再枯燥  

---

## ⚡ 安装

### 前置条件
- Go 1.21+  
- Git  

### 从源码安装
```bash
git clone https://github.com/kawaiirei0/igo.git
cd igo
go build -o igo main.go
# 全局安装（可选）
go install
````

### 直接安装

```bash
go install github.com/kawaiirei0/igo@latest
```

---

## 🚀 快速开始

### 1. 初始化项目配置

```bash
igo init
# 或自定义配置文件名
igo init my-config
```

会生成一个 `igo.yaml` 配置文件。

---

### 2. 编辑配置

```yaml
project:
  name: "my-app"
  version: "1.0.0"
  main: "main.go"

build:
  platforms:
    - "windows/amd64"
    - "darwin/amd64"
    - "linux/amd64"
```

---

### 3. 构建项目

```bash
igo build                   # 所有平台
igo build --platforms "windows/amd64,linux/amd64"
igo build --output ./bin
```

---

### 4. 打包分发

```bash
igo package                 # 默认打包
igo package --format tar.gz # 指定格式
igo package --include "docs/,config/"
```

---

## ⚙️ 配置说明

### 项目信息

```yaml
project:
  name: "your-app-name"
  version: "1.0.0"
  description: "Your app description"
  main: "main.go"
  output: "app-name"
```

### 构建配置

```yaml
build:
  platforms:
    - "windows/amd64"
    - "darwin/amd64"
    - "linux/amd64"
  tags: ["release"]
  flags: ["-ldflags=-s -w"]
  env:
    CGO_ENABLED: "0"
  output_dir: "dist"
  type: "binary"
```

### 打包配置

```yaml
package:
  include:
    - "README.md"
    - "LICENSE"
  exclude:
    - ".git"
    - "*.tmp"
  format: "zip"
  output_dir: "packages"
  compression_level: 6
```

### 环境变量

```yaml
environment:
  APP_ENV: "production"
  DEBUG: "false"
```

### 自定义配置

```yaml
custom:
  company_name: "Your Company"
  config_name: "igo"
```

---

## 📦 命令列表

* `igo init [config-name]` → 初始化配置文件
* `igo run [args...]` → 运行 Go 程序
* `igo build` → 构建多平台二进制
* `igo package` → 打包分发
* `igo validate` → 校验配置
* `igo config` → 查看当前配置

---

## 🖥 支持平台

* **Windows**: amd64, 386, arm64
* **macOS**: amd64, arm64
* **Linux**: amd64, 386, arm, arm64
* **FreeBSD**: amd64, 386, arm, arm64

---

## 📂 支持的打包格式

* **ZIP**
* **TAR**
* **TAR.GZ**

---

## 🔧 变量替换

配置文件中可以用 `${}` 进行替换：

```yaml
custom:
  company: "Acme"
  version: "1.0.0"

build:
  output_dir: "dist/${custom_company}-${custom_version}"
```

可用变量：

* `${project_name}`
* `${project_version}`
* `${env_KEY}`
* `${custom_key}`

---

## 📚 示例

### Web 应用

```yaml
package:
  include:
    - "static/"
    - "templates/"
    - "config.yaml"
  exclude:
    - "*.log"
    - ".env"
```

### CLI 工具

```yaml
package:
  include:
    - "README.md"
    - "docs/"
    - "examples/"
  format: "tar.gz"
```

---

## 🛠 开发

```bash
git clone https://github.com/kawaiirei0/igo.git
cd igo
go build -o igo main.go
go test ./...
```

---

## 📌 Roadmap

* [ ] Docker 支持
* [ ] CI/CD 集成
* [ ] 插件系统
* [ ] GUI 界面
* [ ] 云端发布

---

## 📄 License

MIT License → [LICENSE](LICENSE)

---

## 🔗 支持与反馈

* [GitHub Issues](https://github.com/kawaiirei0/igo/issues)
* [GitHub Wiki](https://github.com/kawaiirei0/igo/wiki)
* [GitHub Discussions](https://github.com/kawaiirei0/igo/discussions)