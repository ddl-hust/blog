---
title: 网页小说制作 kindle 电子书
date: 2020-01-04
update:
slug: 
tag:
  - kindle
copyright: true
comment: true
---

## 弄啥咧

上个月看完了心理测量者动画第一季和第二季，剧场版还有刚刚完结的第三季。看完之后对这部作品想更加深入地了解一下。对反乌托邦题材的作品也更加感兴趣了，就像之前看的《1984》、《美丽新世界》等等。综合最近读过的书，以及生活中的遭遇，准备写一篇万字左右的感想。（:又挖坑啦😂

去亚马逊 kindle 商店里找了找并没有中文版，搜了一下貌似也没与找到纸质出版的中文版。读原文是不可能的啦，日本语苦手😂。只能去找现有的资源了，找了大半天终于在 [亲小说](http://www.qinxiaoshuo.com/book/PSYCHO-PASS心理测量者))上找到了资源😂，但并没有下载的选项，对于咱这种靠技术搬砖的社畜来讲，还是拒绝当伸手党，把这些文本抓取下来，自己制作一个 mobi 格式的电子书看吧。

![image-20200103094338061](https://blog.502.li/img/image-20200103094338061.png)

看了一下，心理测量者这本书的 [目录页面](https://www.wenku8.net/novel/1/1765/index.htm) 包含了所有的章节，就想到了大致思路。

- 从这本书的目录页面过滤出来所有章节的 URL
- 分析各个章节的源码，过滤出中文文本
- shell 脚本 for 循环，通过 curl 遍历上一步获取到的 URL ，通过关键字过滤出中文文本
- 获取完全部章节的文本之后，分析一下文本，添加 `<h2>` 标签二级标题，方便生成目录
- 昨晚上述步骤后，生成的文本中还有大量的 html 标签没有去除，需要使用 `pandoc` 转 `.docx`，或者转成 markdown
- 最终得到 `.docx` 之后，使用 `Calibre` 将 `.docx` 转换成 Kindle 支持的 `mobi` 格式

## 怎么弄

### 获取各章节的 URL

-  `ctrl + s` 另保存 `pass.html` 这个目录页的源代码，拿到这个页面的源代码之后，仔细分析一下，可以看到，这个页面的各个章节还是比较有规律的。通过简单的  grep 和 sed 替换就很轻松地把这些 URL 提取出来。

```html
<div class="chapters">
<a alt="序" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35056fec85e5b101448.html">序</a>
<a alt="01 犯罪系数" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b101449.html">01 犯罪系数</a>
<a alt="02 有能者" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144a.html">02 有能者</a>
<a alt="03 饲育的作法" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144b.html">03 饲育的作法</a>
<a alt="04 谁都不知道的你的假面" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144c.html">04 谁都不知道的你的假面</a>
<a alt="05 无人知道的你的面孔" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144d.html">05 无人知道的你的面孔</a>
<a alt="06 狂王子的归还" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144e.html">06 狂王子的归还</a>
<a alt="07 紫兰的花语" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b10144f.html">07 紫兰的花语</a>
<a alt="08 之后是，沉默" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101450.html">08 之后是，沉默</a>
<a alt="09 乐园的果实" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101451.html">09 乐园的果实</a>
<a alt="10 玛土撒拉的游戏" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101452.html">10 玛土撒拉的游戏</a>
<a alt="11 圣者的晚餐" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101453.html">11 圣者的晚餐</a>
</div>
</div>
<div class="volume volume_close">
<div class="volume_title" onclick="exchange_volume(this)">
第二卷 下
<i class="volume_expand"></i>
</div>
<div class="volume_info">
<img src="./PSYCHO-PASS心理测量者_轻小说在线阅读_亲小说网_files/1716.jpg" onerror="this.src=&#39;http:\/\/static.qinxiaoshuo.com:4000/bookimg/1716.jpg&#39;;this.onerror=null">
<span>第二卷 下</span>
</div>
<div class="chapters">
<a alt="XX 消逝的情人节" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101454.html">XX 消逝的情人节</a>
<a alt="12 youthful days" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101455.html">12 youthful days</a>
<a alt="13 深渊来的招待" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b101456.html">13 深渊来的招待</a>
<a alt="14 甜蜜的毒" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b101457.html">14 甜蜜的毒</a>
<a alt="15 硫磺洒落的城市" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b101458.html">15 硫磺洒落的城市</a>
<a alt="16 制裁之门" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b101459.html">16 制裁之门</a>
<a alt="17 铁石心肠" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b10145a.html">17 铁石心肠</a>
<a alt="18 无果之约" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b10145b.html">18 无果之约</a>
<a alt="19 透明的影" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b10145c.html">19 透明的影</a>
<a alt="20 正义所在" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b10145d.html">20 正义所在</a>
<a alt="21 血的褒奖" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b10145e.html">21 血的褒奖</a>
<a alt="22 完美的世界" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b10145f.html">22 完美的世界</a>
<a alt="尾声" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b101460.html">尾声</a>
<a alt="唐之杜志恩和六合冢弥生" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b101461.html">唐之杜志恩和六合冢弥生</a>
<a alt="涛声响彻的房间" href="http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b101462.html">涛声响彻的房间</a>
```

- 其中 `alt` 是过滤出每个章节 `URL` 行的关键字
- 使用 `awk` 截取去掉 `html` 标签，只保留 `URL` 
- 其实应该用正则表达式来截取这些 URL ，哎咱太菜了，书到用时方恨少啊😥，正则不会 `awk` 来凑😂

```bash
grep alt pass.html | awk -F "href=\"" '{print $2}'| awk -F "\"" '{print $1}'
http://www.qinxiaoshuo.com/read/0/1716/5d77d35056fec85e5b101448.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b101449.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144a.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144b.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144c.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144d.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35156fec85e5b10144e.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b10144f.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101450.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101451.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101452.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101453.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101454.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35256fec85e5b101455.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b101456.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b101457.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b101458.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b101459.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b10145a.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b10145b.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35356fec85e5b10145c.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b10145d.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b10145e.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b10145f.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b101460.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b101461.html
http://www.qinxiaoshuo.com/read/0/1716/5d77d35456fec85e5b101462.html
```

### 分析各个章节的源码

![image-20200103095824495](https://blog.502.li/img/image-20200103095824495.png)

```html
<div id="chapter_content">
            第一卷 上  序<br/><br/>网译版 转自 豆瓣()<br/><br/>翻译：クリーオウ<br/><br/>……………………
</div>
```

可见，每个章节的中文文本，全部在 `<div id="chapter_content">` ，而且中文文本全部压缩在了一行里，可以通过 ``<br/><br/>`` 这个关键字来过滤出这一行中文文本。

```bash
curl http://www.qinxiaoshuo.com/read/0/1716/5d77d35056fec85e5b101448.html | grep "<br/><br/>"
```

![image-20200103100302697](https://blog.502.li/img/image-20200103100302697.png)

### 遍历各个章节

```bash
╭─root@debian ~/book
╰─$ grep alt pass.html | awk -F "href=\"" '{print $2}'| awk -F "\"" '{print $1}'> url.log
╭─root@debian ~/book
╰─$ for url in $(cat url.log);do curl ${url} | grep "<br/><br/>" >> pass_book.html;done
```

### 添加 `<h2>` 二级标题

在每一行的行首添加一个 ``<h2>`` 标签，接着使用 sed 替换每一行第一个 `<br/><br/>` 匹配项为 `<h2/>` 一定要添加闭合标签，不然的话在转换成 markdown 或者 .docx 时候会把这一整行当作二级标题。

```bash
sed -i 's/^/<h2>/g' pass_book.html
sed -i 's/<br\/><br\/>/<h2\/>/' pass_book.html
```

### 转换为 markdown 格式

使用 `pandoc` 转 `markdown` 的效果很不理想，其中有大量的 `\` 需要删除掉，还是转 .docx 的格式合适一些，而且 `Calibre` 也支持 `.docx` 格式

![image-20200103103719653](https://blog.502.li/img/image-20200103103719653.png)

### 转 mobi 格式

打开 calibre 导入刚刚制作好的 .docx 文件，点击 `转换书籍` 那里，根据对应的设备设置好参数。

![image-20200103104121602](https://blog.502.li/img/image-20200103104121602.png)

### 制作完成

![image-20200103105014635](https://blog.502.li/img/image-20200103105014635.png)

![image-20200103105027192](https://blog.502.li/img/image-20200103105027192.png)

![image-20200103105034243](https://blog.502.li/img/image-20200103105034243.png)

### 效果

![image-20200103104618047](https://blog.502.li/img/image-20200103104618047.png)

### 注意事项

- 由于总共文本有 20 万字，转换成 azw3 格式会卡死，不知道什么原因😥
- 还有什么更好的办法评论区交流一哈😂
