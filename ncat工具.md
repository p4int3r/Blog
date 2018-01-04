# NCAT

包含在Nmap工具中。

## 代理功能

    ncat -l 8080 --proxy-type http --proxy-auth user:password

## Broker中介功能

服务器端：

    ncat -l 333 --broker

客户端：

    ncat 1.1.1.1 333
    批量执行命令：ncat 1.1.1.1 333 --sh-exec "echo `pwd`"
    批量传文件：ncat --send-only 1.1.1.1 <inputfile

# SOCAT工具

被称为增强版nc工具。双向数据流通道工具。

安装：

    apt-get install socat

连接端口：

    socat - tcp:1.1.1.1:80

侦听端口：

    socat - tcp4-listen:22
    socat - tcp-l:333

接收文件：

    socat tcp4-listen:333 open:2.txt,create,append

发送文件：

    cat 1.txt | socat - tcp4:1.1.1.1:333

## 流量操控部分

远程shell-服务器端：

    socat tcp-l:23 exec:sh,pty,stderr

端口转发：

    socat tcp4-listen:22,fork tcp4:1.1.1.1:22

远程执行命令：

    服务器：socat -udp-l:2001
    客户端：echo "`id`" | socat -udp4-datagram:1.1.1.1:2001

UDP全端口任意内容发包：

    for PORT in {1..65535};do echo "aaa" | socat - udp4-datagram:1.1.1.1:$PORT;sleep .1;done

二进制编辑器：

    echo -e "\0\14\0\0\c" | socat -u - file:/usr/bin/squid.exe,seek,seek=0x00074420

简单的Web服务器：

    socat -T 1 -d -d tcp-l:10081,reuseaddr,fork,crlf system:"echo -e \"\\\"HTTP/1.0 200 OK\\\nDocumentType:text/plain\\\n\\\ndate:\$\(date\)\\\server:\$SOCAT_SOCKADDR:\$SOCAT_SOCKPORT\\\nclient:\SOCAT_PEERADDR:\$SOCAT_PEERPORT\\\n\\\"\";cat;echo -e \"\\\"\\\n\\\"\""
