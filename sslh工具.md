# sslh工具

端口分配器

1. 根据客户端第一个包检测协议类型；
2. 根据协议检测结果将流量转发给不同目标；
3. 支持HTTP，HTTPS，SSH，OPENVPN，tinc，XMPP和其他可基于正则表达式判断的任何协议类型；
4. 适用于防火墙允许443端口入站访问流量的环境。

## 配置

配置文件`/etc/default/sslh`

    RUN=yes
    DAEMON_OPTS #(配置访问资源)

## 启动服务

    service sslh start

