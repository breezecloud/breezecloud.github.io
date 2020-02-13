---
layout: post
title:  "jekyll简明使用说明"
date: 2019-2-28 08:15:00 +0800
categories: jekyll update
---

1. 什么是jekyll？
jekyll是一个简单的免费的Blog生成工具（将纯文本转化为静态网站和博客），类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如Disqus。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。我们可以在客户端生成静态网站，然后上传到服务器。

2. 安装
* 下载windows安装包，参考http://www.madhur.co.in/blog/2013/07/20/buildportablejekyll.html
软件包位于:https://github.com/madhur/PortableJekyll ,已经包括git包
* git clone https://github.com/scotte/jekyll-clean.git #下载主题
* gem isntall bundler #更新bunller
* bundle install #安装依赖
* 启动
cd jekyll-clean
jekyll server # 启动本地 http://127.0.0.1/jekyll-clean ( jekyll serve --watch)
jekyll build #重新生成静态网站

3. 上传到git
* 新建github pages
* git clone git@github.com:yourname/yourname.github.io.git #复制github
* 下载jekyll模板，并安装上面的步骤本地测试
* 将下载的模板copy到yourname.github.io目录，注意不要覆盖.git目录
* 修改_config.yml
url: https://scotte.github.io/jekyll-clean 改为https://breezecloud.github.io/
baseurl: /jekyll-clean 改为 ''(根目录)
* 上传到服务器
git add . 
git commit -a -m "版本说明"
git push 
* jekyll serve 浏览https://breezecloud.github.io/
* 设置域名，如果你想设置自己的域名，在breezecloud.github.io的目录下有一个CNAME目录，将自己的域名写如（比如：youery.cn），同时设置域名解析服务器增加记录（@表示直接解析）
<br>![](/images/2019-2-28-1.png)


4. 添加文章
_post目录下增加md文件，增加头信息如：
```
---
layout: post
title:  "给树莓派安装ArchLinux"
date: 2017-12-15 16:25:06 -0700
---
```
生成的静态网站会自动根据时间排序，之后执行jekyll build重新生成site<br>
参考：<br>
https://www.jianshu.com/p/9f71e260925d<br>
https://www.jekyll.com.cn