---
title: 21.docker+jenkins自动部署前端代码
date: 2018-09-25 23:12:50
tags:
---

> 服务器为腾讯云Ubuntu

### 一、安装Docker

1. 更新`apt`包
    ```bash
    $ sudo apt update
    ```

2. 安装最新版本的Docker

    中文版教程： [如何在Ubuntu上安装使用Docker](https://cloud.tencent.com/developer/article/1167995)
    
    不过最好还是看英文版：[install-using-the-repository](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-repository)
    
    或者使用官方Ubuntu存储库中提供的Docker安装包，但是可能不是最新的版本。
    ```bash
    # 使用内置安装
    $ sudo apt-get install docker.io
    # 卸载
    $ sudo apt-get purge docker.io
    ```

### 二、安装Jenkins

使用docker安装Jenkins非常方便，只需要执行下面命令即可。
```bash
docker run --name devops-jenkins --user=root -p 8080:8080 -p 50000:50000 -v /opt/data/jenkins_home:/var/jenkins_home -d jenkins/jenkins:lts
```
更多定制参数可以参考 https://github.com/jenkinsci/docker

### 三、配置Jenkins

启动Jenkins之后，在浏览器输入http://部署jenkins主机IP:端口 (8080)
![image](../images/21/1.png)

根据提示，从输入administrator password 或者可以通过启动日志
```bash
docker logs devops-jenkins
```
查看这个password
![image](../images/21/2.png)

选择安装插件方式，这里默认第一个。
<!-- ![image](../images/21/3.png) -->

进入插件安装界面，连网等待插件安装。

安装完插件后，进入创建管理员界面。
![image](../images/21/5.png)

输入管理员信息后，点击Continue as admin。

![image](../images/21/6.png)
保存Jenkins URL后Jenkins就安装完毕了。

### 四、自动化流程

在正式使用Jenkins前，我们先来看一下自动部署的流程，具体如下：

- 编写修改代码，并提交代码到代码仓库（git）
- Jenkins自动检测到仓库代码更新了，从仓库拉去最新代码
- Jenkins自动编译
- 

Jenkins之所以流行，是因为它的高灵活性和高易用性。这得益于它的插件机制，只需安装对应功能的插件，即可使它适应各种业务场景。
