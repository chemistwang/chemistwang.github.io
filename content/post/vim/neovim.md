---
# layout: post
title: "打造neovim"
subtitle: ""
date: 2023-05-17
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "念念不忘vim"
---

## 环境

- M1 macOS 13.0

## 1. [安装 neovim](https://github.com/neovim/neovim/wiki/Installing-Neovim)

```bash
brew install neovim
```

可以在 `~/.bash_profile` 设置下环境变量方便命令输入

```bash
...
alias vim=nvim
alias vi=nvim
...
```

## 2. [安装字体](https://www.nerdfonts.com/font-downloads)

```bash
brew tap homebrew/cask-fonts && brew install --cask font-jetbrains-mono-nerd-font
brew tap homebrew/cask-fonts && brew install --cask font-symbols-nerd-font
```

## 3. 设置字体

从 `iTerm2` 打开，会出现问号之类的乱码，需要设置字体

![设置字体](/post/vim/images/setfont.jpg)

## 4. [安装 nvchad](https://nvchad.com/docs/quickstart/install)

```bash
# 安装
git clone https://github.com/NvChad/NvChad ~/.config/nvim --depth 1 && nvim
# 卸载
# Linux / Macos (unix)
rm -rf ~/.config/nvim
rm -rf ~/.local/share/nvim
```
