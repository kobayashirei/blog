---
title: Minecraft 服务器 Linux 安装指南（支持 Java & NeoForge）
published: 2025-10-31
description: '本文档适用于 **Ubuntu/Debian/CentOS/Arch 等主流 x86_64/ARM Linux 系统**，可以快速搭建 Minecraft 服务器，并支持 NeoForge/Forge 等 mod 平台。'
image: ''
tags: [mc,server]
category: 'mc'
draft: false 
lang: ''
---


## 1️⃣ 系统准备

1. 确保服务器系统为 Linux x86_64 或 ARM 架构。
2. 安装必要工具：

```bash
sudo apt update && sudo apt install -y wget curl unzip tar
```

或在 CentOS:

```bash
sudo yum install -y wget curl unzip tar
```

---

## 2️⃣ 安装 MCSManager 面板（可选）

MCSManager 是一个网页面板，可以方便管理 Minecraft 服务器实例。

```bash
sudo su -c "wget -qO- https://script.mcsmanager.com/setup_cn.sh | bash"
```

安装完成后，访问面板：

```
http://<服务器IP>:23333
```

> 面板支持快速创建实例、管理启动参数、上传模组等操作。

---

## 3️⃣ 安装 Java 环境（推荐 Javam）

Javam 是一个便捷的 Java 管理工具，可以自动安装兼容 Minecraft 的 JDK 版本。

```bash
curl -sSL https://raw.githubusercontent.com/USYDShawnTan/javam/main/javam.sh | bash -s -- --install-only
```

---

## 4️⃣ 创建 Minecraft 服务器实例

### 4.1 下载服务器安装包

根据需求选择服务器类型：

* **NeoForge**（支持 mods 和 NeoForge 功能）
* **Forge**（原生 mod 支持）
* **Fabric**（如需 Fabric mod）

下载地址:

- https://neoforged.net/
- https://files.minecraftforge.net/net/minecraftforge/forge/

### 4.2 安装服务器

执行安装程序：

```bash
java -jar neoforge-21.1.213-installer.jar --installServer
```

* 安装路径可自定义，默认在当前目录生成服务器文件夹。
* 安装完成后会生成 `run.sh` 启动脚本。

---

## 5️⃣ 初次启动

首次启动用于生成配置文件、mods 文件夹及基础世界文件：

```bash
sh ./run.sh
```

首次启动可能会下载必要的库文件，请耐心等待。

---

## 6️⃣ 上传 Mods

1. 上传你喜欢的 mod 到服务器的 `mods` 文件夹：

```
/<项目目录>/mods/
```

2. 确保 mod 与服务器版本兼容，例如 NeoForge 21.1.x 对应 Minecraft 1.21.1。

> ⚠️ 注意：Java 版 mods 只对 Java 客户端生效，Bedrock/手机客户端无法使用。

---

## 7️⃣ 正式启动服务器

```bash
sh ./run.sh
```

---

## 8️⃣ 手机/平板访问（Bedrock 版）配置

### 8.1 安装 GeyserMC

GeyserMC 是一个 **Bedrock-to-Java 代理**，允许手机和平板客户端加入 Java 服务器。

1. 下载 GeyserMC 最新版本：

```bash
wget https://ci.opencollab.dev/job/GeyserMC/job/Geyser/job/master/lastSuccessfulBuild/artifact/bootstrap/standalone/target/Geyser-Standalone.jar -O geyser.jar
```

2. 创建文件夹并放入 Geyser：

```bash
mkdir -p /opt/minecraft/bedrock
mv geyser.jar /opt/minecraft/bedrock/
cd /opt/minecraft/bedrock
```

3. 启动 Geyser（首次启动会生成配置文件）：

```bash
java -jar geyser.jar
```

### 8.2 配置 Geyser

编辑 `config.yml`，主要配置：

```yaml
remote:
  address: 127.0.0.1   # Minecraft Java 服务器地址（本机服务器）
  port: 25565          # Java 服务器端口
  auth-type: online    # Mojang/微软账号验证

bedrock:
  address: 0.0.0.0
  port: 19132          # Bedrock 客户端访问端口
```

### 8.3 安装 Floodgate（免账号验证）

Floodgate 是 Geyser 的插件，允许手机无需 Microsoft/Bedrock 账号即可进入服务器。

1. 下载 Floodgate 最新版本：

```bash
wget https://ci.opencollab.dev/job/GeyserMC/job/Floodgate/job/master/lastSuccessfulBuild/artifact/standalone/target/floodgate.jar
```

2. 放入 Geyser 的 `plugins` 文件夹：

```
/opt/minecraft/bedrock/plugins/
```

3. 重启 Geyser：

```bash
java -jar geyser.jar
```

---

## 9️⃣ 访问方式

* **Java 客户端**：`<服务器IP>:25565`
* **Bedrock/手机客户端**：`<服务器IP>:19132`

> ⚠️ 请确保防火墙允许 25565（Java）和 19132（Bedrock）端口。

---

* 服务器启动后即可通过 **Java 客户端** 访问：`<服务器IP>:25565`
* 如果需要支持手机/平板等 Bedrock 客户端，可后续安装 **GeyserMC + Floodgate**。

---

## 8️⃣ 高级管理（可选）

* **MCSManager 面板**：可以通过面板管理启动、停止、上传 mods、备份世界。
* **自定义启动参数**：修改 `run.sh` 或 `server.properties`，调整内存、端口、最大玩家数等。
* **自动更新 Java**：使用 Javam 管理工具更新 Java 版本，保证服务器稳定性。

---

## 9️⃣ 小贴士

1. **备份世界**：定期备份 `world` 文件夹，防止意外丢失。
2. **安全性**：开放端口 25565，注意防火墙配置；若面板公网访问，建议设置密码或反向代理。
3. **Mods 兼容性**：安装前最好查看 mod 官方文档，避免版本冲突。


---

✅ 完成以上步骤后，你的服务器即可实现 **电脑 + 手机同时访问** Minecraft Modded 世界。
