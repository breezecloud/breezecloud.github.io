layout: post
title:  "python调试工具Smiley使用手册"
date: 2019-3-12 08:15:00 +0800
categories: jekyll update
---
python调试工具Smiley使用手册
===
原文：https://smiley.readthedocs.io/en/latest/
Smiley is a tool for spying on your Python programs and recording their activities. It can be used for post-mortem debugging, performance analysis, or simply understanding what parts of a complex program are actually used in different code paths.
Smiley是一个工具，用于监视您的Python程序并记录它们的活动。它可以用于事后调试、性能分析，或者简单地理解复杂程序的实际执行的不同代码路径。

一、快速指南
1、安装
$ pip install smiley

2、使用
在一个终端下执行：(test.py是smiley软件包test_app目录下的示例)
$ smiley monitor
在另一个终端下执行：
smiley run ./test.py
smiley run --remote ./test.py
监视器会话将显示应用程序的执行路径和局部变量。

3、将参数传递给跟踪程序
$ smiley run -- ./test.py -e
--将命令序列与运行选项分开

二、命令参考
run
Run an application and trace its execution.

monitor
Listen for trace data from an application running under the run command.

record
Listen for trace data from an application running under the run command and write it to a database for later analysis.

list
Show the runs previously recorded in the database.

delete
Delete runs from the database.

replay
Given a single run id, dump the data from that run in the same format as the monitor command.

server
Run a web server for browsing the pre-recorded run data collected by the record command.

report
Export a set of HTML files describing the pre-recorded run data collected by the record command.

stats show
Show the profiling data from a run.

stats export
Dump the profiling data from a run to a local file.

help
Get help for the smiley command or a subcommand.

三、服务器模式
服务器命令在本地端口上启动Web服务器，为浏览记录和运行捕获的运行数据提供用户界面。它连接到同一个数据库，因此当捕获新的运行时，它们将出现在用户界面中。

1、Run List
服务器监听http://127.0.0.1:8080 默认情况下。在浏览器中访问该页面会使服务器以反向时间顺序返回数据库中的运行列表。对于每次运行，该列表显示其id、“Description”、开始和结束时间以及任何最终错误消息。单击最左边列中的“X”链接将永久删除从数据库中运行的链接。

2、运行详细信息
单击其中一个Run id值将打开为该运行记录的详细信息。详细信息页显示程序运行时的逐行状态，包括程序控件所在的位置(文件名、行号和源行)以及局部变量和函数的返回值。这是由监控器和重放，以一种更容易阅读的格式。

3、源文件
RunDetails视图中的每个文件名都链接到一个页面，该页面显示Python文件的完整来源，就像在程序执行时一样。

4、文件列表
对于具有多个源文件的应用程序，可以通过导航到文件列表视图并从列表中选择文件来检查源代码。

5、统计分析
stats视图显示运行的分析器输出，并按累积时间排序。与运行详细信息一样，每个文件名都链接到模块的完整源代码

6、调用图
调用图视图使用gprof2dot和图文生成一个树图，显示在程序的不同部分中使用了多少时间，以便更容易地将注意力集中在使用时间最多的区域上。

注
若要使此页面正常工作，dot命令必须已安装。安装smiley应该自动安装gprof2dot的。