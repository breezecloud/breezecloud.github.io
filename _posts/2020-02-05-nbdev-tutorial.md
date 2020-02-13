---
layout: post
title:  "jupyter-notebook&nbdev&github实现python开发全过程"
date: 2020-02-05 08:15:00 +0800
categories: jekyll update
---

前言
===
&emsp;&emsp;以前一直用sublime文本编辑器写代码。不得不说作为一个业余编程爱好者，这款文本编辑器太好用了，几乎满足了所有编写代码的需求。但同时缺点也很明显，你必须写完代码到命令行窗口下测试，或者版本管理等任何其它事情，虽然sublime有很多插件也许能满足你的需求，但毕竟插件功能有限，也不一定很好找到合适的。直到最近看到有一篇介绍nbdev的文章，介绍基于jupyter-notebook的nbdev说成可以完成整个编程生命周期的神器，可以结合github自动完成很多工作。正好在冠状肺炎病毒肆虐之际，有时间在家里学习一下，是否可以能让我能鸟枪换炮呢？

jupyter-notebook
===
&emsp;&emsp;先来说说jupyter-notebook。这是一个交互式笔记本，支持运行40 多种编程语言。Jupyter-Notebook在浏览器中创建和共享程序文档，支持实时代码，数学方程，可视化和 markdown。 -用途包括：数据清理和转换，数值模拟，统计建模，机器学习等等。如果用它来写python，有点类似python自带的IDLE，但比IDLE功能多，编写更多类型的文件，可以保存代码运行的结果。它具有以下优势：<br>-
-可选择语言：支持超过40种编程语言，包括Python、R、Julia、Scala等。
-分享笔记本：可以使用电子邮件、Dropbox、GitHub和Jupyter Notebook Viewer与他人共享。
-交互式输出：代码可以生成丰富的交互式输出，包括HTML、图像、视频、LaTeX等等。
-大数据整合：通过Python、R、Scala编程语言使用Apache Spark等大数据框架工具。支持使用pandas、scikit-learn、ggplot2、TensorFlow来探索同一份数据

1.  安装
&emsp;&emsp;安装很简单，因为是基于python开发，直接用```pip install jupyter```经过漫长的等待顺利安装完成（pip服务器实在太慢，考虑换一个国内的）。安装时注意python和其下面的pip命令执行路径是否在PATH环境变量中，如果不在建议加上。
2. 运行
&emsp;&emsp;在命令行下执行：
```
jupyter notebook
```
命令会启动一个web服务器，并自动打开默认浏览器，自动指向http://localhost:8888/tree的链接，显示当前目录的文件列表。
3. 简单的使用
```
jupyter notebook --generate-config
```
在用户目录下生成配置文件，打开“.jupyter”文件夹，可以看到里面有个配置文件，下次启动会使用这个默认配置文件。
新建一个文档：
![](/images/2020-2-5-1.png)
输入：
![](/images/2020-2-5-2.png)
在这里输入一条然后运行：
![](/images/2020-2-5-3.png)
查看快捷键：
![](/images/2020-2-5-4.png)
这样jupyter基本使用就可以了，我们也可以使用ctl+s随时保存文档；这样我们所有操作记录就保存下来，下次打开直接使用。

