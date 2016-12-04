---
layout: post
title: "当Selenium 遇上Docker， CI 不在无聊"
date: 2016-11-19 06:15:20
categories: Selenium Docker
---

# 问题
使用Selenium 框架来进行自动化测试需要一个浏览器，比如Firefox 或者Chrome。这也意味着需要一个GUI 界面，而CI 环境里的操作系统往往没有图形化界面。
如何解决这个问题呢？ 以前我们可能会为虚拟机安装一个GUI，然后通过VNC 来访问。但这需要改变已有的CI 环境。

# Docker的出现
访问Docker 的首页，首先映入你眼帘的是DockerStyle 模式。

>Build, Ship, Run

使用Docker 你可以保证本地和CI 的环境一致，大大减低集成难度和调试成本。
而且可以很方便的扩展到多台机器。
