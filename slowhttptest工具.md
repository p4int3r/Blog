# slowhttptest工具

## 安装

	apt-get install slowhttptest

## 使用

    slowhttptest -c 1000 -H -g -o my_header_status -i 10 -r 200 -t GET -u http://192.168.1.1 -x 24 -p 3

    slowhttptest -c 1000 -B -g -o my_body_stats -i 110 -r 200 -s 8192 -t FAKEVERB -u https://myseceureserver/resources/loginform.html -x 10 -p 3

    slowhttptest -c 1000 -H -g -o my_header_stats -i 10 -r 200 -t GET -u https://myseceureserver/resources/index.html -x 24 -p 3

    slowhttptest -c 1000 -X -r 1000 -w 10 -y 20 -n 5 -z 32 -u http://someserver/somebigresource -p 5 -l 350 -e x.x.x.x:8080