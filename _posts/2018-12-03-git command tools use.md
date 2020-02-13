---
layout: post
title:  "git命令行工具使用简明手册v0.2"
date: 2018-12-03 08:15:00 +0800
categories: jekyll update
---
* [(一)介绍篇](#1)
   * [git是什么](#2)
   * [安装客户端](#3)
* [(二)基础篇](#4)
   * [最基本的Git命令](#5)
   * [如何进行回退](#6)
   * [操作远程仓库](#7)
* [(三)提高篇](#8)
   * [分支管理](#9)
   * [解决冲突](#10)
   * [stash](#11)
   * [多人协作](#12)
   * [Rebase](#13)
   * [分支管理策略](#14)
   * [使用标签](#15)
   * [其它技巧](#16)

<span id="1">(一)介绍篇</span>
=========
&emsp;&emsp;你如果没用过git，那你肯定不好意思说自己是一个软件开发人员。git自诞生迅速成为最流行的分布式版本控制系统。2008年GitHub网站的上线更是让git火得一塌糊涂！  
&emsp;&emsp;作为一个命令行爱好者，本文档介绍了基于命令行的git使用（主要在winodws环境下）。这篇文档是git的入门教程，主要参考了！[Git教程]（https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000） 如果想进一步了解git，推荐网上的《Git权威指南》，指南本身也是开源的，可以在https://github.com/jiangxin/docker-gotgit 找到。本教程也放在github上，可以在[个人网站](http://youery.cn/2018/12/git-command-tools-use)找到，由于个人水平有限，难免有疏忽和错误之处欢迎批评指正。

<span id="2">git是什么</span>
=========
&emsp;&emsp;git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。最初git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。git与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，这意味着它并不依赖于中心服务器来保存你文件的旧版本。任何一台机器都可以有一个本地版本的控制系统，其实就是一个硬盘上的文件，我们称之为仓库（repository）。如果是多人协作的话，你还需要一个线上仓库，用来同步代码等信息，支持多人协作，比如著名的GitHub等网站。当然，git是一个开源系统，你也可以自己搭建一个git服务器，用于公司内部的软件版本控制。

<span id="3">安装客户端</span>
=========
*  Linux  
大部分linux发行版应该已经包含了git客户端，如果你在命令行下执行git成功，那么说明你的机器上已经安装了git。如果没有安装，可以用发行版的软件包管理工具安装，如基于debain的发行版可以用apt-get install git安装。
*  Windows  
最初git是基于linux的命令行方式，和windows的shell有较大差异，最初在windows下模拟linux命令行的软件还有问题。不过随着软件的成熟，gitscm的windows下git客户端已经非常好用，可以到https://gitforwindows.org/下载安装，我们只要执行Git Bash程序就可以进入命令行方式。要注意在Git Bash里，本地硬盘,如C:\myfile文件，在gitscm命令行下写成/C/myfile。
基本配置及目录
```
git config --global user.name "My Name" 
git config --global user.email myEmail@example.com
```
安装好客户端之后，首先执行这两个命令来设置使用者名字和邮件，这代表了你在提交时候的身份。其实config命令修改了git的配置文件，Git相关的配置文件有三个
* windows在gitscm安装目录下gitconfig ，linux在/etc/gitconfig：包含了适用于系统所有用户和所有项目的值。
* windows在用户目录下.gitconfig ，linux在~/.gitconfig：只适用于当前登录用户的配置。
* 位于git项目目录中的.git/config：适用于特定git项目的配置。
对于同一配置项，三个配置文件的优先级是1<2<3

1. git init  
&emsp;&emsp;进入某个空的文件夹下，打开Git Bash命令窗口输入git init。命令主要用来初始化一个空的git本地仓库。执行完上面的命令，当前目录下会自动生成.git隐藏文件夹，该隐藏文件夹就是git版本库，下面讨论的所有对象都存储在这个目录里。初始化完成之后，你就可以在这个文件夹里创建文件进行管理了。

<span id="4">(二)基础篇</span>
=========
&emsp;&emsp;上一篇介绍了git的安装，基本配置命令和初始化一个本地库的命令。下面继续学习git的基本概念和操作。刚刚学习GIT时，似乎让人很迷惑，以至于误解，误用。百度学习了半天也不得要领。但是事实上不应该如此难以理解，只要你理解到这个命令究竟在干什么。首先我们来看几个术语：
* HEAD   
当前活跃分支的游标，也就是在当前分支你最近的一个提交，可以用 checkout 命令改变 HEAD 指向的位置。形象的记忆就是：你现在在哪儿，HEAD 就指向哪儿，所以 Git 才知道你在那儿！HEAD是git内置的定义好的特定含义功能，不可以修改。master，origin都是常用的公共命名方式，可以有自己的定义
* master  
首次创建仓库时默认分支的名字，在大多数情况下，master是指主干分支。那这个master到底是什么呢？其实它在.git目录下对应了一个引用文.git/refs/heads/master文件，而该文件的内容便是该分支中最新的一次提交的ID
*Origin  
默认的远程仓库的名字。
* Index  
index也被称为staging area或暂存区，是指一整套即将被下一个提交的文件集合，也可以理解是当前文件集的一个快照。
* Working Copy  
working copy代表你正在工作的那个文件集
&emsp;&emsp;另外在初始状态下，文件默认是不被git管理的，我们称之为未跟踪状态。Git下文件有三种状态，你的文件可能处于其中之一：已提交(committed)、已修改(modified)和已暂存(staged)。 已提交表示数据已经安全的保存在本地数据库中。 已修改表示修改了文件，但还没保存到数据库中。 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中，如下图：  
![](/images/2018-12-03-1-1.png)  


<span id="5">最基本的Git命令</span>
=========
1. git add 文件名  
&emsp;&emsp;如果文件没被跟踪，则文件加入工作目录，如果已经在工作目录并且修改过，则加入暂存。git add . 命令会将所有文件加入工作目录或暂存区。  
1. git status  
&emsp;&emsp;执行命令后会显示一个报告，如：
![](/images/2018-12-03-1-2.png)  
&emsp;&emsp;这份报告应该这样解读：Changes to bo committed就是如果要提交到本地库，需要做什么；Changes not staged for commit就是如果要加入暂存库，需要做什么。另外还贴心的提供了一些帮助。这样就比较容易理解显示的内容了。同时嫌太啰嗦的话可以执行：git status –s，文件前面的第一列对应Changes to bo committed，第二列对应Changes not staged for commit，M-修改 D-删除 A-第一次加入（本地库还没有），对应上图结果：
![](/images/2018-12-03-1-3.png)  
1. git commit  
&emsp;&emsp;暂存区域提交到本地库。会启动文本编辑器以便输入本次提交的说明。 (默认会启用 shell 的环境变量 $EDITOR 所指定的软件，一般都是 vim 或 emacs。当然也可以按照 起步 介绍的方式，使用 git config --global core.editor 命令设定你喜欢的编辑软件。）
&emsp;&emsp;如果不想打开编辑器直接提交可以用命令git commit –m “本次提交的说明”,也可以git commit –a 将修改的文件直接从工作目录提交的本地库（联同暂存库的修改）。为了便于理解，描述一下简单工作流程三部曲：
* 当你第一次checkout（签出）一个分支，HEAD就指向当前分支的最近一个commit。在working copy的文件集和HEAD,INDEX中的文件集是完全相同的。所有三者都是相同的状态，GIT很happy。此时执行git status没有任何报告。
* 当你对一个文件修改，Git感知到了这个修改，并且说：“嘿，文件已经变更了！你的working copy不再和INDEX区, HEAD相同！”，随后GIT标记这个文件是修改过的。此时执行git status会显示INDEX区需要修改。
* 然后，当你执行一个git add,它就将文件保存的暂存区，并且说：“嘿，OK，现在你的working copy和INDEX区是相同的，但是他们和HEAD区是不同的！” 此时执行git status会显示本地库需要修改
&emsp;&emsp;当你执行一个git commit,GIT就创建一个新的commit，随后HEAD就指向这个新的commit，而index,working copy的状态和HEAD就又完全匹配相同了，GIT又一次HAPPY了。

1. git diff 文件名  
&emsp;&emsp;执行git diff会报告当前文件和INDEX区之间的差异，如下图： 
![](/images/2018-12-03-1-4.png)  
&emsp;&emsp;什么？看不懂报告？可以学习一下diff命令，报告的格式是一样的。说实话我也看不懂，太晦涩了。其它用法如下：
* git diff --cached  ```[<path>...]``` 比较暂存区与最新本地版本库（本地库中最近一次commit的内容）
* git diff HEAD ```[<path>...]``` 比较工作区与最新本地版本库 
&emsp;&emsp;还有……，可以百度一下。

<span id="6">如何进行回退</span>
=========
&emsp;&emsp;既然是版本管理，当然可以回退。在你每次提交的时候，相当于所有的文件保存了一个快照，版本管理上叫做基线。一般可以根据某个基线进行版本发布，当然，如果发现某个提交的版本有问题，可以回退到这个版本查找问题。

1. git log  
&emsp;&emsp;进行多次提交之后，可以通过本命令查看提交历史。请注意上面的一窜十六进制数字。因为git是一个分布式系统，该串代号代表了一个提交，而且是全球唯一的数字，这样在以后提交到远程服务器时不会有冲突。在下面的回退命令中，可以输入该数字（前几位），就可以代表回退到该版本。
![](/images/2018-12-03-1-5.png)  
1. git checkout –– 文件（场景一）  
&emsp;&emsp;撤销本地改动代码，就是把文件在工作区的修改全部撤销，如果修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；如果添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
1. git reset HEAD文件（场景二）  
&emsp;&emsp;可以把暂存区的修改撤销掉（unstage），重新放回工作区。用HEAD时，表示最新的版本。当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD ```<file>```，就回到了场景1，第二步按场景1操作。
1. git reset HEAD^ （场景三）  
&emsp;&emsp;已经提交了不合适的修改到版本库时，想要撤销本次提交，只能采用版本回退，不过前提是没有推送到远程库。命令中上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100，版本号也可以直接写十六进制的编号。	Reset命令三个参数的含义：
* git reset --mixed：此为默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息
* git reset --soft：回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可
* git reset  --hard：彻底回退到某个版本，本地的源码也会变为上一个版本的内容，此命令（从本地库回复），所有可能会丢失部分工作，请慎用！
&emsp;&emsp;总结一下上面讲到的提交和回退命令：
![](/images/2018-12-03-1-6.png)  


<span id="7">操作远程仓库</span>
=========
&emsp;&emsp;基础篇可以让我们方便的在本机管理自己的文件。但git更强大的功能在于多人协作，所有必须可以让本地的仓库和远程仓库进行上传、下载、合并等，这样别人也可以看到你的代码并进行修改。
1. git remote –v  
&emsp;&emsp;查看远程仓库，看起来像这样：  
origin	https://github.com/schacon/ticgit (fetch)  
origin	https://github.com/schacon/ticgit (push)  
&emsp;&emsp;显而易见前面是简称，后面是权限。  
1. git remote add ```<shortname> <url>```     
&emsp;&emsp;添加一个新的远程 Git 仓库，同时指定一个你可以轻松引用的简写：
1. git fetch ```<url>```  
&emsp;&emsp;git remote add pb https://github.com/paulboone/ticgit，之后你想拉取 Paul 的仓库中有但你没有的信息，可以运行 git fetch pb。现在 Paul 的 master 分支可以在本地通过 pb/master 访问到。这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。
1. git clone ```<url>```  
 &emsp;&emsp;clone 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。 所以，git fetch origin 会抓取克隆（或上一次抓取）后新推送的所有工作。 必须注意 git fetch 命令会将数据拉取到你的本地仓库 - 它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。
1. git pull  
&emsp;&emsp;命令来自动的抓取然后合并远程分支到当前分支。 这对你来说可能是一个更简单或更舒服的工作流程；默认情况下，git clone 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 master 分支（或不管是什么名字的默认分支）。 运行 git pull 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。
1. git push [remote-name] [branch-name]  
&emsp;&emsp;当你想分享你的项目时，必须将其推送到上游。 当你想要将 master 分支推送到 origin 服务器时（再次说明，克隆时通常会自动帮你设置好那两个名字），那么运行这个命令就可以将你所做的备份到服务器：
1. git push origin master  
&emsp;&emsp;只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 你必须先将他们的工作拉取下来并将其合并进你的工作后才能推送。
1. git remote show [remote-name] / git remote show origin  
&emsp;&emsp;remote show origin
 ```
 Fetch URL: https://github.com/schacon/ticgit
  Push  URL: https://github.com/schacon/ticgit
  HEAD branch: master
  Remote branches:
    master                               tracked
    dev-branch                           tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```
&emsp;&emsp;它同样会列出远程仓库的 URL 与跟踪分支的信息。 这些信息非常有用，它告诉你正处于 master 分支，并且如果运行 git pull，就会抓取所有的远程引用，然后将远程 master 分支合并到本地 master 分支。 它也会列出拉取到的所有远程引用。
1. git remote rename  
&emsp;&emsp;如果想要重命名引用的名字可以运行 git remote rename 去修改一个远程仓库的简写名。 例如，想要将 pb 重命名为 paul，可以用 git remote rename pb paul这样做。值得注意的是这同样也会修改你的远程分支名字。 那些过去引用 pb/master 的现在会引用 paul/master。
1. git remote rm  
&emsp;&emsp;如果因为一些原因想要移除一个远程仓库，如你已经从服务器上搬走了或不再想使用某一个特定的镜像了，又或者某一个贡献者不再贡献了 - 可以使用git remote rm paul。
最后总结一下git各种操作，如下图。
![](/images/2018-12-03-1-7.png)  

<span id="8">(三)提高篇</span>
=========
&emsp;&emsp;git的强大体现在分布式的架构和高效率的分支功能，支持从简单到复杂的项目。提高篇介绍的功能可以应付复杂项目和多人协作，学会了这些功能，复杂项目的代码管理你也能轻松驾驭。

<span id="9">分支管理</span>
====
分支就是你从开发主线上创建一个分支，然后在不影响主线的同时继续提交代码。到开发完毕后，再一次性合并到原来的A主线分支上，这样既安全，又不影响别人工作。
1. git checkout -b dev  
    创建分支dev,命令加上-b参数表示创建并切换，相当于以下两条命令
```
$ git branch dev
$ git checkout dev
```
此时状态如图：  
![](/images/2018-12-03-1-8.png)  
2. git branch 查看分支  
此时对当前工作区文件进行修改并提交。
此时状态如图：  
![](/images/2018-12-03-1-9.png)  
现在，dev分支的工作完成，我们就可以切换回master分支：  
```
git checkout master
```
此时状态如图：  
![](/images/2018-12-03-1-10.png)  
3. git merge 合并分支  
现在，我们把dev分支的工作成果合并到master分支上
```
$ git merge dev
Updating d46f35e..b17d20e
Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```
此时状态如图：  
   ![](/images/2018-12-03-1-11.png)  
4. git branch -d 删除分支  
```
$ git branch -d dev
Deleted branch dev (was b17d20e).
```
这时可以放心的删除dev分支了。但假如分支还没有被合并，如果删除，将丢失掉修改（此时git会报错），如果要强行删除，需要使用大写的-D参数：
```
$ git branch -D dev
Deleted branch dev (was 287773e).
```

5. 分支策略  
  通常，合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。
```
$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```
此时状态如图：  
   ![](/images/2018-12-03-1-12.png)   

<span id="10">解决冲突</span>
====
&emsp;&emsp;当不同的分支对同一个文件进行了修改，这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，就必须首先解决冲突，再提交完成合并。
&emsp;&emsp;比如feature1和master分支的readme.txt有冲突，当合并到master分支时：
```
$ git merge feature1
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
```
  可以用git status显示合并的结果：
```
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

    both modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
打开readme.txt文件，发现Git用```<<<<<<<，=======，>>>>>>>```标记出不同分支的内容。我们修改如下后保存之后再次提交
```
$ git add readme.txt 
$ git commit -m "conflict fixed"
[master cf810e4] conflict fixed
```
此时状态如图：  
![](/images/2018-12-03-1-13.png)   
用带参数的git log也可以看到分支的合并情况，最后，删除feature1分支。
```
$ git branch -d feature1
Deleted branch feature1 (was 14096d0).
```
小结：当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。  
用git log --graph命令可以看到分支合并图。

<span id="11">stash</span>
====
&emsp;&emsp;当你修复一个bug的。很自然地，你想创建一个分支来修复它。但是，当前正在dev上进行的工作还没有提交，这时怎么办？幸好，Git还提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：  
```
$ git stash
Saved working directory and index state WIP on dev: f52c633 add merge
```
&emsp;&emsp;现在，用git status查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支。修复完成后，切换到master分支，并完成合并，最后删除临时分支。
&emsp;&emsp;切换回dev分支，执行```git status```，发现工作区是干净的，刚才的工作现场存到哪去了？用git stash list命令看看：
```
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```
&emsp;&emsp;工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：
* 用```git stash appl```y恢复，但是恢复后，stash内容并不删除，你需要用```git stash drop```来删除；
* 用```git stash pop```，恢复的同时把stash内容也删了  
&emsp;&emsp;你可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash，用命令：
```
git stash apply stash@{0}
```

<span id="12">多人协作</span>
====
1. 推送分支
推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：
```
$ git push origin dev
```
2. 抓取分支
多人协作时，大家都会往master和dev分支上推送各自的修改。当你的小伙伴从远程库clone时，默认情况下，你的小伙伴只能看到本地的master分支。现在，你的小伙伴要在dev分支上开发，就必须创建远程origin的dev分支到本地，于是他用这个命令创建本地dev分支：
```
$ git checkout -b dev origin/dev
```
现在，他就可以在dev上继续修改，然后，时不时地把dev分支push到远程：
```
$ git push origin dev
```
&emsp;&emsp;当然，此时也可能会引起冲突，一样也需要解决冲突之后再重新push，因此，多人协作的工作模式通常是这样：
   1. 首先，可以试图用```git push origin <branch-name>```推送自己的修改；
   1. 如果推送失败，则因为远程分支比你的本地更新，需要先用```git pull```试图合并；
   1. 如果合并有冲突，则解决冲突，并在本地提交；
   1. 没有冲突或者解决掉冲突后，再用```git push origin <branch-name>```推送就能成功！
   1. 如果```git pull```提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令```git branch --set-upstream-to <branch-name> origin/<branch-name>```。  

<span id="13">Rebase</span>
====
&emsp;&emsp;git rebase用于把一个分支的修改合并到当前分支，合理使用rebase命令可以使我们的提交历史干净、简洁！假设你现在基于远程分支"origin"，创建一个叫"mywork"的分支。
```
$ git checkout -b mywork origin
```
假设远程分支"origin"已经有了2个提交，如图  
![](/images/2018-12-03-1-14.jpg)  
现在我们在这个分支做一些修改，然后生成两个提交(commit),但是与此同时，有些人也在"origin"分支上做了一些修改并且做了提交了. 这就意味着"origin"和"mywork"这两个分支各自"前进"了，它们之间"分叉"了,如图：  
![](/images/2018-12-03-1-15.jpg)   
在这里，你可以用"pull"命令把"origin"分支上的修改拉下来并且和你的修改合并； 结果看起来就像一个新的"合并的提交"(merge commit):
![](/images/2018-12-03-1-16.jpg)  
但是，如果你想让"mywork"分支历史看起来像没有经过任何合并一样，你也许可以用 git rebase:
```
$ git checkout mywork
$ git rebase origin
```
&emsp;&emsp;这些命令会把你的"mywork"分支里的每个提交(commit)取消掉，并且把它们临时 保存为补丁(patch)(这些补丁放到".git/rebase"目录中),然后把"mywork"分支更新 为最新的"origin"分支，最后把保存的这些补丁应用到"mywork"分支上。  
![](/images/2018-12-03-1-17.jpg)  
&emsp;&emsp;在rebase的过程中，也许会出现冲突(conflict). 在这种情况，Git会停止rebase并会让你去解决 冲突；在解决完冲突后，用"git-add"命令去更新这些内容的索引(index), 然后，你无需执行 git-commit,只要执行:
```
$ git rebase --continue
```
&emsp;&emsp;这样git会继续应用(apply)余下的补丁。在任何时候，你可以用--abort参数来终止rebase的行动，并且"mywork" 分支会回到rebase开始前的状态。
```
$ git rebase --abort
```
&emsp;&emsp;rebase操作可以把本地未push的分叉提交历史整理成直线,同时rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。

<span id="14">分支管理策略</span>
====
&emsp;&emsp;实际开发中，我们应该按照几个基本原则进行分支管理。  
1. 首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；那在哪干活呢？干活都在dev分支上。到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；
2. 你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。
3. 添加一个新功能时，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。
4. 并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？  
   * master分支是主分支，因此要时刻与远程同步；
   * dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
   * bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
   * feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

<span id="15">使用标签</span>
====
&emsp;&emsp;发布一个版本时，我们通常先在版本库中打一个标签（tag），这样就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。
所以，标签也是版本库的一个快照。Git 的标签虽然是版本库的快照，但其实它就是指向某个 commit 的指针（跟分支很像对不对？但是分支可以移动，标签不能移动）。  
&emsp;&emsp;Git有commit，为什么还要引入tag？比如有一个commit号是6a5819e...这样的一窜数字，不好记忆和交流，所以，tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起。  
在Git中打标签非常简单，首先，切换到需要打标签的分支上：
```
$ git branch
* dev
  master
$ git checkout master
Switched to branch 'master'
```
&emsp;&emsp;然后，敲命令git tag <name>就可以打一个新标签：
```
$ git tag v1.0
```
&emsp;&emsp;可以用命令git tag查看所有标签：
```
$ git tag
v1.0
```
&emsp;&emsp;可以找到历史提交的commit id，打标签。比方说要对应的commit id是f52c633，敲入命令：
```
$ git tag v0.9 f52c633
```
&emsp;&emsp;再用命令git tag查看标签：
```
$ git tag
v0.9
v1.0
```
&emsp;&emsp;注意，标签不是按时间顺序列出，而是按字母排序的。可以用```git show <tagname>```查看标签信息。还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：
```
$ git tag -a v0.1 -m "version 0.1 released" 1094adb
```
&emsp;&emsp;如果标签打错了，也可以删除：
```
$ git tag -d v0.1
Deleted tag 'v0.1' (was f15b0dd)
```
&emsp;&emsp;如果要推送某个标签到远程，使用命令```git push origin <tagname>```：
```
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
 * [new tag]         v1.0 -> v1.0
 ```
&emsp;&emsp;或者，一次性推送全部尚未推送到远程的本地标签：
```
$ git push origin --tags
Total 0 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
 * [new tag]         v0.9 -> v0.9
 ```
&emsp;&emsp;如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：
```
$ git tag -d v0.9
Deleted tag 'v0.9' (was f52c633)
```
&emsp;&emsp;然后，从远程删除。删除命令也是push，但是格式如下：
```
$ git push origin :refs/tags/v0.9
To github.com:michaelliao/learngit.git
 - [deleted]         v0.9
```

<span id="16">其它技巧</span>
====
1. 忽略特殊文件  
在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。不需要从头写.gitignore文件，GitHub已经为我们准备了各种配置文件，只需要组合一下就可以使用了。所有配置文件可以直接在线浏览：https://github.com/github/gitignore
2. 有没有经常敲错命令？比如git status？status这个单词真心不好记。如果敲```git st```就表示```git status```那就简单多了，当然这种偷懒的办法我们是极力赞成的。我们只需要敲一行命令，告诉Git，以后st就表示status：
```
$ git config --global alias.st status
```
当然还有别的命令可以简写，很多人都用co表示checkout，ci表示commit，br表示branch：
3. 修改缺省编辑器  
再有些命令git会启动一个vim编辑器，不过vim实在太难用了，我自己习惯用sublime，执行：
```
git config –global core.editor ‘C:\\Program Files\\Sublime Text 3\\sublime_text.exe’
```
这样git会自动启动sublime。你可以修改成自己习惯的编辑器。如果想再命令行中方便使用sublime，回到用户目录（cd命令），打开.bashrc文件，加上一行
```
alias sublime="/c/Program\ Files/Sublime\ Text\ 3/sublime_text.exe"
```
再命令行中收入sublime就可以打开sublime编辑器了。
（完）

&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin.jpg)