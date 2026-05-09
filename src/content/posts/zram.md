---
title: "为VPS配置Zram：零成本让你的服务器内存翻倍"
published: 2025-11-19
tags: ["技术教程", "随笔"]
category: "技术教程"
---

本教程适用于 **Debian** 和 **Ubuntu** 系统。

## 1. 安装 zram-tools

首先，更新软件包列表并安装 `zram-tools`：

```bash
sudo apt update
sudo apt install zram-tools
```

## 2. 一键写入推荐配置

下面的命令将写入一个通用且高效的配置。它使用 `zstd` 作为压缩算法，并将 Zram 的大小设置为与物理内存相同（1:1）。

> **Tips：** 在使用这个推荐配置前，请先确保你的内核支持zstd压缩算法。

```bash
cat /sys/block/zram0/comp_algorithm
#列出内核支持的压缩算法
```

如果不支持zstd，请自行在配置文件中换用其他支持的算法。如果你追求zstd的极致能效比，可以考虑安装Xanmod内核补全zstd特性。

```bash
sudo tee /etc/default/zramswap > /dev/null <<'EOF'
# 压缩算法选择
# 速度: lz4 > zstd > lzo
# 压缩率: zstd > lzo > lz4
# 此列表并未包含最新内核中所有可用的算法
# 请查看 /sys/block/zram0/comp_algorithm (当 zram 模块加载后) 以了解您内核当前设置及可用的算法[1]
# [1]  https://github.com/torvalds/linux/blob/master/Documentation/blockdev/zram.txt#L86
ALGO=zstd

# 指定 zram 应使用的内存量
# 基于可用总内存的百分比
# 此设置将优先于并覆盖下方的 SIZE 设置
PERCENT=100

# 指定 ZRAM 设备应使用的静态内存大小，单位为 MiB
# SIZE=256

# 指定交换设备的优先级，更多详情请参阅 swapon(2) 手册
# 数值越高 = 优先级越高
# 此优先级应高于普通硬盘(hdd)/固态硬盘(ssd)上的交换分区
# PRIORITY=100
EOF
```

## 3. 调整内核使用 Swap 的积极度

我们需要编辑 `/etc/sysctl.conf` 文件来调整 `vm.swappiness` 参数。此参数值越高，内核使用 Swap 的积极性就越高。

此处使用 `nano` 编辑器作为示例：

```bash
sudo nano /etc/sysctl.conf
```

在文件中找到 `vm.swappiness = *` 这一行，如果文件中没有这一行，请在末尾自行添加。

对于 Zram 来说，一个较为普适的值是 **80**。

如果你使用第三方内核如上文提到的XanMod，那么你可以尝试调整到160或更激进的值，不过这是否有效有待斟酌。

无论怎样，设置为80是不会错的选择。

例如，添加或修改为： `vm.swappiness = 80`

保存并关闭文件后，执行 `sudo sysctl -p` 使其立即生效。

## 4. 重启并校验成果

为了确保所有配置都已应用，请重启你的 VPS：

```bash
sudo reboot
```

重启后，运行以下命令来检查 Zram 状态：

```bash
zramctl
```

如果配置成功，输出应该类似这样：

```
NAME       ALGORITHM DISKSIZE DATA COMPR TOTAL STREAMS MOUNTPOINT
/dev/zram0 zstd          1.9G   4K   59B    4K       2 [SWAP]
```

**检查输出的关键信息：**

- **ALGORITHM**: 确认是否为你配置的压缩算法（如 `zstd`）。
- **DISKSIZE**: 确认 Zram 的总大小是否符合预期。

如果这两项与你的配置文件不符，很可能是内核不支持 `zstd` 算法，或者配置文件格式错误，导致 `zram-tools` 使用了默认配置。

值得一提的是，你可以定期检查 `DATA`（存入 Zram 的原始数据大小）和 `COMPR`（压缩后实际占用的内存大小）的变化，直观地感受 Zram 带来的内存提升！

希望这能帮到你！
