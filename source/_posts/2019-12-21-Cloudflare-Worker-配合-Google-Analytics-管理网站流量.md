---
layout: post
title: Cloudflare Worker 配合 Google Analytics 管理网站流量
ptype: post
mathjax: false
noindex: false
nofollow: true
date: 2019-12-21 16:46:50
categories:
 - 笔记本
tags:
 - SEO
 - Analytics
 - Cloudflare
keywords: cloudflare, google analytics, cloudflare worker, 流量, SEO, 教程
description: 
purl: google-analytics-cooperates-cloudflare-worker-to-manage-your-website-trafic
---



笔者一直使用 Google Analytics 分析流量，以助于更好的提供内容。但是由于大多访问量来自国内，Google 国内的数据中心又……

又于是在浏览 [@SukkaW](https://skk.moe) 的博客的时候偶然发现了解决方法，便在这里分享出来，以供大家参考。

<!--more--> 

---

### Cloudflare Workers 简介

官网介绍：

> Build serverless applications on Cloudflare's global cloud network spanning 194 cities across over 90 countries. Cloudflare Workers provides a lightweight JavaScript execution environment that allows developers to augment existing applications or create entirely new ones without configuring or maintaining infrastructure.

翻译：

> 利用 Cloudflare 90 多个国家／194 个城市的全球云网络搭建一个**无后端**的应用程序。Cloudflare Workers 提供了一个轻量的执行环境，使开发人员无需配置维护基础架构就可以扩展或者构建一个应用程序。



简而言之就是提供轻量级的服务器直接实现一些无后端的服务。

Cloudflare 国内还和百度有合作，光国内就又有 20 余台服务器。我们可以很方便的将用 Cloudflare Workers 来将数据转发至 Google Analytics 。

Cloudflare 依然十分慷慨地给了每日 10 万次的免费请求额度，一般的站点应该都是够的了，这里分享一下如何实现。

---

### 使用

#### 1. 新建一个 `Worker` 

登录进你的 Cloudflare 账号，然后进入 `Workers` App (Menu -> Workers) 。

然后新建一个 `Worker` ，删除原来的代码，再加入以下代码。

```
//const AllowedReferrer = 'skk.moe'; // ['skk.moe', 'suka.js.org'] multiple domains is supported in array format

addEventListener('fetch', (event) => {
    event.respondWith(response(event));
});

async function senData(event, url, uuid, user_agent, page_url) {
    const encode = (data) => encodeURIComponent(decodeURIComponent(data));

    const getReqHeader = (key) => event.request.headers.get(key);
    const getQueryString = (name) => {
        const pattern = new RegExp('(^|&)' + name + '=([^&]*)(&|$)', 'i');
        const r = url.search.substr(1).match(pattern);
        return (r !== null) ? unescape(r[2]) : null;
    };

    const reqParameter = {
        headers: {
            'Host': 'www.google-analytics.com',
            'User-Agent': user_agent,
            'Accept': getReqHeader('Accept'),
            'Accept-Language': getReqHeader('Accept-Language'),
            'Accept-Encoding': getReqHeader('Accept-Encoding'),
            'Cache-Control': 'max-age=0'
        }
    };

    const pvData = `tid=${encode(getQueryString('ga'))}&cid=${uuid}&dl=${encode(page_url)}&uip=${getReqHeader('CF-Connecting-IP')}&ua=${user_agent}&dt=${encode(getQueryString('dt'))}&de=${encode(getQueryString('de'))}&dr=${encode(getQueryString('dr'))}&ul=${getQueryString('ul')}&sd=${getQueryString('sd')}&sr=${getQueryString('sr')}&vp=${getQueryString('vp')}`;

    const perfData = `plt=${getQueryString('plt')}&dns=${getQueryString('dns')}&pdt=${getQueryString('pdt')}&rrt=${getQueryString('rrt')}&tcp=${getQueryString('tcp')}&srt=${getQueryString('srt')}&dit=${getQueryString('dit')}&clt=${getQueryString('clt')}`

    const pvUrl = `https://www.google-analytics.com/collect?v=1&t=pageview&${pvData}&z=${getQueryString('z')}`;
    const perfUrl = `https://www.google-analytics.com/collect?v=1&t=timing&${pvData}&${perfData}&z=${getQueryString('z')}`

    await fetch(pvUrl, reqParameter);
    await fetch(perfUrl, reqParameter);
}

