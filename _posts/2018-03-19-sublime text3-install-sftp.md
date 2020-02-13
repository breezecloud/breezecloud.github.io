---
layout: post
title:  "sublime text3 安装sftp插件"
date: 2018-03-19 20:25:06 -0700
---
![](/images/2018-03-19-1-1.png)  
&emsp;&emsp;sublime是我个人最喜欢的文本编辑器，它短小精悍，功能齐全，支持海量插件。虽说不是免费软件但实在是令人爱不释手，无法放弃。可是每次重新安装sublime时安装插件都挺折腾的，网上方法虽多但不一定成功，这次把安装过程记录下来，也是为了能给自己做一个记录，将来可以方便查阅。
1. 下载安装sublime text3
    下载地址https://www.sublimetext.com/3，当前版本是3143。该软件是收费软件（个人用户$80，my god！），没钱的主请自行搜索注册码注册。
2. 安装Package Control组件，按ctrl＋｀组合键，输入以下内容后按Enter键
  仅针对sublime text 3 版本：
```
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```
![](/images/2018-03-19-1-2.png)  
  当状态栏显示安装完毕后退出sublime，重新进入。
3. window 按ctrl＋shift＋p， mac 按 command＋shift＋p键。 在弹出的输入框中输入： install  然后选择第一条install package。在接下来的输入框中输入：sftp，并选择SFTP插件，完成安装之后查看菜单就又sftp的菜单。
![](/images/2018-03-19-1-3.png)  
![](/images/2018-03-19-1-4.png)  
![](/images/2018-03-19-1-5.png)  
&emsp;&emsp;sftp插件如何使用请参考https://www.cnblogs.com/shined/p/5799393.html

&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin.jpg)

