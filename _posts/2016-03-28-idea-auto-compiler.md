---
layout: post
title: IntelliJ IDEA 学习笔记-开启自动编译
categories: IDEA
---

# 开启自动编译

- 快捷键 `Command + ,` 进去设置 Perferences:Build,Execution,Deployment -> Compiler 勾选右侧的 Build project automatically。

![IDEA Compiler](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/idea/idea-compiler.png)

- 快捷键 `Command + Shift + A` 调出搜索命令:registry -> 找到 `compiler.automake.allow.when.app.running` 并勾选.

![IDEA registry](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/idea/idea-registry.png)

![IDEA APP Running](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/idea/idea-compile-when-app-running.png)
