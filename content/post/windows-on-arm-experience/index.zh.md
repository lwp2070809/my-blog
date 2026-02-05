---
title: Windows on ARM 体验报告
date: 2026-02-05T14:00:00+08:00
lastmod: 2026-02-05T14:00:00+08:00
draft: false
slug: windows-on-arm-experience
categories: []
tags: ["科技", "硬件", "Windows on ARM", "WoA"]
description: 
---

2023年6月我购买了`华为 MateBook E Go 2023`(标准版, 8cx gen3, 16+256), 作为我的上一台二合一电脑`Surface Pro 7`(i5, 8+128)的替代. 这台电脑作为主力设备在我前公司使用了一年, 主要进行Java开发和服务器运维, 并在现公司作为移动办公的设备使用. 下面是近三年来的使用体验. 这篇文章即撰写与此设备. 将使用`WoA`表示`Windows on ARM`.

## 硬件和OEM

这部分和`WoA`的体验关系不是特别大, 因此不会占用太多篇幅. 

考虑到只有平替款`Surface Pro X`一半价格的高性价比, 我确实不应该对硬件要求太高, 但的确存在一些比较影响使用体验的部分. 首先赠送的键盘和磁吸后保护套体验不算好, 磁吸后保护套非常的松垮, 很容易脱落, 键盘没有背光, 触摸板频频失灵几乎不可用, 唯一的优点是似乎走了2.4G协议和电脑连接, 可以脱离电脑使用. 当前硬件方面还是存在优点的, 比如在这个价位比较少见的高质量屏幕.

OEM软件和驱动部分是相当糟糕的. 除了高通提供的adreno图形驱动外, 其他驱动出场即停更, 甚至一年后的`Windows 24h2`都存在不兼容现象. 很多驱动都是x64架构的而非原生ARM64, 触摸驱动也运行在用户态, 性能和体验较差, 实际上据我所知2022款用户的触摸体验更糟糕, 不得不从2023款的驱动解包触摸驱动更新才能勉强可以使用. 配套的OEM软件同样是x64, 并且缺乏像dell那样完善的电源管理功能 (我唯一会使用的OEM功能), 使得我到手就使用干净的Windows镜像重新安装了系统.

## 性能和续航

非常满意的一点. 这颗低频版`Snapdragon 8cx Gen 3 @ 2.69 GHz`处理器的能耗表现在同期处理器中非常的厉害, 并且离电性能也可圈可点. 在Windows 11, 省电模式, 能源之星启用的情况下, 以仅仅40Wh的电池, 可以达到中度办公续航8小时的体验, 并且不会感觉到明显卡顿. 无论是此前`Surface Pro 7`的`i5-1035G4`, 还是后来新公司给我的thinkbook笔记本电脑搭载的`i5-1035G7`都无法达到这样的效果. 以thinkbook笔记本电脑为例, 搭载了57Wh的电池在同样的条件下却仅仅只有4小时的续航, 简直难以置信.

## 开发体验

`Visual Studio Code`的`Remote Development`直接杀死比赛, 借助`Dev Container`, 我直接在远程服务器安装用于开发的docker容器, 并将本机的VSCode连接至容器进行开发. 无需考虑开发套件是否提供Windows的ARM64版本, 也不会占用本机内存. 我花了一点时间从使用IDEA的Spring Boot开发迁移到VSCode, 并且总结了一套最佳实践. 当然, 后来流行的`Github Copilot`等AI结对编程方式使得越来越多的Spring Boot开发者也迁移到VSCode, 则是后话了. 现在我是全栈开发, 无论是Golang, Vue还是Spring Boot, 我都能顺利的在这台电脑上进行开发工作, 并和位于公司的台式电脑共享工作进度.

在新公司我还承接了一部分.net开发工作, 需要到客户现场安装软件和调试, 因此我在这台电脑上安装了`Visual Studio`, 交叉编译x64版本的WPF软件同样没有问题.

## 办公体验

我列出我使用的办公软件, 这些软件提供了在Windows上原生ARM64的体验, 未列出的软件不代表没有WoA版本, 可能只是我没有使用.

* SSH工具: Tabby

* 文本编辑器: Notepads.app

* VPN: OpenVPN, WireGuard

* 代理: v2rayN

* IDE: VSCode

* 解压缩: 7zip

* 视频播放器: Screenbox

* 浏览器: Edge Firefox

* 办公: Office 全家桶

* 其他工具: PowerToys 微软键盘和鼠标中心 能源之星x 微软待办

* 其他开发工具: git AnotherRedisDesktopManager 

---

下面是一些只有x64版本的软件, 但是在WoA可以以兼容模式正常使用.

* 数据库GUI: HeidiSQL, DBeaver

* 电子书管理: calibre

* 远程桌面: RustDesk

* 其他工具: LocalSend

---

微信的特别使用方式:

我使用的是一个存在于微软应用商店的legacy版本的微信, 版本号为2.6.3.0, 这是一个几乎只能使用基础功能, 并且部分功能存在bug的版本, 但是不到50MB的内存占用, 非常轻量的CPU占用, 似乎并不存在的流氓行为, 让我在这台二合一电脑上依然坚持使用这个版本.

这个版本已经下架, 似乎也不能入库了, 但是使用之前入库的微软账号登录商店还是可以正常下载. [链接](https://apps.microsoft.com/detail/9NBLGGH4SLX7?hl=zh-cn&gl=US&ocid=pdpshare)

还需要手动修复因为不兼容新版Windows系统的bug:

```powershell
mklink /d "C:\Users\lwp20\AppData\Roaming\Tencent\WeChatAppStore" "C:\Users\lwp20\AppData\Local\Packages\TencentWeChatLimited.forWindows10_sdtnhv12zgd7a\LocalCache\Roaming\Tencent\WeChatAppStore"
```

## 娱乐

* Telegram: 在Windows上提供了ARM64版本, 但似乎存在兼容性问题, 启动时间长且每次启动都会出现弹窗.

* 聚合直播: 在WoA上体验非常好, 原生ARM64, 非常轻巧和高性能.

* 弹弹Play: 只有x64版本, 在播放一些高码率视频时, 似乎会出现掉帧, 可以使用弹弹Play提供的远程服务器功能, 在本机使用Edge浏览器播放, 性能表现会好很多.

---

关于游戏

我没有测试过steam游戏, 根据网上看到的测评, 是可以运行很多游戏的. 由于我是Xbox用户, 并且我不想在这台电脑有限的存储空间中安装太多软件, 因此只选择Microsoft Store和Xbox PC的游戏. 在2025年8月之前, 只能在Microsoft Store安装游戏.

## 一些踩坑