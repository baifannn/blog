---
title: 'Mac技巧01-brew提高速度'
date: 2020-03-20 15:03:19
tags:
- mac
- osx
categories:
- mac技巧
---

这里用中科大的，另外还有清华的可用

### 切换
```shell
# 步骤一
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

# 步骤二
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

#步骤三
brew update
```

注意这里需要等待一会，因为要更新资源。

更新完后使用brew update，brew install速度变快很多了，不会卡在那半天没动静，替换镜像完成。

### 复原
```shell
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core

brew update
```