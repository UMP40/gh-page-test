---
layout: post
title: 用 Tiny Tiny RSS 搭建自己的 RSS 服务端
ptype: post
mathjax: false
noindex: false
date: 2020-01-20 15:44:04
categories:
 - 笔记本
tags:
 - RSS
 - Tiny Tiny RSS
 - 自建
keywords: rss, tiny tiny rss, 使用, 教程, 搭建, 高效, 快速, 自动化, feedly, inoreader, 订阅, 多元, vps, docker, 宝塔, 自动更新, 定时, 过滤器, filter
description: 'RSS 曾是互联网上最普遍的信息获取方式。但是，随着各类智能推荐的信息流平台的诞生，2020 的 RSS 似乎早就度过了其黄金时代。然而，现在的互联网世界越来越多元化，质量也参差不齐。而利用自己的订阅源，避开繁杂的互联网漩涡，高效地浏览所需的信息，似乎更显价值。'
purl: use-tt-rss-to-build-your-own-rss-service
thumbnail: https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200130100413.png
---





RSS 曾是互联网上最普遍的信息获取方式。但是，随着各类智能推荐的信息流平台的诞生，2020 的 RSS 似乎早就度过了其黄金时代。

然而，现在的互联网世界越来越多元化，质量也参差不齐。而利用自己的订阅源，避开繁杂的互联网漩涡，高效地浏览所需的信息，似乎更显价值。

<!--more-->

## 为何选择自建 RSS

RSS 就像互联网世界中的「通知栏」。一个好的 RSS 服务端才能更好地发挥 RSS 特长。

而笔者认为，一个好的 RSS 服务，应该满足：

1. 排版简洁，可读性高
2. 订阅方便，支持批量操作 (导入／导出)
3. 可自定义过滤规则

放眼目前主流的 RSS 服务，都或多或少的有那么些不足。比如插入广告、不支持过滤规则等。但其实也能理解，毕竟这些服务器还是一笔不小的开支，而过滤规则又需要额外的算力。所以总需要广告／付费来维持。

但是，仅仅为了这点需求，动辄上百元的花费感觉有点不值。再看现在的 VPS ，许多商家都有自己的轻量套餐，一些时期还会推出各种优惠。比如笔者的 VPS 就是在 2020 跨年双旦优惠的时候入手了一台国外主机，13.99 美金一年。

相比之下，笔者之前使用的 Inoreader 高级版就需要 30 美金一年才能解锁过滤器和去广告。这样一来，选择购买 VPS 自建的性价比就凸显了。更何况一台 VPS 能做的还远不止搭建 RSS 这一点。而且自建让数据完全掌握在自己手里，更可控且更安全。

## Tiny Tiny RSS

目前已经有许多开源的自建 RSS 解决方案。笔者选择的是 [Tiny Tiny RSS](https://tt-rss.org/) ，使用广泛功能完善，而且也比较好的满足了笔者的需求。

至于如何安装，主流的有 Docker 镜像安装。而本文介绍一种可视化更好的借助宝塔面板的安装方式。

## 安装

### 0. 准备环境

- **Docker CE**：后面配置全文爬取的时候会用到

  0. 使用 SSH 连接 VPS

  1. 安装所需软件包

     ```
     sudo yum install -y yum-utils \ device-mapper-persistent-data \ lvm2
     ```

  2. 建立稳定库

     ```
     sudo yum-config-manager \ --add-repo \ https://download.docker.com/linux/centos/docker-ce.repo
     ```

  3. 安装*最新*社区即容器化

     ```
     sudo yum install docker-ce docker-ce-cli containerd.io
     ```

     中途默认 `y` 通过。如果最后有 `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35` 即成功

  4. 启动服务并设置开机自启

     ```
     sudo systemctl start docker
     sudo systemctl enable docker
     ```

- **MySQL 数据库**：用于存放 RSS 服务的数据

  软件管理 -> 运行环境

  安装 MySQL 和 PHP (v5.6 以上)

  P.S. 建议初次运行宝塔面板的时候直接一键安装 LNMP

- **安装 fileinfo 兼容插件**：由于 MySQL 与高版本 PHP 有兼容性问题，所以需要在 软件管理 -> PHP -> 设置 -> 安装扩展 处安装 fileinfo

至此环境部分全部准备完毕。

### 1. 新建一个网站

点击 网站 -> 添加站点

输入一个指向主机 IP 的域名 (且没被其他网站占用)，在数据库处选择创建 MySQL 数据库，并且选择你安装的 PHP 版本。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120122027.jpg)

