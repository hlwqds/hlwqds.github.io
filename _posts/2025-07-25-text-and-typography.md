---
title: 从内核出发
date: 2024-07-25 00:11:00
pin: true
categories: [kernel]
tag: [kernel]
---

# 编译内核

## 内核代码下载

```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux-2.6.git
```

## 配置内核

```bash
make config
# 使用这个命令会逐一遍历所有配置项，要求用户选择

make menuconfig
# 使用这个命令可以让你基于图形界面选择

make defconfig
# 这个命令会基于默认的配置为你的体系结构创建一个配置

# 命令完成后这些配置项会存放在内核代码根目录下的.config文件中。你可以直接修改它。修改过配置文件之后，使用如下命令
make oldconfig

# 当使用CONFIG_IKCONFIG_PROC后，你可以在运行时看到内核配置文件被存放在/proc/config.gz下，复制这个文件后你可以方便的克隆出一个相同配置的内核。
zcat /prco/config.gz > .config
make oldconfig

# 一旦内核配置好了，就可以使用一个简单的命令来编译它
make
```

## 同步和并发

- linux是抢占多任务操作系统。内核的进程调度程序即兴对进程进行调度和重新调度。内核必须和这些任务同步

- linux你和支持对称多处理系统(SMP)。所以，如果没有适当的保护。同时在两个或多个处理器上执行的内核代码很可能会同时访问共享的同一个资源。

- 中断是异步到来的，完全不顾及当前正在执行的代码。也就是说，如果不加以适当的保护，中断完全有可能在代码访问资源的时候到来，这样，中断处理程序就有可能访问同一资源

- linux内核可以被抢占。所以，如果不加以适当的保护，内核中一段正在执行的代码可能会被另外一段代码抢占，从而可能导致几段代码同时访问相同的资源。

常用的解决竞争的方法是自旋锁和信号量。
