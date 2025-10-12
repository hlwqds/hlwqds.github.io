---
title: 虚拟机v2ray配置
date: 2024-10-12 00:11:00
pin: true
categories: [tools]
tag: [virtual, tools]
---

# 虚拟机v2ray配置

## 方法

查看v2ray中局域网http端口
![alt text](../image.png)

查看虚拟网卡的地址

```powershell
ipconfig
   本地链接 IPv6 地址. . . . . . . . : fe80::76c5:e5f3:a00b:65d%18
   IPv4 地址 . . . . . . . . . . . . : 192.168.124.1
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
```

在虚拟机中配置代理

```bash
echo 'export http_proxy=http://192.168.124.1:10811
export https_proxy=http://192.168.124.1:10811' > /etc/profile.d/proxy
source /etc/profile.d/proxy
```

取消代理

```bash
echo 'export http_proxy=
export https_proxy=' > /etc/profile.d/proxy.disable
source /etc/profile.d/proxy.disable
```
