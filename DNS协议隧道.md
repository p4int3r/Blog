# DNS协议隧道

DNS隧道原理：注册受自己控制的DNS记录。

## dns2tcp

### 安装

C/S架构（dns2tcpc/dns2tcpd）

    apt-get install dns2tcp

### 配置

第一种：服务端配置文件`/etc/dns2tcpd.conf`；
第二种：用户根目录`.dns2tcprcd`；

在配置文件中配置侦听IP，侦听端口，域名，提供的资源。
添加`key=password`来配置密码。

### 启动服务端

    dns2tcpd -F -d 1 -f /etc/dns2tcpd.conf

    选项说明：
    -F 前端运行
    -d debug级别，1-3
    -f 指定配置文件

### 客户端连接

    dns2tcpc -c -k pass -d 1 -l 2222 -r ssh -z test.lab.com

    选项说明：
    -k 连接密码
    -c 启动流量压缩
    -l 客户端本地的侦听端口
    -r 服务器端配置文件中使用的资源
    -z 利用该域名去查找DNS服务器

### 隧道嵌套

在已经建立好的DNS隧道的基础上，使用SSH隧道进行嵌套。

    ssh -CfNg root@127.0.0.1 -p 2222 -D 7002

## iodine

支持多平台：Linux，BSD，Mac OS，Windows

### 启动服务端

    iodined -f -c 10.0.0.1 test.lab.com

    选项说明：
    -f 前端显示
    -c 不检查客户端IP地址
    10.0.0.1 该IP是隧道在本地虚拟网卡的IP

### 启动客户端

    iodine -f 10.0.0.2 test.lab.com

    选项说明：
    10.0.0.2 该IP为本地DNS服务器的IP