确认无误后点击提交。

### 2. 上传 tt-rss 的包

前往 [官方 Git 地址](https://git.tt-rss.org/git/tt-rss/src/master) 下载最新镜像。

定位到刚刚创建网站的根目录，一般是 `/www/wwwroot/domain` ，删除原来的文件，上传镜像、解压，然后将解压出来的文件夹里面的内容全选剪贴到网站根目录，再把压缩文件和文件夹删除。

最后根目录大概是这个模样：

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120122121.jpg)

### 3. 给予网站根目录适当权限

如果使用默认权限，在后面安装的时候可能会报错失败。我们提前先给好所需的权限。

0. 使用 SSH 连接 VPS

1. 定位到网站根目录：

   ```
   cd /www/wwwroot/damain
   ```

2. 给予权限：

   ```
   chown -R www:www *
   chmod -R 755 *
   ```

这样下来后期就几乎不会再出错。

### 4. 打开网址，安装

初次打开网址，会跳转安装页面。

1. **Datebase type**：数据库类型，选择 **MySQL**
2. **Username**：用户名，使用数据库的用户名，在点击右侧数据库选项中可以查看
3. **Password**：数据库密码，在右侧数据库选项中可以查看，也可点击后方按钮直接复制
4. **Host name**：直接 `localhost` 即可
5. **Port**：端口，填默认的 `3306` 

然后点击下方测试配置 (Test configuration) ，检查是否正确。

之后会生成 `config.php` 文件，点击 Save configuration 保存。

然后就可以键入 domain 打开网址，默认账户：admin，默认密码：password

## 初次使用

### 更改账户／密码

登录后，请立刻更改密码。

右上角 -> 偏好设置 (Preferences) -> 用户 (User)

点击 admin 便可修改密码，修改后需要重新登录。

建议平时使用新添一个 User 账户，但给予的是**普通用户**权限。

### Preferences 基本配置

在 **Preferences 偏好设置** 里，还有许多配置需要注意。

这里可以设置语言、时区 (默认都是智能选取) ，以及主题，是否允许 API 登陆此账号 (使用第三方客户端时需要打开) 。

由于 VPS 的空间有限，所以建议清除一定时期以前的旧文章。至于是否清除未读文章，看大家的使用习惯。

Tiny Tiny RSS 默认两栏样式，点击标题以后在下方展开内容。关闭 **合并模式** 后变为三栏式布局。

### Feeds 订阅源设置

在 **Feeds** 可以管理订阅 (添加／删除／修改／分类) ；在 **OPML** 对订阅批量批量操作 (导入／导出) 。

如果以前使用其他 RSS 服务，可以在原服务端处批量导出称 OPML 或者 XML，然后进入 Feeds -> OPML 处相应导入即可无痛迁移。

你甚至可以对单独订阅源设置名称、更新频率，使用的插件等。

### Filter 过滤器

过滤器可能是我转移到自建 RSS 的一个重要原因，有了这个帮助就可以更高效地处理大量内容。

Tiny Tiny RSS 的过滤器分为 **匹配** 和 **操作** 两个环节。如果满足 “匹配” ，就执行 “操作” 。点击 **Create filter 创建过滤器** 创建新的过滤规则，**Caption 标题** 为规则的名称，**Match 匹配** 为匹配规则，而 **Apply actions 应用操作** 则是对被匹配的文章执行的操作。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120140133.jpg)

