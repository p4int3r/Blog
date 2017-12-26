# FAQ

**问：如何在CentOS 7下安装python-pip?**

**答：** 直接执行`yum install python-pip`安装，如果没有python-pip包，则执行命令`yum -y install epel-release`，再次执行`yum install python-pip`进行安装，最后执行`pip install --upgrade pip`进行升级。

**问：如何安装docker-compose工具？**
**答：** 在安装docker-compose工具之前需要先安装python-pip工具。执行`pip install docker-compose`进行安装，如果安装后执行docker-compose报错，则执行`pip install --upgrade backports.ssl_match_hostname`，执行`docker-compose --version`查看版本信息，则说明安装成功。

**问：如何强力删除Windows系统上的顽固文件？**
**答：** 将以下代码保存为bat批处理文件，再将顽固文件拖至该批处理文件上完成删除。

    DEL /F /A /Q \\?\%1
    RD /S /Q \\?\%1