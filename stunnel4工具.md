# stunnel4工具

跨平台工具

## 安装

    apt-get install stunnel4

## 服务器端配置

    生成证书：
    openssl req -new -days 365 -nodes -x509 -out /etc/stunnel/stunnel.pem -keyout /etc/stunnel/stunnel.pem

## 配置

配置文件/etc/stunnel/stunnel.conf

    cert=/etc/stunnel/stunnel.pem
    setuid=stunnel4
    setgid=stunnel4
    pid=/var/run/stunnel4/stunnel4.pid
    [mysqls]
    accept=0.0.0.0:443
    connect=1.1.1.1:3306

stunnel4自动启动：

    /etc/default/stunnel4
    enabled=1

## 启动

    service stunnel4 start

## 客户端配置

配置文件：/etc/stunnel/stunnel.conf

    client=yes
    [mysqls]
    accept=3306
    connect=192.168.1.11:443

自动启动：

    /etc/default/stunnel4
    enabled=1

## 启动

    service stunnel4 stop/start

