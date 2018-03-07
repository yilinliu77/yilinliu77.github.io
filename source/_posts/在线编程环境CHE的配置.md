---
title: 京东云服务器及Eclipse Che的配置
date: 2017-09-27 12:00:15
tags:
---

`Eclipse Che`是一个动态配置的按需IDE(集成开发环境)，它运行在云端提供可替代JetBrains IntelliJ Java IDE 的开源环境，并且允许在任何机器上打开浏览器写代码。

为了能让同学们独立地在自己的服务器上使用这个开发环境，我们将`Che`的基本环境已经在服务器上搭建好，现在还需要同学们完成最后一步将自己的服务器的公网IP输入到配置文件里面去

# 目标
- 完成在线开发环境的搭建
- 熟悉常见的服务器连接操作

# 步骤
## 京东云 创建实例教程
>by 熊鑫

1. 登录京东云的[官网](http://www.jcloud.com/)

2. 登录账号

3. 点击创建云主机的链接**(链接请咨询助教)**，创建主机

   选择官方镜像 ubuntu 16.04 64位![photo](http://owzhs7l26.bkt.clouddn.com/17-9-28/95709251.jpg)

   VPC选择默认选项就行，其他都不用设置

   ![pic](http://owzhs7l26.bkt.clouddn.com/17-9-28/80146896.jpg)

   立即购买就行了

4. 接下来进入控制台界面，登云主机创建好以后，先停止云主机

    ![tws-che](http://otbwgn2nv.bkt.clouddn.com/7645304aa94b0c2240f4ee8cb6cc7d93.png)

    ![tws-che](http://otbwgn2nv.bkt.clouddn.com/7f48906fa73b0e9b7cc316e4f9531fee.png)

   停止以后，在更多菜单中选择重置系统，

   ![](http://owzhs7l26.bkt.clouddn.com/17-9-28/87628379.jpg)

   点击更换镜像，镜像类型为共享，选择共享镜像为tws-eclipse-che-v1 Ubuntu 16.04，如上图所示。

   密码自己设置自己的密码，记住就行，然后点击确定。然后会提示是否重置，确定就行。

   然后就是等待系统重置完成。

   系统重置完成之后是停止状态，再启动就行了。

5. 云主机实例创建完成。

6. 查看自己的公网IP地址

    ![tws-che](http://otbwgn2nv.bkt.clouddn.com/7645304aa94b0c2240f4ee8cb6cc7d93.png)

## 修改CHE配置
>by 刘奕林

### 需要工具
- **Xshell**（一款远程连接服务器进行管理的软件）·[教程](http://jingyan.baidu.com/article/295430f13fb4db0c7f005065.html)

    也可以用Putty

- **自己的公网IP**



### 修改配置文件
首先需要连接到自己的服务器修改`che.env`文件，将里面的IP地址设为自己服务器的公网IP地址，按照如下步骤
- 打开`Xshell`，选择`新建站点`，输入服务器的公网IP并连接，并输入账号密码，默认进入`~`目录

    ![tws-che教程](http://otbwgn2nv.bkt.clouddn.com/5bfc64ee548e669975726924884905fc.png)

- 从登入服务器的`~`目录进入配置文件所在目录

    `cd EclipseChe`

    `ls -l`

    可以看到如下输出
    ```
    drwxr-xr-x 2 root root  4096 Sep 26 18:49 backup
    -rw-r--r-- 1 root root 24015 Sep 26 23:30 chebak.env
    -rw-r--r-- 1 root root 24015 Sep 28 13:59 che.env
    -rw-r--r-- 1 root root 52382 Sep 28 13:59 cli.log
    drwxr-xr-x 2 root root  4096 Sep 26 18:52 docs
    -rwxr-xr-x 1 root root   385 Sep 27 13:39 init-che.sh
    drwxr-xr-x 6 root root  4096 Sep 26 18:52 instance
    ```
    其中`che.env`是我们需要修改的文件

- 输入`./init-che.sh`运行脚本，会被问及自己的公网IP，输入自己的公网IP后回车
    ```
    $ ./init.sh
    请输入你的公网IP
    111.222.333.444(你的公网IP)
    ```

- 静静的等待`che`重启成功，大概五分钟左右

### 添加workspace
第二步我们需要添加自己的workspace，打开浏览器，输入自己的公网`IP地址:8080`访问`che`的管理界面,例如`111.222.333.444:8080`

- 在左边的菜单栏选择`WorkSpaces`

- 单击`Add WorkSpace`

- 在`SELECT STACK`下，将滚动条拉到最下面选择本次训练营主要用到的`tws-node-7`这一个环境

![tes-workspace](http://otbwgn2nv.bkt.clouddn.com/b8fd38c5f6f1ead157575385a7b1a7e4.png)

- 单击网页下侧的绿色按钮`CREATE`完成配置

然后选择左侧的新建的workspace就可以开始体验在线编程环境了~
