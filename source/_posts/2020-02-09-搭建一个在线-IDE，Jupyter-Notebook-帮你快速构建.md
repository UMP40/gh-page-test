---
layout: post
title: 搭建一个在线 IDE，Jupyter Notebook 帮你快速构建
ptype: post
mathjax: false
noindex: false
date: 2020-02-09 16:46:17
categories:
 - 笔记本
tags:
 - Web
 - Jupyter Notebook
 - IDE
 - Nginx
keywords: jupyter, jupyter notebook, IDE, python, 搭建, 自建, 教程
description: '过年探亲，总会是一件很惬意的事情。但是碰巧不巧，这年来了点状况，各路返工返校全部延后。看着呆在家里快成一条死鱼的自己，无奈只有一台手机 / iPad，完全与生产力搭不上边。不急，如果你折腾了我即将介绍的东西，那只要有一个浏览器，就能在任何地方编写、运行代码；如果没有，那你也可以把自己关在家折腾一天，把所有坑全部踩一遍，这样减少人口流动，也算是对抗疫做些贡献。'
purl: build-an-online-ide-by-jupyter-notebook
---

> 本文同步载于 [少数派](https://sspai.com/post/58789) ，部分 markdown 语法在该页面渲染失败。

其实本文和以前 [那篇文章](/post/build-an-online-ide-by-jupyter-notebook-archived/) 操作十分类似，这次只是稍微解释的详细一点~~（废话多一点）~~。

以下是原文。

<!--more-->

---

过年探亲，总会是一件很惬意的事情。但是碰巧不巧，这年来了点状况，各路返工返校全部延后。看着呆在家里快成一条死鱼的自己，无奈只有一台手机 / iPad，完全与生产力搭不上边。不急，如果你折腾了我即将介绍的东西，那只要有一个浏览器，就能在任何地方编写、运行代码；如果没有，那你也可以把自己关在家折腾一天，把所有坑全部踩一遍，这样减少人口流动，也算是对抗疫做些贡献。



## Jupyter Notebook 介绍

回到正题。

什么是 IDE？IDE 全称是**集成开发环境** (Integrated Development Environment) ，是一种辅助开发人员的工具。在该工具内可以编辑源代码文件、并编译打包成可执行的程序。一个 IDE 通常包括：编程语言编辑器、自动构建工具、调试器，有的还包含编译器 / 解释器 [^1]。

所以，目标人群很清晰。如果你完全对编程不感兴趣、不想「手玩」几个源代码的话，你可以关闭本页面了。

OK，接下来再来看 Jupyter Notebook 。Jupyter Notebook 的前身是 iPython Notebook ，而在 2014 年 Jupyter 正式从 iPython 中衍生出来，成为一个独立的项目，为数十种编程语言的交互式计算提供开源软件、开放标准和服务。Juypter Notebook 则是一个基于 Web 端的交互式计算环境，用于创建 Jupyter Notebook 文档 (.ipynb) ，由一个个单元格组成，单元格可以包含代码、Markdown 文本、数学、图标和富媒体 [^2]。看得出来，在这样的支持下，我们可以一边编写、运行代码，一边用 Markdown 做注释、记笔记，十分方便。



## Jupyter Notebook 上手

既然是要运行在网页端，一台云服务器肯定是不能缺席的。至于云服务器的购买建议就不在本文的探讨范围内了，大家根据自身条件和需求判断。

到这里我默认大家已经购买好、并通过 SSH 连接上了远程服务器。

由于我正在学习 Python ，就拿这门语言的配置举例。其他语言也遵循 环境 - 工具配置 的主流程，会在文末是稍微提一下。

### 开发环境安装

众所周知，Python 通过模块化特性，避免了许多「造轮子」等行为，提升了编程效率。而实际使用中，难免需要调用各种第三方模块。这时我们可以通过一个 Package 来一次性安装许多常用模块，[Anaconda](https://www.anaconda.com/) 就是这样一个基于 Python 的数据处理和科学计算平台，它已经内置了许多非常有用的第三方库。我们可以选择直接安装 Anaconda 来跳过安装许多模块、依赖的步骤。但是，Anaconda 过于强大的同时避免不了过于庞大，所以我选取的是 [Miniconda](https://conda.io/miniconda.html) ，可以理解为 Anaconda 的精简版，去除了许多也许有用但并不是非常常用的模块，体积也大大缩减。

我们使用官方提供的一键脚本安装：

```bash
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh

bash Miniconda3-latest-Linux-x86_64.sh
```

然后是安装 IPython 和 Jupyter ：

```bash
conda install ipython

conda install jupyter
```

### 工具配置

```bash
jupypter notebook --generate-config
```

执行上述命令来在 `~/.jupyter` 目录下创建一个 `jupyter_notebook_config.py` 的配置文件。

用编辑器打开这个配置文件。太长？没关系，我们只需要关注几个地方，其他使用默认配置即可。

- `c.NotebookApp.port` 

  这个是服务将要运行在的本地端口，填写一个还没有被服务占用的端口。然后 IP 默认是本机 IP (localhost) ，后面也是通过 Nginx 端口映射到本地端口，所以不用管。

- `c.NotebookApp.allow_origin` 

  因为我们需要外网访问，所以请设置为**允许任意** (\*) 即 `c.NotebookApp.allow_origin = '*'` 。

- `c.NotebookApp.open_browser` 

  由于远程服务器大多不安装图形界面，自然也不会有浏览器，这里要设置为 `False` 。这样每次启动服务的时候就不会尝试打开浏览器了，也就不会报错（找不到浏览器）。

- `c.NotebookApp.notebook_dir` 

  这里设置 Notebook 的文件目录，默认是家目录 (~) ，可以设置为一个专门存放代码的目录。

- `c.NotebookApp.allow_root` 

  出于安全考量，Jupyter Notebook 默认禁止了在 root 用户下运行，也不建议这么做。你可以新建一个账户来运行 Jupyter Notebook ，或者将这里设为 `True` ，这样就允许 root 用户运行了。

单独拎出来提一下关于密码。由于这项服务是要暴露在公网的，你或许也不想任何人都可以使用你的计算资源，或者查看、修改你的源码文件，这时可以给 Jupyter Notebook 设置一个密码。

```bash
python -c "import IPython;print(IPython.lib.passwd())"
```

执行上述命令，根据提示输入你想设置的密码。然后将最后生成的以 `sha1` 开头的文本保存下来，这是你的密码经过散列函数加密后得到的内容。

接着在配置文件中找到这两行：

```python
c.NotebookApp.password = u'sha1:xxx' # 刚刚得到的那个加密密码
c.NotebookApp.password_required = True # 需要密码登陆
```

经过一点点配置后，在终端中执行 `jupyter notebook` 就可以启动服务了。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200209161214.PNG)

### Nginx 端口映射

到上一步为止，我们已经可以通过 `{服务器 ip}:{端口号}` 的形式访问了（如果开启了防火墙则需要允许特定端口访问），但使用这种形式访问不如用域名访问那般友好，而且不支持 HTTPS 也会带来安全隐患。

这时我们可以借助 **Nginx 反向代理** 将域名映射到本机制定端口。[Nginx](https://www.nginx.com/) 是一个异步框架的网页服务器，高效、并发能力强。我们先来安装 Nginx ，以 CentOS 为例：

```bash
# 安装
sudo yum install nginx

# 开启
sudo systemctl start nginx

# 查看状态
sudo systemctl status nginx
```

确认开启无误后，我们修改 Nginx 配置文件，让其监听指定域名并重定向到本机 Jupyter Notebook 的服务端口。

先 upstream 声明：

```nginx
upstream jupyter-notebook {
    server 127.0.0.1:{本机 Jupyter Notebook 端口};
}
```

这样后面就可以使用 `http://jupyter-notebook` 统一调用 Jupyter Notebook 服务地址了。

然后是 server 部分，假设我们使用 `jupyter.domain.com` 来访问 Jupyter Notebook 。

```nginx
server {
    listen 80;
	listen 443 ssl http2;
    server_name jupyter.domain.com;
    
    # SSL & https start
    
    if ($server_port !~ 443){
        rewrite ^(/.*)$ https://$host$1 permanent;
    }
    #HTTP_TO_HTTPS_END
    
    ssl_certificate    /path/to/your/ssl_certificate/xxx.pem;
    ssl_certificate_key    /path/to/your/ssl_certificate_key/xxx.key;
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    # SSL & https ends
    
    error_page 497  https://$host$request_uri;
    
    # Nginx 反向代理到本地端口
    location / {
    	proxy_pass http://jupyter-notebook;
        proxy_set_header      Host $host;
    }
    
    #一键申请SSL证书验证目录相关设置
    location ~ \.well-known{
        allow all;
    }
    
    location ~ /api/kernels/ {
        proxy_pass            http://jupyter-notebook;
        proxy_set_header      Host $host;
        # websocket support
        proxy_http_version    1.1;
        proxy_set_header      Upgrade "websocket";
        proxy_set_header      Connection "Upgrade";
        proxy_read_timeout    86400;
    }
	location ~ /terminals/ {
        proxy_pass            http://jupyter-notebook;
        proxy_set_header      Host $host;
        # websocket support
        proxy_http_version    1.1;
        proxy_set_header      Upgrade "websocket";
        proxy_set_header      Connection "Upgrade";
        proxy_read_timeout    86400;
	}
}
```

- 第二、第三行为监听端口，HTTP 为 80 端口，HTTPS 为 443 端口

- 第四行为希望监听到的域名
- 13 - 19 行为 SSL 证书设置，需要注意13 - 14 行为你的证书路径，需要是**绝对路径** 
- 后面则是转发相关的配置

再次终端输入 `jupyter notebook` 运行服务，在浏览器中输入域名，不出意料就能看到输入密码的页面了。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200209161216.PNG)

### 保持后台运行

现在程序一直是占用着终端的，而且如果关闭终端连接就会中断服务。我们需要让 Jupyter Notebook 静静地呆在后台运行。

先创建 `jupyter.log` 日志文件记录运行状态。

```bash
touch ~/.jupyter/jupyter.log
```

然后执行命令：

```bash
nohup jupyter notebook > ~/.jupyter/jupyter.log 2>&1 &
```

其中 `nohub` 表示 `no hang up` 不挂起，默默在后台运行，并将输出重载到日志文件中。

此时如果要中止程序，则需要 `ps -A` 检查所有进程 pid，然后执行 `kill {pid}` 来中止服务。



## 其他语言配置

如果你学习的不是 Python 语言，但只要是在 [这个页面](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels) 提到过的语言，都是支持的（Jupyter 的版本达到要求的前提下）。在安装好 conda 后，可以使用命令：

```
conda install {Name}
```

来安装其他 kernel。

比如拿 C++ 举例，只需要执行：

```bash
conda install xeus-cling
```

**但是！**为了防止某些无法理解的错误发生（如不同环境间的冲突或某些库不兼容），请考虑新建一个虚拟环境，然后在里面单独安装。如创建一个名为 `cling` 的虚拟环境，当然也可以是别的：

```bash
# 新建环境
conda create -n cling

# 切换到新建的虚拟环境
conda activate cling
# 出错可以尝试 source activate cling

# 给新的环境安装 jupyter notebook
conda install jupyter notebook

# 安装 xeus-cling (C++)
conda install xeus-cling

# 检查是否成功安装 kernel
jupyter kernelspec list
```

如果未出错则会显示目前已经（在独立环境中和主环境中）安装的 kernel ：

```bash
python3    /anaconda3/envs/cling/share/jupyter/kernels/python3
xcpp11     /anaconda3/envs/cling/share/jupyter/kernels/xcpp11
xcpp14     /anaconda3/envs/cling/share/jupyter/kernels/xcpp14
xcpp17     /anaconda3/envs/cling/share/jupyter/kernels/xcpp17
```

这时候到 Web 端，新建操作就会可以选择多种语言了。

![图片来源网络](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20200209161215.png)



## 后

全文到这里就要暂告一段落了。我们讨论了：

- Jupyter Notebook 的安装和配置
- 将 Jupyter Notebook 借助 Nginx 得以在公网访问
- 通过 Conda 管理多语言配置

限于篇幅，这次重点介绍搭建的过程，而对于许多工具背后的设计理念和哲学并没有深究。

回顾 Jupyter Notebook，Web 端、方便、交互式、Markdown 、多元……，这些特点一直伴随着它。可是，你真的需要它吗？

需要明确的是，对于工具的探索终究是为了更好地服务于你，好工具的定义对于每个人都是不一样的。这款工具能够满足你的需求，能否提升效率，只有你自己说了算。而过度热忱于追求工具，淡化了任务本身，显然是一种本末倒置的行为。

所以，你会来玩玩这个吗？

---

[^1]: https://en.wikipedia.org/wiki/Integrated_development_environment
[^2]: https://en.wikipedia.org/wiki/Project_Jupyter

