# nping工具

## 使用

    nping [Probe mode] [Options] {target specification}

TCP全连接Dos攻击

    nping --tcp-connect --rate=10000 -c 100000000 -q 1.1.1.1

查看公网IP
    nping --echo-client "public" echo.nmap.org --udp
