---
title: 从Seafile到Jenkins
date: 2017-03-16 10:32:45
tags: Jenkins
categories: 后端

---

![20170315254652017-03-16.jpg](http://7xk0q3.com1.z0.glb.clouddn.com/20170315254652017-03-16.jpg)

因为项目需要，又重新搭建了一次Seafile和Jenkins,Seafile是个人的私有云盘。之前搭建过一次，因为服务器太贵，一气之下全部卸了。这次搭建更加顺手，但是还是没能快速搭建和部署，还是花了半天。所以记录一下，下次再次搭建的时候，如果忘了直接照着做就ok。节约时间。Jenkins是自动化测试，部署，打包工具，两者都使得工作或者生活更方便，安全。这篇文章实在Centos7下进行搭建。Debian系列也没什么差别，上车。

<!--more-->

## Jenkins搭建CI环境

[Jenkins](https://jenkins.io/)是开源软件，并且更新很快。对比较流行的`git`仓库,如果**Github,Gitlab,Bitbuck**等等都有良好的支持，对项目构建工具**Maven,Gradle,ant**等等也支持的很好。包括邮件提醒，XMPP机器人集成，现在还包含丁丁集成。算是比较完善了。而且Jenkins的主题图标也是我喜欢的风格。来欣赏一波。

![2017031590034Screen Shot 2017-03-15 at 11.01.31 PM.png]( )

好了，不扯犊子了。首先我们检查，更新一下源。

### 更新源

```shell
sudo yum install epel-release
sudo yum update
```

### 安装Java8

- 下载

```shel
wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u102-b14/jdk-8u102-linux-x64.rpm
```

- 安装

```shell
sudo yum localinstall jdk-8u102-linux-x64.rpm
```
下载JDK的位置在`/usr/java/jdk1.8.0_102`

- 配置

打开`/etc/profile`
`sudo vim /etc/profile`
```shell
export JAVA_HOME=/usr/java/jdk1.8.0_102/
export JRE_HOME=/usr/java/jdk1.8.0_102/jre
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export PATH
```
然后`source /etc/profile`，测试一下`java -version`。

### 安装Jenkins

-  下载

```shell
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
```

- 安装 

```shell
sudo rpm --import http://pkg.jenkins-ci.org/redhat-stable/jenkins-ci.org.key
sudo yum install jenkins
```

- 运行

```shell
sudo systemctl start jenkins.service
sudo systemctl enable jenkins.service
```

### Nginx安装配置

- 安装

找到server，在下边再添加一个server，如下，因为Jenkins默认开启端口是8080，所以配置如下:

```shell
sudo yum install nginx
```

- 配置

```shell
sudo vi /etc/nginx/nginx.conf
```

```shell
server {
    listen       80;
    server_name  jenkins.jiangtao.tech;

        location / {
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://127.0.0.1:8080;
        }
}
```

好了，用域名访问吧。

打开网页后有个初始密码，用vim打开文件进入，创建用户。最后进入主页，如下:

![2017031552461Screen Shot 2017-03-15 at 11.17.00 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017031552461Screen Shot 2017-03-15 at 11.17.00 PM.png)

我使用的是Gitlab+Jenkins进行动态部署，都比较简单。

### 安装一波Maven

```shell
wget http://www-eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar xzf apache-maven-3.3.9-bin.tar.gz
sudo mv apache-maven-3.3.9 /usr/local/
sudo vim /etc/profile
export M2_HOME=/usr/local/apache-maven-3.3.9
export PATH=${M2_HOME}/bin:${PATH}
# 检查一下
mvn -version 
```



### 安装一波Git

```shell
sudo yum install git
git config --global user.name "xxx"
git config --global user.email "xxx@gmail.com"
ssh-keygen -t rsa -C "xxx@gmail.com"
```

一路回车，最后将`.ssh`下面的`.pub`文件复制到Git仓库配置。

剩下的就自己去折腾吧。一定要记得，使用Maven构建的时候千万不要加mvn，因为插件默认已经给你加了。



## Seafile

关于Seafile,相比之下就简单了好多，可以直接看[官方文档](https://manual-cn.seafile.com/deploy/using_mysql.html)进行安装。

事情太多了，但每天一篇博客还是要写的。就当记笔记吧。前端的东西又是一天没动了。实在太忙。