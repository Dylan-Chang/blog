
## Notice
nginx 502 - 检查 php-fpm 启动 

使用lsof
lsof -i:端口号查看某个端口是否被占用 

## [Firewall](http)

### 开启80端口

firewall-cmd --zone=public --add-port=80/tcp --permanent

出现success表明添加成功

 

命令含义：

--zone #作用域

--add-port=80/tcp  #添加端口，格式为：端口/通讯协议

--permanent   #永久生效，没有此参数重启后失效

### 重启防火墙

systemctl restart firewalld.service

### 运行、停止、禁用firewalld

启动：# systemctl start  firewalld

查看状态：# systemctl status firewalld 或者 firewall-cmd --state

停止：# systemctl disable firewalld

禁用：# systemctl stop firewalld



## curl http://localhost
