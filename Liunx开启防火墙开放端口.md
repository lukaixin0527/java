# Liunx防火墙

### 查看防火墙状态

```
firewall-cmd --state
```

### 开启防火墙

```
systemctl start firewalld.service
```

### 关闭防火墙

```
systemctl stop firewalld.service
```

### 重启防火墙

```
service firewalld restart
```

### 查看防火墙所有开放的端口

```
firewall-cmd --zone=public --list-ports
```

### 查看监听的端口

```
netstat -lnpt
```

### 检查端口被哪个进程占用

```
netstat -lnpt |grep 5672
```

### 查看进程的详细信息

```
ps 6832
```

### 添加指定需要开放的端口

```
添加指定需要开放的端口： firewall-cmd --add-port=123/tcp --permanent
重载入添加的端口： firewall-cmd --reload
查询指定端口是否开启成功： firewall-cmd --query-port=123/tcp
```

### 移除指定端口

```
firewall-cmd --permanent --remove-port=123/tcp
```