对于每个过滤规则支持多个匹配规则，点击 **Add 添加** 添加新规则，可以设置匹配的对象和指定订阅源，支持正则表达式。多个匹配规则默认是「与」的关系，如果要改成「或」的话就打开 **Match any rule 匹配任意规则** 。

而对于被筛选出来的文章也有多种操作，一个特殊的操作是 **Publish article** ，会被放入 **Published articles 已发布的文章** 区域。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120135831.jpg)

## 高级配置

###订阅更新

Tiny Tiny RSS 支持两种更新方式：**简单更新**和**定时更新**。

#### 简单更新

简单更新模式，指每次打开的时候就立即更新。

可以用宝塔面板或者 vi 终端打开网站根目录下的 `config.php` ，然后定位到 `SIMPLE_UPDATE_MODE` 并将其改为 `true` ，即可。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120140741.jpg)

#### 定时更新

#### 使用 cron 实现：

由于 Tiny Tiny RSS 无法 root 运行，我们不能使用宝塔计划任务。这里要手动添加定时任务。

```
# 设置每 240 分钟 (4 个小时) 更新一次
echo "*/240 * * * * www /www/server/php/56/bin/php /www/wwwroot/domain/update.php --feeds --quiet" >> /var/spool/cron/www

# 赋予 www 文件权限
chmod 600 /var/spool/cron/www

# 重新加载／启动
service crond reload
service crond restart
```

可以通过一下命令检查是否运行：

```
tail -f /var/log/cron
```

如果报错，就检查一下目录是否正确。

P.S. (2020-1-27 更新) 博主用上面的方法不是很成功，换了以下方法：

```
# 在 root 账户下
echo "*/240 * * * * su -m www -c \"/www/server/php/56/bin/php /www/wwwroot/domain/update.php --feeds --quiet\"" >> /var/spool/cron/root

# 剩下的赋予权限及重载与上面的类似
chmod 600 /var/spool/cron/root
service crond reload
service crond restart
```

#### 使用 daemon 配合  systemd 更新 (官方推荐) ：

在 `/etc/systemd/system/` 创建一个 `ttrss.service` ，输入：

```
[Unit]
Description=ttrss_backend
After=network.target mysql.service postgresql.service

[Service]
User=www
ExecStart=/www/wwwroot/domain/update_daemon2.php

[Install]
WantedBy=multi-user.target
```

注意将路径改为自己的。

然后使用命令启动：

```
systemctl daemon-reload
systemctl enable ttrss
systemctl start ttrss
```

查看服务状态：

```
systemctl status ttrss
```

### 全文爬取

其实 RSS 不受待见的一个原因是它会分走站点的部分流量。所以有些网站放不开手，要么不提供 RSS 订阅，要么订阅后只能看前几行，想看全文还是要打开网页查看。但这样的跳转是很影响体验的，而 Tiny Tiny RSS 也有全文爬取的插件。

#### 安装 mercury_fulltext 插件

0. 定位到插件目录：

   ```
   cd /www/wwwroot/domain/plugins.local
   ```

1. 安装插件 (Git) ，当然也可以手动下载插件然后放到相应位置

   ```
   git clone https://github.com/HenryQW/mercury_fulltext.git
   ```

2. 在 **Preferences 偏好设置** 中启用该插件

#### 搭建 Mercury Parser API

[项目地址](https://github.com/HenryQW/mercury-parser-api) 

我们已经在 #准备环境 中安装了 **Docker CE** ，现在只需要输入：

```
docker run -p 3000:3000 --restart=always -d wangqiru/mercury-parser-api
```

#### 配置 mercury_fulltext

在 Preferences -> Feeds -> Mercury Fulltext settings (mercury_fulltext) 中填入 API 地址 `localhost:3000` 。

笔者是直接在本机搭建，如果在其他服务器上搭建，就用 `host + ip` 的形式。

#### 使用

右键需要打开此插件的订阅源，点击 `编辑信息源` -> `插件` 位置打开 mercury_fulltext 插件。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120121722.jpg)

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120120351.jpg)

