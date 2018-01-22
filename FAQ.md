# FAQ

**问：如何在CentOS 7下安装python-pip?**

**答：** 直接执行`yum install python-pip`安装，如果没有python-pip包，则执行命令`yum -y install epel-release`，再次执行`yum install python-pip`进行安装，最后执行`pip install --upgrade pip`进行升级。

**问：如何安装docker-compose工具？**

**答：** 在安装docker-compose工具之前需要先安装python-pip工具。执行`pip install docker-compose`进行安装，如果安装后执行docker-compose报错，则执行`pip install --upgrade backports.ssl_match_hostname`，执行`docker-compose --version`查看版本信息，则说明安装成功。

**问：如何强力删除Windows系统上的顽固文件？**

**答：** 将以下代码保存为bat批处理文件，再将顽固文件拖至该批处理文件上完成删除。

    DEL /F /A /Q \\?\%1
    RD /S /Q \\?\%1

**问：如何快速打开Windows系统的【程序和功能】菜单？**

**答：** 【Win】+【R】键打开【运行】窗口，然后运行以下命令：

    appwiz.cpl

**问：如何修改Linux系统中的域名服务器？**

**答：** 修改配置文件`/etc/resolv.conf`。

**问：如何启动Linux ssh服务？**

**答：** 使用以下命令：

    service ssh start

**问：如何在Linux中查询ASCII表？**

**答：** 使用以下命令：

    man ascii

**问：如何在Linux中配置Java环境？**

**答：**

（1）下载最新的Java JDK安装包；
（2）解压缩文件并移动至`/opt`；

    tar -zxvf jdk-8u91-linux-x64.tar.gz
    mv jdk1.8.0_91 /opt
    cd /opt/jdk1.8.0_191

（3）设置环境变量，在`~/.bashrc`中添加以下内容；

    export JAVA_HOME=/opt/jdk1.8.0_91
    export CLASSPATH=.:${JAVA_HOME}/lib
    export PATH=${JAVA_HOME}/bin:$PATH

执行`source ~/.bashrc`；

（4）安装并注册；

    update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_91/bin/java 1
    update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_91/bin/javac 1
    update-alternatives --set java /opt/jdk1.8.0_91/bin/java
    update-alternatives --set javac /opt/jdk1.8.0_91/bin/javac

查看结果：

    update-alternatives --config java
    update-alternatives --config javac

（5）测试；

    java -version

**问：如何在Linux中计算子网？**

**答：** 使用以下命令：

    ipcalc 127.0.0.1/8

