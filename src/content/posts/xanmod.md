---
title: "优秀的第三方Linux内核XanMod，白嫖性能提升！"
published: 2025-11-19
tags: ["技术教程", "随笔"]
category: "技术教程"
---

很多 VPS 自带的内核要么缺少 TCP BBR，要么版本老旧连 zstd 都不支持，于是我们的 Xanmod 就要上场了。这是一个通用性很强的第三方 Linux 内核，支持各种高级优化特性，为你的 VPS 更换它几乎相当于完全免费的性能提升，而且安装过程非常简单，值得一试。

## 安装步骤 (全程约 3 分钟)

该指南适用于所有主流的 64 位 Debian、Ubuntu 及其衍生发行版。

**0. 安装 gnupg 软件包**

```bash
sudo apt update
sudo apt install gnupg
```

**1. 添加 GPG 密钥**

```bash
wget -qO - https://dl.xanmod.org/archive.key | sudo gpg --dearmor -vo /etc/apt/keyrings/xanmod-archive-keyring.gpg
```

**2. 添加软件源**

```bash
echo "deb [signed-by=/etc/apt/keyrings/xanmod-archive-keyring.gpg] http://deb.xanmod.org $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/xanmod-release.list
```

**3. 更新并安装内核**

```bash
sudo apt update && sudo apt install linux-xanmod-x64v3
```

*注：`x64v3` 适配绝大多数云服务器的 CPU 架构 (2015 年后)，是通用首选。*

**4. 重启系统**

```bash
sudo reboot
```

## 安装校验

重启后，通过命令确认内核和 BBRv3 是否都已生效。

**检查内核版本**

```bash
uname -r
```

预期输出应包含 **`xanmod`** 字样，例如：

```
6.17.0-xanmod1-x64v3
```
