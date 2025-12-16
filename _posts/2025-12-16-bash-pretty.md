---
title: bash美化
date: 2025-12-16 00:11:00
pin: true
categories: [bash]
tag: [bash, pretty]
description: DPDK 与 内存管理知识点汇总
---

# bash美化

## starship

```bash
# Enable Starship prompt
if command -v starship > /dev/null; then
    eval "$(starship init bash)"
fi
```

### 效果

![image-20251216101137075](https://cdn.jsdelivr.net/gh/hlwqds/hlwqds.github.io@main/storage/image-20251216101137075.png)

### 注意事项

会和oh-my-bash冲突，需要将对应的主题关闭

```bash
# Set name of the theme to load. Optionally, if you set this to "r    andom"
# it'll load a random theme each time that oh-my-bash is loaded.
# OSH_THEME="font"
```

## eza

```bash
# Enable Eza aliases
if command -v eza > /dev/null; then
    alias ls='eza'
    alias ll='eza -l --icons'
    alias la='eza -la --icons'
    alias lt='eza --tree --icons'
fi
```