然后再右键该订阅源，点击 `调试信息源` ，勾选 `force refetch` ，然后 continue。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120121818.jpg)

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120121122.jpg)

完成后就发现讨厌的 *Read more ...* 没有了！

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120120253.jpg)

### 平台使用

#### 插件：fever

Tiny Tiny RSS 毕竟还是个小众的框架，需要插件使得第三方服务端适配。

目前比较好的选择仍是 [fever](https://github.com/rannen/tinytinyrss-fever-plugin) ，尽管它已经 3 年没有更新，但是大体不影响使用。

1. 上 GitHub 下载，解压源码
2. 将解压得到的文件夹里的 fever 文件夹上传至插件文件夹 (网站根目录/plugins.local)
3. 在 Preferences 偏好设置中会出现一个新的页面设置独立密码并记录下 RSS 链接。

#### 桌面端：浏览器

其实就桌面端而言，浏览器的体验已经很优秀了。如果不满意还可以使用 **Theme 主题** ，比如笔者比较喜欢的 [Feedly Theme](https://github.com/levito/tt-rss-feedly-theme) 。可以参照 `Readme.md` 的使用方法，也可以下载到电脑，解压，然后利用宝塔面板上传到指定位置 (网站根目录/theme.local) ，只用上传 CSS 文件。一个主题有时包含许多个 CSS ，是主题的不同模式 (如暗色模式等) ，大家选择性上传。

然后在 **Preferences 偏好设置** -> **Theme 主题** 中选取一个要使用的 CSS，保存即可。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120121433.jpg)

####Android: Feedme

安卓上比较推荐一款应用：Feedme，一款 Google Play 上的应用，直接支持导入Tiny Tiny RSS。初次打开的时候可以选择 **Tiny Tiny RSS 登录** ，输入网址、用户名、登录密码即可使用。但是需要在偏好设置中打开 **启用 API** 。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200120121624.jpg)

#### iOS: Reeder / iPadOS: Mr.Reader

iOS／iPadOS 上的应用，采用 fever 独立 RSS 链接和密码使用。

## 结尾

到这里，笔者已经完全交代了直至我自己满意的配置了，并且在融合的过程中，它已经可以很好地替代了之前使用的 Inoreader。

确实，熟练的话半个小时也许就能使用。我却折腾了 2、3 个小时才从各种坑中爬出来。

那么问题来了——值得吗？

在抖音、头条这些 App 兴起的衬映下，RSS 似乎就显得十分「老套」了。可是这些快资讯平台带来的浅阅读质量实在无法拿捏，而且久而久之还可能丧失独立深度思考的能力。

尽管我对这些 App 保持了距离，但是各类新媒体还是源源不断地冒出来。我空余刷社交媒体、他人博客、各种资讯网站的时间，从半个小时、一个小时到了不久前接近 4 个小时每天。

其实每个人一生能够掌握的信息是极其有限的，也许就几十个 G 。整个 2019 年互联网平均每天产生的数据都有 491EB 这么多，在这种巨大的悬殊下怎么甘心任时间在各种廉价资讯中消磨殆尽。

这回，我把部分高质量感兴趣的网站、博客放进 RSS ，把一些重要的 Telegram 通知频道／Twitter 通过工具也放进 RSS ，设置为 4 个小时更新一次。每天在通勤、午休、晚上定期查阅。也许我获取到的有效信息数量与之前是差不多的，可是现在据数字健康统计，每天花费在这上面的时间压缩至 1.5 个小时左右。这也许能让我有精力去做一些更*大块*的事情，更有意义的事情……

所以，我的答案是，肯定的。

