# SSH隧道

## SSH本地端口转发

将一个本地端口与远程服务器建立隧道。

### 配置文件/etc/ssh/sshd_config

    PermitRootLogin yes
    Port 53
    PasswordAuthentication yes

### 启动服务

    service ssh start

### 重启服务

    service ssh restart

### 建立隧道

    ssh -L <listen port>:<remote ip>:<remote port> user@<ssh server> -p <ssh server port>
    ssh -CfN -L <listen port>:<remote ip>:<remote port> user@<ssh server> -p <ssh server port>
    ssh -fNg -L <listen port>:<remote ip>:<remote port> user@<ssh server> -p <ssh server port>

    选项说明：
    -f 后台运行进程
    -N 不执行登录shell
    -g 复用访问时作为网关，支持多主机访问本地侦听端口
    -C 压缩数据传输

## SSH远程端口转发

### 建立隧道

    ssh -fNg -R <listen port>:<remote ip>:<remote port> user@<SSH server> -p <ssh server port>

## SSH动态端口转发

### 建立隧道

    ssh -CfNg -D <listen port> user@<SSH server> -p <server port>

### 设置Socks代理

在浏览器设置代理。

## X协议转发

    ssh -X user@<SSH server> -p <server port>

