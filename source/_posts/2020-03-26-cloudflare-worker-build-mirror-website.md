---
title: 使用 CloudFlare Workers 搭建 telegram 频道镜像站
date: 2020-03-26
updated:
slug: cloudflare-worker-build-mirror-website
categories: 技术
tag:
  - GFW
  - CloudFlare
  - telegram
copyright: true
comment: true
---

## 一次偶遇

昨天在咱的 [让图片飞起来 oh-my-webp.sh ！](https://blog.k8s.li/oh-my-webpsh.html) 收到了 [Chris](https://ichr.me/) 大佬的回复，咱就拜访了一下大佬的博客😂，无意间发现 [Cloudflare Worder 免费搭建镜像站](https://blog.ichr.me/post/cloudflare-worker-build-mirror-website/) 这篇博客。于是呐，咱也想着能不能玩一玩这个 Workers 。虽然之前听说过有用 Workers 做很多好玩的事儿，比如加速网站、代理 Google 镜像站什么的。不过这写对于咱来说不是很刚需就没有折腾。刚好咱的 telegram 电报频道 [RSS Kubernetes](https://t.me/rss_kubernetes) 人也比较多了，为了提高一点影响力，咱就想着能不能把频道的预览界面反代到咱域名上。虽然之前尝试使用 nginx 进行反代，但是效果不尽人意，于是当时就弃坑没在玩儿。在春节的时候咱也看到过有人反代的，不过那个项目折腾起来也是很麻烦，咱这种菜鸡还是溜了溜了。看到 [Cloudflare Worder 免费搭建镜像站](https://blog.ichr.me/post/cloudflare-worker-build-mirror-website/) 这篇博客后就心血来潮，就搞一搞吧😂

## 劝退~~三~~一连

首先你要有个 Cloudflare 账户，这时必须的。关于 Cloudflare 的注册咱就不多说啦，不过咱倒是建议大家伙把域名的 DNS 解析放到 Cloudflare 上来，好处多多：有把 https 小绿锁、免费的 ~~加速~~ 减速 CDN （墙内）、域名访问统计等等可玩性比较高😋。需要注意的是 Cloudflare 的 Worker 一天 10 万次免费额度，也够咱喝一壶的啦，不用担心不够用。

## 新建 Worker

登录到 [Cloudflare](https://dash.cloudflare.com)的~~大盘~~面板，点击左上角的 `Menu` ----> `Worker` 进入到 Worker 页面。新注册的用户会提示设置一个 `workers.dev` 顶级域名下的二级子域名，这个子域名名称设置好之后是不可更改的，之后你新创建的 Worker 就会使以这个域名而二级子域名开始的，类似于  `WorkerName.yousetdomain.workers.dev` 。`yousetdomain` 就是你要设置的二级子域名，`WorkerName` 可以自定义，默认是随机生成的。也可以给自己的域名添加一条 CNAME 到 `WorkerName.yousetdomain.workers.dev` ，这样使用自己的域名就可以访问到 `Worker` 了。

设置好二级子域名之后选择 free 套餐计划，然后进入到 Worker 管理界面，创建一个新的 `Worker`  然后再 `Script` 输入框里填入以下代码。**俗话说代码写得好，同行抄到老**，咱的 `worker.js` 代码是参考自 [Workers-Proxy](https://github.com/Siujoeng-Lau/Workers-Proxy) ，不过咱去掉了一些无关紧要的代码，原代码是加入了辨别移动设备适配域名、屏蔽某些 IP 和国家的功能。对于咱的 telegram 频道镜像站来说，这都是多余的，于是就去掉了。

![image-20200326184802562](https://blog.502.li/img/image-20200326184802562.png)

```javascript
// Website you intended to retrieve for users.
const upstream = 't.me'

// Custom pathname for the upstream website.
const upstream_path = '/s/rss_kubernetes'

const https = true

// Replace texts.
const replace_dict = {
    '$upstream': '$custom_domain',
    'telegram.org': '$custom_domain'
}

addEventListener('fetch', event => {
    event.respondWith(fetchAndApply(event.request));
})

async function fetchAndApply(request) {

    let response = null;
    let url = new URL(request.url);
    let url_hostname = url.hostname;

    if (https == true) {
        url.protocol = 'https:';
    } else {
        url.protocol = 'http:';
    }
    
    var upstream_domain = upstream;
    url.host = upstream_domain;
    if (url.pathname == '/') {
        url.pathname = upstream_path;
    } else {
        url.pathname = upstream_path + url.pathname;
    }

    let method = request.method;
    let request_headers = request.headers;
    let new_request_headers = new Headers(request_headers);

    new_request_headers.set('Host', url.hostname);
    new_request_headers.set('Referer', url.hostname);

    let original_response = await fetch(url.href, {
        method: method,
        headers: new_request_headers
    })

    let original_response_clone = original_response.clone();
    let original_text = null;
    let response_headers = original_response.headers;
    let new_response_headers = new Headers(response_headers);
    let status = original_response.status;

    new_response_headers.set('access-control-allow-origin', '*');
    new_response_headers.set('access-control-allow-credentials', true);
    new_response_headers.delete('content-security-policy');
    new_response_headers.delete('content-security-policy-report-only');
    new_response_headers.delete('clear-site-data');

    const content_type = new_response_headers.get('content-type');
    if (content_type.includes('text/html') && content_type.includes('UTF-8')) {
        original_text = await replace_response_text(original_response_clone, upstream_domain, url_hostname);
    } else {
        original_text = original_response_clone.body
    }

    response = new Response(original_text, {
        status,
        headers: new_response_headers
    })
    return response;
}

async function replace_response_text(response, upstream_domain, host_name) {
    let text = await response.text()

    var i, j;
    for (i in replace_dict) {
        j = replace_dict[i]
        if (i == '$upstream') {
            i = upstream_domain
        } else if (i == '$custom_domain') {
            i = host_name
        }

        if (j == '$upstream') {
            j = upstream_domain
        } else if (j == '$custom_domain') {
            j = host_name
        }
        
        let re = new RegExp(i, 'g')
        text = text.replace(re, j);
    }
    return text;
}
```

需要注意的是，像 `t.me` 域名下的站点，比如我的 `https://t.me/s/rss_kubernetes` ，它的 js 和 css 样式文件是使用的 telegram.org 域名。所以我们需要在 `replace_dict` 那里定义好替换的正则表达式，将  `https://t.me/s/rss_kubernetes`页面里的  `telegram.org` 同样进行反代才行。

```shell
ubuntu@blog:~$ curl https://t.me/s/rss_kubernetes | grep "<script src="
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  141k  100  141k    0     0  85287      0  0:00:01  0:00:01 --:--:-- 85237
    <script src="//telegram.org/js/jquery.min.js"></script>
    <script src="//telegram.org/js/jquery-ui.min.js"></script>
    <script src="//telegram.org/js/widget-frame.js?29"></script>
    <script src="//telegram.org/js/telegram-web.js?8"></script>

```

这个文本替换功能很好玩儿，在 Cloudflare 官方的博客里还有个 demo [introducing-cloudflare-workers](https://cloudflareworkers.com/#c62c6c0002cb236166b794c440870cca:https://blog.cloudflare.com/introducing-cloudflare-workers) 。使用这个功能咱有解锁了一个玩具，稍后再讲😂。

>   Here is a worker which performs a site-wide search-and-replace, replacing the word "Worker" with "Minion". [Try it out on this blog post.](https://cloudflareworkers.com/#c62c6c0002cb236166b794c440870cca:https://blog.cloudflare.com/introducing-cloudflare-workers)

~~剽窃~~修改好代码之后点击左下角的 `Save and Deploy` 然后 Preview 看看页面是否显示正常，如果显示正常恭喜你成功啦。

![image-20200326190914520](https://blog.502.li/img/image-20200326190914520.png)

## 自定义域名

回到 Worker 的管理页面，点击 `rename` 即可修改 Worker 的三级子域名。不过咱还是不太喜欢 `WorkerName.yousetdomain.workers.dev` 这么长的域名，想使用咱自己的二级子域名访问。

首先回到域名管理的页面，进入到自己域名顶部那一栏里的 `Workers` ，在那里添加相应的路由即可。

![image-20200326191340166](https://blog.502.li/img/image-20200326191340166.png) 

点击 `Add Route` ，再 Route 那一栏输入好自己的域名，注意最后的 `/*` 也要加上，然后 Worker 选择刚刚创建的那个即可。接着再添加 `CNAME` 记录到自己的 `WorkerName.yousetdomain.workers.dev` ，这样就能使用自己的域名访问啦😋

![image-20200326191435923](https://blog.502.li/img/image-20200326191435923.png)

## 文本替换

前文提到的文本替换功能帮咱解决了一个小问题。咱友链关注的一个博客 [chanshiyu.com](https://chanshiyu.com/#/) 没有提供 RSS ，他是将博客内容放在 GitHub issue 上，所以只能通过 RSSHUB 来订阅 GitHub 的 issue 来获取博客的更新。但 RSSHUB 获取的是 GitHub issue 的链接而非原博客的链接，于是咱想了想就用 `Worker` 进行替换不得了。

通过 RSSHUB 获取到的 RSS 数据如下：

```xml
<rss xmlns:atom="http://www.w3.org/2005/Atom" version="2.0">
<channel>
<title>
<![CDATA[ chanshiyucx/blog Issues ]]>
</title>
<link>https://github.com/chanshiyucx/blog/issues</link>
<atom:link href="http://rsshub.app/github/issue/chanshiyucx/blog" rel="self" type="application/rss+xml"/>
<description>
<![CDATA[
chanshiyucx/blog Issues - Made with love by RSSHub(https://github.com/DIYgod/RSSHub)
]]>
</description>
<generator>RSSHub</generator>
<webMaster>i@diygod.me (DIYgod)</webMaster>
<language>zh-cn</language>
<lastBuildDate>Thu, 26 Mar 2020 11:00:06 GMT</lastBuildDate>
<ttl>60</ttl>
<item>
<title>
<![CDATA[ Telegram 电报机器人 ]]>
</title>
<description>
<![CDATA[
]]>
</description>
<pubDate>Wed, 25 Mar 2020 03:54:04 GMT</pubDate>
<guid isPermaLink="false">https://github.com/chanshiyucx/blog/issues/108</guid>
<link>https://github.com/chanshiyucx/blog/issues/108</link>
```

其中的 `<guid isPermaLink="false"> ` 和 `<link>` 中的链接 github.com/chanshiyucx/blog/issues/ 替换为 chanshiyu,com/post/ 即可。于是还是同样的方法新建一个 Worker，然后修改一下 `worker.js` 的代码就可以啦。

```javascript
// Website you intended to retrieve for users.
const upstream = 'rsshub.app'

// Custom pathname for the upstream website.
const upstream_path = '/github/issue/chanshiyucx/blog'

// Whether to use HTTPS protocol for upstream address.
const https = true

// Replace texts.
const replace_dict = {
    '$upstream': '$custom_domain'
}

addEventListener('fetch', event => {
    event.respondWith(fetchAndApply(event.request));
})

async function fetchAndApply(request) {

    const region = request.headers.get('cf-ipcountry').toUpperCase();
    const ip_address = request.headers.get('cf-connecting-ip');
    const user_agent = request.headers.get('user-agent');

    let response = null;
    let url = new URL(request.url);
    let url_hostname = url.hostname;

    if (https == true) {
        url.protocol = 'https:';
    } else {
        url.protocol = 'http:';
    }

    var upstream_domain = upstream;
    url.host = upstream_domain;
    if (url.pathname == '/') {
        url.pathname = upstream_path;
    } else {
        url.pathname = upstream_path + url.pathname;
    }

    let method = request.method;
    let request_headers = request.headers;
    let new_request_headers = new Headers(request_headers);

    new_request_headers.set('Host', url.hostname);
    new_request_headers.set('Referer', url.hostname);

    let original_response = await fetch(url.href, {
        method: method,
        headers: new_request_headers
    })

    let original_response_clone = original_response.clone();
    let original_text = null;
    let response_headers = original_response.headers;
    let new_response_headers = new Headers(response_headers);
    let status = original_response.status;

    const content_type = new_response_headers.get('content-type');
    if (content_type.includes('text/html') && content_type.includes('UTF-8')) {
        original_text = await replace_response_text(original_response_clone, upstream_domain, url_hostname);
    } else {
        original_text = original_response_clone.body
    }

    response = new Response(original_text, {
        status,
        headers: new_response_headers
    })
    let text = await response.text()

  // Modify it.
  let modified = text.replace(
      /github.com\/chanshiyucx\/blog\/issues\//g, "chanshiyu.com\/#\/post\/")

  // Return modified response.
  return new Response(modified, {
    status: response.status,
    statusText: response.statusText,
    headers: response.headers
  })
    return response;
}
```

部署好 Worker 之后就可以使用 RSS 来订阅啦😋

![image-20200326193643610](https://blog.502.li/img/image-20200326193643610.png)

## 解锁更多好玩儿的😋

关于 Cloudflare 的 Workers 还有更多好玩的等待你去发现，咱就推荐一下啦：

-   [Cloudflare Worker 免费搭建镜像站](https://blog.ichr.me/post/cloudflare-worker-build-mirror-website/)
-   [从现在起，任何人都可以在Cloudflare上使用Workers运行JavaScript！](https://blog.cloudflare.com/zh/cloudflare-workers-unleashed-zh/)
-   [尽情编写代码吧：改善开发人员使用Cloudflare Workers的体验](https://blog.cloudflare.com/zh/just-write-code-improving-developer-experience-for-cloudflare-workers-zh/)
-   [使用 Cloudflare Workers 加速 Google Analytics](https://blog.skk.moe/post/cloudflare-workers-cfga)
-   [使用 Backblaze B2 和 Cloudflare Workers 搭建可以自定义域名的免费图床](https://blog.meow.page/2019/09/24/free-personal-image-hosting-with-backblaze-b2-and-cloudflare-workers)
-   [使用 Cloudflare Workers 提高 WordPress 速度和效能教學](https://free.com.tw/cloudflare-workers-wordpress/)
-   [使用Cloudflare Workers反带P站图片](https://yojigen.tech/archives/post19/)

最后宣传一下咱的[@rss_kubernetes](https://t.me/rss_kubernetes) 频道，国内用户可以访问 [tg.k8s.li](https://tg.k8s.li)，如果你对 docker 、K8s、云原生等感兴趣，就到咱碗里来吧😂。不订阅咱的频道也可以通过咱的 [tg.k8s.li](https://tg.k8s.li) 镜像站来查看 RSS 推送信息。