# slowhttptest工具

## 安装

	apt-get install slowhttptest

## 使用

    slowhttptest -c 1000 -H -g -o my_header_status -i 10 -r 200 -t GET -u http://192.168.1.1 -x 24 -p 3