async function response(event) {
    const url = new URL(event.request.url);

    const getReqHeader = (key) => event.request.headers.get(key);

    const Referer = getReqHeader('Referer');
    const user_agent = getReqHeader('User-Agent');
    const ref_host = (() => {	
        try {
            return new URL(Referer).hostname;
        } catch (e) {
            return ""
        }
    })();

    let needBlock = false;

    needBlock = (!ref_host || ref_host === '' || !user_agent || !url.search.includes('ga=UA-')) ? true : false;

    if (typeof AllowedReferrer !== 'undefined' && AllowedReferrer !== null && AllowedReferrer) {
      let _AllowedReferrer = AllowedReferrer;

      if (!Array.isArray(AllowedReferrer)) _AllowedReferrer = [_AllowedReferrer];
    
      const rAllowedReferrer = new RegExp(_AllowedReferrer.join('|'), 'g');

      needBlock = (!rAllowedReferrer.test(ref_host)) ? true : false;
      console.log(_AllowedReferrer, rAllowedReferrer, ref_host);
    }

    if (needBlock) {
        return new Response('403 Forbidden', {
            headers: { 'Content-Type': 'text/html' },
            status: 403,
            statusText: 'Forbidden'
        });
    }

    const getCookie = (name) => {
        const pattern = new RegExp('(^| )' + name + '=([^;]*)(;|$)');
        const r = (getReqHeader('cookie') || '').match(pattern);
        return (r !== null) ? unescape(r[2]) : null;
    };

    const createUuid = () => {
        let s = [];
        const hexDigits = '0123456789abcdef';
        for (let i = 0; i < 36; i++) {
            s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
        }
        s[14] = '4'; // bits 12-15 of the time_hi_and_version field to 0010
        s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1); // bits 6-7 of the clock_seq_hi_and_reserved to 01
        s[8] = s[13] = s[18] = s[23] = '-';

        return s.join('');
    };

    const _uuid = getCookie('uuid');
    const uuid = (_uuid) ? _uuid : createUuid();

    // To sent data to google analytics after response id finished
    event.waitUntil(senData(event, url, uuid, user_agent, Referer));

    // Return an 204 to speed up: No need to download a gif
    let response = new Response(null, {
        status: 204,
        statusText: 'No Content'
    });

    if (!_uuid) response.headers.set('Set-Cookie', `uuid=${uuid}; Expires=${new Date((new Date().getTime() + 365 * 86400 * 30 * 1000)).toGMTString()}; Path='/';`);

    return response
}
```

保存、应用即可。

#### 2. 插入对应 JS 到网站

插入以下文本到你的网站的 `</head>` 里，注意将默认信息换成是你自己的。

```
<script>
  window.ga_tid = "UA-XXXXX-Y"; // The trackerID of your site.
  window.ga_api = "https://example.com/xxx/"; // The route of your cloudflare workers you just registered before.
</script>
<script src="https://cdn.jsdelivr.net/npm/cfga@1.0.1" async></script>
```

关于 `ga_tid` 应该不用说，Google Analytics 的 Track ID ，在 [管理 -> 跟踪信息 -> 跟踪代码] 主可以找到。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20191221210306.jpg)

`ga_api` 后跟刚刚创建 Worker 的路径，类似 `workername.yourname.worker.dev` 的形式，可以直接在这获取。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20191221210613.jpg)

如果你也使用 Hexo ，只需在**配置文件**的类似 `/layout/_partial/` 内会有一个 `head` 为文件名的文件，在块 `</head>` 下加入，在下次 Generate 的时候就会给每个页面添加上那一段代码了。

![](https://cdn.jsdelivr.net/gh/chralpha/imgbag/20191221210805.jpg)



---



### 完成

按照上述步骤走下来，不出意料已经可以完成了。

你可以打开监测网站的一个页面，看看 Google Analytics 是否有实时访问记录。



Ended. 

