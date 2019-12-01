# Tigase快速开始指南

这篇博文并不完全翻译，因为是有关安装的教程。需要的请查看官方教程。

Note: Tigase快速开始指南。原文链接：[Quick Start Guide](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#about)

## 最低需求

在将Tigase服务安装到电脑之前，请确保你首先安装了**JDK8+**,Tigase推荐使用Oracle JDK,OpenJDK也许也可以运行正确，但是可能一些新的特性并不支持。

管理员权限，我们推荐你以管理员权限来安装Tigase Server。

## 具体内容

下面的的文章包含了一些关于快速使用或者开发Tigase Server的主题。请查看下边的列表找到你想要的列表。

## 使用GUI Installer安装

如果你不想手动安装Tigase服务，你可以使用GUI installer进行安装，这种方式不仅仅会复制文件到相应的地方，同时也会对重要的参数和数据库进行设置。因此，我们推荐使用这种方式进行安装。

### 安装前的准备

在你使用GUI installer进行安装前，你需要拥有Java工作环境。GUI installer仅仅只需要jre运行时环境，Tigase需要JDK环境。请注意，当前最小的JDK运行环境是1.6版本。在设置完JAVA_HOME环境之后，即可下载GUI installer进行安装。

### 下载GUI installer

你可以在[download section](https://projects.tigase.org/projects/tigase-server/files)中发现当前最新的安装包。当进入此页面后，你将会看到一个列表页面。你也许会困惑在那么多选择中如何选择。所有的Tigase二进制文件采用传统的包名命名。这种方式可以使你选择变得很容易**tigase-server-x.y.z-bv.ext**是包名的形式，其中'x', 'y', 'z' and 'v' 是版本名，他会因为从一个发布版本到另外一个发布版本而改变。Ext是文件拓展名。最后下载文件的文件后缀是**.jar**文件。我们推荐你下载安装最新的版本，因为它包含了最近的更新与改进。

### 运行jar包

正常情况下，在系统中安装完jdk后，系统会有一个默认配置，通过双击允许运行jar包。如果你使用这种方式无法安装，那么你需要进行如下：

- 如果你使用Windows系统，你可以使用命令行选项来进行安装。
- 选择menu菜单，并且选择run。

 使用Web Installer安装运行下面的命令进行安装

```shell
java -jar c:\download\tigase-server-4.1.0-b1315.jar
```

## 使用Web Installer安装

如果Tigase服务安装完成，他的默认配置文件在*etc/init.properties*。如果这个文件没有被修改，那么你可以运行web installer。它将一步步的对你的配置文件进行更改。如果你正在安装Windows上安装环境，please see the [Windows Installation](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#winWebInstall) section.

### 下载解压

首先下载解压Tigase XMPP Server。你可以下载 [official binaries](https://projects.tigase.org/projects/tigase-server/files)。如下:

```shell
$ wget http://build.xmpp-test.net/nightlies/dists/2015-01-12/tigase-server-7.0.0-SNAPSHOT-b3752-dist-max.tar.gz
$ tar -xf tigase-server-7.0.0-SNAPSHOT-b3752-dist-max.tar.gz
$ cd tigase-server-7.0.0-SNAPSHOT-b3752
```

请不要以root方式运行。

### 启动服务

```shell
scripts/tigase.sh start
```

### 验证Tigase运行状态

```bash
$ lsof -i -P
COMMAND   PID   USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
java    18387 tigase  141u  IPv6 22185825      0t0  TCP *:8080 (LISTEN)
java    18387 tigase  148u  IPv6 22185834      0t0  TCP *:5222 (LISTEN)
java    18387 tigase  149u  IPv6 22185835      0t0  TCP *:5223 (LISTEN)
java    18387 tigase  150u  IPv6 22185836      0t0  TCP *:5290 (LISTEN)
java    18387 tigase  151u  IPv6 22185837      0t0  TCP *:5280 (LISTEN)
java    18387 tigase  152u  IPv6 22185838      0t0  TCP *:5269 (LISTEN)
```

### 连接Web Installer

在连接之前有以下注意事项：

这个设置界面是受限访问的。Tigase的默认用户名是admin。密码是tigase

Web Installer将只被校验一次，然后在init.properies文件和进程中，安装器将被移除。如果需要使用Web Installer，将有将有一下几个条件:

- 拥有一个jid为管理员的账户，配置在init.properies.
- 管理员的用户名和密码需要手动添加到init.properies中，

根据ip打开相应的http://ip/setup/。

### Restart the Server

我们推荐在你的系统中手动的停止或者重启服务，在Tigase中根目录中使用下面的命令：

```shell
./scripts/tigase.sh stop

./scripts/{OS}/init.d/tigase start etc/tigase.conf
```

不管你的系统是Linux, gentoo, debian, mandriva或者redhat.

为了进一步调节你的服务，你应该编辑*/etc/tigase.conf*.确保*JAVA_HOME*路径正确，并且如果你希望减小内存需求。你应该需要用*JAVA_OPTIONS** -Xmx (max), and -Xms (initial)*。他会在系统被初始化的时候读取配置文件并进行加载。

## 使用Console Installer安装

### Installation Using the text-mode Installer

当你在桌面机器上安装Tigase server或者在一个拥有用户界面的服务机上安装，你可以使用上面的GUI Installer方式。但是当管理员使用ssh登录服务器时，上面的方式就不可行了。因为这种方式比较广泛和普遍。所以Tigase团队实现了利用console模式来安装服务。

注意：这种方式的安装仅仅适用于**Tigase server**4.1.5+。

#### 前提条件和重要的注意事项

在你尝试安装之前，请记住：

- 这个版本是第一个alpha版本，出现任何问题可以反馈给[Artur Hefczyc](mailto:artur.hefczyc@tigase.net) or Mateusz Fiolka.
- 安装之前，还需要配置JDK,**Java JDK v1.6 or newer**.

### 运行jar文件

首先下载你所需要的jar包，然后执行:

```she
java -jar nameOfTheDownloadedJarFile.jar -console
```

### 安装步骤

下面几个是命令行安装的注意要点：

- 记住JDK路径，后面配置的第一步需要。
- ​

## 手动命令行安装





## Windows 安装



## Tigase Server网络说明



## Tigase 脚本选择





##  