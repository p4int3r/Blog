# NTP放大攻击

1. 发现NTP服务

	nmap -sU -p123 192.168.1.1

2. 发现漏洞

	ntpdc -n -c monlist 192.168.1.1
	ntpq -c rv 192.168.1.1
	ntpdc -c sysinfo 192.168.1.1

3. 配置文件

	/etc/ntp.conf

	restrict -4 default kod nomodify notrap nopeer noquery
	restrict -6 default kod nomodify notrap nopeer noquery