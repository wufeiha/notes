Linux中有两种防火墙软件，ConterOS7.0以上使用的是firewall，ConterOS7.0以下使用的是iptables，本文将分别介绍两种防火墙软件的使用。

## Firewall

- 开启防火墙：

```shell
systemctl start firewalld
```

- 关闭防火墙：

```shell
systemctl stop firewalld
```

- 查看防火墙状态：

```shell
systemctl status firewalld
```

- 设置开机启动：

```shell
systemctl enable firewalld
```

- 禁用开机启动：

```shell
systemctl disable firewalld
```

- 重启防火墙：

```shell
firewall-cmd --reload
```

- 开放端口（修改后需要重启防火墙方可生效）：

```shell
firewall-cmd --zone=public --add-port=8080/tcp --permanent
```

- 查看开放的端口：

```shell
firewall-cmd --list-ports
```

- 关闭端口：

```shell
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
```