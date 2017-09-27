---
title: 在线编程环境CHE的配置
date: 2017-09-27 12:00:15
tags:
---

`Eclipse Che`是一个动态配置的按需IDE(集成开发环境)，它运行在云端提供可替代JetBrains IntelliJ Java IDE 的开源环境，并且允许在任何机器上打开浏览器写代码。

为了能让同学们独立地在自己的服务器上使用这个开发环境，我们将`Che`的基本环境已经在服务器上搭建好，现在还需要同学们完成最后一步将自己的服务器的公网IP输入到配置文件里面去

# 目标
- 完成在线开发环境的搭建
- 熟悉常见的服务器连接及文件上传操作

# 修改CHE配置
**需要工具**
- **Xshell**（一款远程连接服务器进行管理的软件）·[教程](http://jingyan.baidu.com/article/295430f13fb4db0c7f005065.html)

    也可以用Putty+Winscp
- **配置脚本**·[下载](http://otbwgn2nv.bkt.clouddn.com/changeHostIP.sh)

## 修改配置文件
首先需要连接到自己的服务器修改`che.env`文件，将里面的IP地址设为自己服务器的公网IP地址，按照如下步骤
- 打开`Xshell`，选择`新建站点`，输入服务器的公网IP并连接，并输入账号密码，默认进入`~`目录

    ![tws-che教程](http://otbwgn2nv.bkt.clouddn.com/5bfc64ee548e669975726924884905fc.png)

- 从登入服务器的`~`目录进入配置文件所在目录

    `cd EclipseChe`

    `ls -l`

    可以看到如下输出
    ```
    drwxr-xr-x 2 root root  4096 Sep 26 18:49 backup
    -rw-r--r-- 1 root root 24015 Sep 26 19:42 che.env
    -rw-r--r-- 1 root root 14192 Sep 26 20:02 cli.log
    drwxr-xr-x 2 root root  4096 Sep 26 18:52 docs
    drwxr-xr-x 6 root root  4096 Sep 26 18:52 instance
    ```
    其中`che.env`是我们需要修改的文件

- 为方便同学更改文件，我们已经将更改操作编辑为脚本，点击下载[脚本](http://otbwgn2nv.bkt.clouddn.com/changeHostIP.sh)

- 打开`Xshell`，点击新建文件传输，进入到`~/EclipseChe/`目录下，输入`put`命令，在弹出的对话框选择下载到本地的脚本，将他上传至`~/EclipseChe/`目录下

    ![tws-配置教程](http://otbwgn2nv.bkt.clouddn.com/9529022cd0b7eff9cbe13994f5e02612.png)

- 在`Xshell`中切换到刚刚的ssh的窗口查看脚本并输入`chmod +x changeHostIP.sh`赋予脚本执行权限

- 输入`./changeHostIP.sh`运行脚本，会被问及自己的公网IP，输入自己的公网IP后回车
    ```
    $ ./test.sh
    请输入你的公网IP
    111.222.333.444
    ```

- 静静的等待`che`重启成功，大概五分钟左右

## 添加workspace
第二步我们需要添加自己的workspace，打开浏览器，输入自己的公网`IP地址:8080`访问`che`的管理界面,例如`111.222.333.444:8080`

- 在左边的菜单栏选择`WorkSpaces`

- 单击`Add WorkSpace`

- 在`SELECT STACK`下，将滚动条拉到最下面选择本次训练营主要用到的`tws-node-7`这一个环境

![tes-workspace](http://otbwgn2nv.bkt.clouddn.com/b8fd38c5f6f1ead157575385a7b1a7e4.png)

- 单击网页下侧的绿色按钮`CREATE`完成配置

然后选择左侧的新建的workspace就可以开始体验在线编程环境了~
