---
title: 搭建Git服务器及多人协作开发常见命令
date: 2017-08-10 18:35:59
tags:
- Git
- 分支策略
categories: Git
---
# 搭建Git服务器
Github的项目必须为开放的并且在中国的访问速度如无特殊配置通常访问很慢，于是便想在自己的服务器上搭建Git仓库，同时也具备多人协作开发的能力。
## 配置Git和用户
首先安装Git

`sudo apt install git`

配置单独用来访问仓库的git用户

```
groupadd git
adduser git -g git
```

## 创建SSL证书登录
在支持bash的命令行下，输入

`ssh-keygen -C 'your@email.com' -t rsa `

为你生成rsa密钥，可以直接一路回车，执行默认操作，生成的密钥可以在这儿找到
```
C://users//<电脑账户名>//.ssh
├── id_rsa
└── id_rsa.pub #公钥 服务端需要里边内容验证连接着身份
```

将SSH添加到管理

`ssh-add id_rsa`

将公钥里面的内容完整输入至`/home/git/.ssh/authorized_keys`，一行一个密钥

## Tips：多公钥管理
在`.ssh`目录下配置`config`文件，**再将SSH加入SSH管理**，格式如下：
```
Host github.com             //连接域名
  HostName github.com       //域名名称
  User whatseven            //用户
  IdentityFile C:\Users\whatseven\.ssh\id_rsa_github    //对应私钥
  PreferredAuthentications publickey
```

# Git多人协作开发常用命令
完成上述步骤，就能够在自己的服务器上建立仓库并且作为远程分支与本地分支建立联系了，下面是在多人协作开发的一些常用的Git命令。

## 建立裸仓库

在建好的服务器里新建文件夹，在文件夹下使用`git init --bare`新建裸仓库，只存放版本库信息。

开发人员A上用`git clone 仓库地址 仓库在本机的命名`克隆仓库，开始自己的本地开发。


### Tips：为什么要建立裸仓库？

在Git中，如果向普通代码仓库push的话，Git会将推送的内容与工作文件进行比较，它会认为工作文件发生改变，从而影响工作树，常见的是在想远程目标的当前分支push代码时，在远程终端必须使用`git reset --hard`。而裸代码仓库由于没有工作树，所以push所含的变化仅影响裸代码仓库的版本控制。

**在裸仓库中，工作目录下除了版本库信息没有任何东西，会给大家造成一个误解以为裸仓库只记录版本，没有文件。其实在Git系统中，文件是通过不同的对象例如索引，树，块存与版本库里面的，所以裸仓库看似工作目录是空的，在克隆裸仓库是git会将对象从版本库取出并根据版本更改信息组织到正确的位置**


在开发过程中，一般会将项目分为几个分支。
> **Git分支策略**

> ![Git分支策略](http://otbwgn2nv.bkt.clouddn.com/de249adc849a5e3ed721b3f4fc397e4d.png)

1. **master**分支应该保存重大的版本或节点，开发前应保持远程与本地同步
2. **dev**是日常开发分支，平时的开发应在dev上进行，开发前应保持远程与本地同步
3. 剩下的是个人分支，个人的开发在个人分支上进行，在每天或者一个阶段的工作结束后每个人将自己的代码合并到dev分支上。个人分支不用同步至远程分支

## 开发实例

下面通过两个开发人员实例演示多人开发：

开发人员A通过`git clone git@111.111.111.111:/home/git/repo/test.git test.git`将远程的版本库克隆至本机

如无dev分支，A通过`git checkout -b dev`从**master**分支创建**dev**分支

A再次通过`git checkout -b A`从**dev**分支上创建自己的开发分支

A在工作目录里新建内容为"A"的**A.txt**，再创建内容为"公共"的**公共.txt**

A通过`git add A.txt 公共.txt`与`git commit -m "init A,公共"`提交更改

当A完成阶段性工作后，通过`git checkout dev`切换至开发分支，再通过`git merge --no-ff A`将**A**分支合并到**dev**分支,如有冲突则在文件中去删除多余部分

A剩下的工作是要将自己的工作推到大家代码的源头--位于远程的裸仓库中，在Git中，本机Git版本库与远程库的连接是基于分支的，例如本机的**master**与远程的**master**有一条链接，本机的**dev**也应该与远程**dev**有一个链接。

A通过在**dev**分支上`git push -u origin dev`将本机的**dev**分支的更新推送到远程**dev**分支上去，`-u`选项是指在未建立链接的情况下建立一条链接，以后推送更新就只用`git push`即可

此时，A的工作已经全部做完了，当B开始工作时，他必须获取远程库的最新情况以保证自己的工作是建立在最新的工作上的，它可以在建立了链接的情况下通过`git pull`将远程更新拉到本地，后续操作类似A。