nbdev介绍
===
&emsp;&emsp; notebook 或 REPL 不具备的功能，比如：优秀的文档查找功能、优秀的语法高亮功能、集成单元测试，以及（关键的）生成最终可分发源代码文件的能力。nbdev增强了所谓“探索式”编程的功能，同时将 IDE/编辑器开发的优势带入 notebook 系统中，以便用户在 notebook 中完成开发，且不会影响整个项目生命周期。本文的大部分内容基于！[nbdev教程](https://nbdev.fast.ai/tutorial/)


安装
===
&emsp;&emsp;和安装其他python模块一样：
```
pip install nbdev
```

设置Repo
===
1. 新建模板
&emsp;&emsp;在登录自己github账号之后，点击nbdev template(https://github.com/fastai/nbdev_template/generate) 。会自动在自己git账号下产生一个Repo，包括一个符合nbdev的最小模板，你可以以此为起点，将该库clone到本地后进行代码编写。

2. github pages
&emsp;&emsp;该模板doc目录下是一个基于github的个人主页（项目文档）。要启用该主页，需到自己的github上打开刚才生成的Repo，在进入setting菜单，下列到Github Pages,设置"Source" to Master branch /docs选项，保存之后重新回到该选项，会有一个url就代表这个Repo的文档主页。这时可以回到Repo的页面，点击Edit，在Website输入框将刚才的url写入。

修改 settings.init
===
&emsp;&emsp;这个文件自动生成在跟目录下，包含了需要打包库文件的所有信息，去掉以下一些注释去掉(否则生成代码会报错):<br>
```
# lib_name = your_project_name
# user = your_github_username
# description = A description of your project
# keywords = some keywords
# author = Your Name
# author_email = email@example.com
# copyright = Your Name or Company Name
```
目录结构一个为根目录下有jupyter notebokk的文件；doc目录（刚才已经提到）；一个打包生产你python代码的目录。

安装git 钩子
===
&emsp;&emsp;Jupyter Notebooks可能会引起版本冲突，可以在终端中cd到项目文件目录下执行:
```
nbdev_install_git_hooks
```
&emsp;&emsp;这样在commit时会去掉元数据(metadata)，以减少冲突的可能

编辑 00_core.ipynb
===
&emsp;&emsp;编辑00_core.ipynb，你的代码可以从这个文件开始写
```
# default_exp core
```
表示会产生 lib_name/core.py库
```
#export
def say_hello(to):
    "Say hello to somebody"
    return f'Hello {to}!'
```
定义一个函数，#export表示会包含在生成的库文件中。
之后你可以写一些说明文档，包括增加一个测试脚本：
```
assert say_hello("Jeremy")=="Hello Jeremy!"
```

生成代码
===
&emsp;&emsp;你可以在命令行中执行nbdev_build_lib，或者在notebook中执行：
```
from nbdev.export import *
notebook2script()
```
结束之后，可以看到在lib_name目录下生成了相应的core.py文件

编辑index.ipynb
===
&emsp;&emsp;该文档会自动生成doc目录下的index.html。是整个项目的说明文档的起始页面。 帮助文件中的例子应该是notebook包括输出的cell。

生成文档
===
&emsp;&emsp;该只需简单的在命令行执行nbdev_build_docs，就能生成文档。
```
$ nbdev_build_docs
converting: /home/jhoward/git/nbdev/nbs/00_core.ipynb
converting: /home/jhoward/git/nbdev/nbs/index.ipynb
```
在doc目录下可以看到，00_core.ipynb对应生成core.html,index.ipynb对应生成index.html

commit到github
===
&emsp;&emsp;这时，你可以使用git commit/git push将修改提交到github。实际在测试时，因为修改过的文件没有加入跟踪，所以在用了git add -A命令之后，才将所有修改提交到github。
依托了github的最新Actions功能，push之后，github会自动执行测试等任务。<br>
&emsp;&emsp;GitHub Actions 的主要作用就是让用户能够在 GitHub 服务器上直接执行和测试代码，只需几个简单步骤就可以实现构建、共享和执行代码。其实现了所谓的CI持续集成（CONTINUOUS INTEGRATION）的功能。在持续集成环境中，开发人员将会频繁的提交代码到主干。这些新提交在最终合并到主线之前，都需要通过编译和自动化测试流进行验证，而这一切在github都是免费的。
&emsp;&emsp;你可以进到github的项目首页，点击commit，将会看到每次提交的过程，包括测试是否成功等等。

参考
===
nbdev官方网站：[https://nbdev.fast.ai/ ]https://nbdev.fast.ai/<br>
nbdev: use Jupyter Notebooks for everything:[https://www.fast.ai/2019/12/02/nbdev/](https://www.fast.ai/2019/12/02/nbdev/)<br>
中文版：[https://tech.sina.com.cn/roll/2020-02-03/doc-iimxxste8470190.shtml](https://tech.sina.com.cn/roll/2020-02-03/doc-iimxxste8470190.shtml)<br>
