---
layout: post
title: GitHub + PicGo 免费图床搭建及使用教程
ptype: post
keyword: 图床, picgo, GitHub, 免费, 教程, jsdelivr, CDN, 加速, 使用, 配置, 图, host
mathjax: false
date: 2019-10-20 15:41:38
categories:
 - 笔记本
tags:
 - 图床
 - Free
description: '图床是每个博主不可或缺的工具，通过它你才能更好地将图片展示在互联网的任何地方。如果你还在纠结图床服务，不妨试试 GitHub + PicGo 搭建自己的个人免费图床，稳定、简单、高效！配合 jsDelivr 免费 CDN 加速甚至还可获取意想不到的速度体验，堪比一般付费图床。'
purl: free-img-hosting-with-github-and-picgo
---



![](https://cdn.jsdelivr.net/gh/ChrAlpha/imgbag/20191020154650.png)

[PicGo 官方 GitHub 地址](https://github.com/Molunerfinn/PicGo) , [PicGo 中文文档](https://picgo.github.io/PicGo-Doc/zh/) 

## 主理人说

图床，可以理解为让图片「躺好」在互联网上的工具，方便图片在网络上被调用。

如果你也想让图片辅佐内容输出，那么一个得心应手的图床是必不可少的。

如果我们对比一下各路图床：

> - 微博图床添加防盗链，凉凉
> - SM.MS 运营许久，但是使用的人过多，API 不稳定（曾经被滥用），转付费模式后免费版高峰期访问速度更加堪忧。
> - Imgur 国外著名免费图床，但国内访问速度实在不尽人意，且有被墙的风险。
> - 小众图床，不是我不相信「小而美」，但是万一跑路了真的很伤。

尝试过的朋友可能知道，图床选定使用过一段时间，想要转移是非常痛苦的。所以早期选好一个稳定、方便的图床确实很有必要。

那究竟有没有免费且效果还不错的图床选择呢？

**当然有！**折腾过一番图床，我最后还是选择了 GitHub + PicGo 的方式。

优点：

- 全免费！
- 没有大小限制，也不会对图片自行压缩。
- 后台稳定，~~一般~~不会有**莫名其妙**的连不上。

如果配合 jsDelivr 免费 CDN 加速缓存，更能收获意想不到的速度体验。

话不多说，赶紧看看如何用上。

## 使用

### 基本配置

首先我们要有一个 GitHub 仓库来放置这些图片。你需要一个 GitHub 账号，常规注册就好。然后新建一个仓库，名字任意，但一定要是**公开**仓库。

![](https://cdn.jsdelivr.net/gh/ChrAlpha/imgbag/20191020153917.png)

然后在 Github -> Settings -> [personal accese token](https://github.com/settings/tokens) 这里新建一个 token ，打开 repo 权限，然后将显示的 token 复制出来（最好存下来，这个 token 只会出现一次）。

![](https://cdn.jsdelivr.net/gh/ChrAlpha/imgbag/20191020153242.png)

之后在 图床设置 -> GitHub 设置 中把 token 导入进去。

![](https://cdn.jsdelivr.net/gh/ChrAlpha/imgbag/20191020153631.png)

至此已经可以导入了！

可选：将 GitHub 设为默认图床，这样以后上床的时候就不用再选择。

### 导入方法

1. 在导入区，把图片拖进去，或者选择文件

   ![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/picgo-2.0.gif)

2. 右键图片选择 `使用 PicGo 导入`

   ![](https://cdn.jsdelivr.net/gh/ChrAlpha/imgbag/20191020161140.png)

3. 如果你使用的是 Mac ，那么运行后在上方的控制条中会有个图标。

   ![](https://user-images.githubusercontent.com/12621342/34242310-b5056510-e655-11e7-8568-60ffd4f71910.gif)

   直接将图片拖到那个位置就上传了

4. 如果你使用的是 windows ，可以开启一个悬浮窗，将图片拖到悬浮窗即可。

### PicGo 配置

点击 `PicGo 配置` ，可以看到一些关于软件的设置，比如可以选择只将你常用的图床放在侧边栏，或者使用时间戳重命名等（可有效防止重名）。

![](https://cdn.jsdelivr.net/gh/ChrAlpha/imgbag/20191020161156.png)

- 关联阅读：[ChrAlpha - jsDelivr 免费加速静态资源](https://chralpha.com/post/use-jsdelivr-speed-up-static-files-visits/) 

## 后

PicGo 支持了 Mac, Windows, Linux，是一款开源免费的 electron-app，如果大家觉得好用，可以上 GitHub 去给他们点 star 。
