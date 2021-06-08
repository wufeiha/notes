Linux中有两种防火墙软件，ConterOS7.0以上使用的是firewall，ConterOS7.0以下使用的是iptables，本文将分别介绍两种防火墙软件的使用。

- 开启防火墙：

```shell
systemctl start iptables.service
```

- 关闭防火墙：

```shell
systemctl stop iptables.service
```

- 查看防火墙状态：

```shell
systemctl status iptables.service
```

- 设置开机启动：

```shell
systemctl enable iptables.service
```

- 禁用开机启动：

```shell
systemctl disable iptables.service
```

- 查看filter表的几条链规则(INPUT链可以看出开放了哪些端口)：

```shell
iptables -L -n
```



![展示图片](https://user-gold-cdn.xitu.io/2019/6/13/16b51180ea597ac9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 查看NAT表的链规则：

```shell
iptables -t nat -L -n
```



![展示图片](https://user-gold-cdn.xitu.io/2019/6/13/16b511810ad4af70?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 清除防火墙所有规则：

```shell
iptables -F
iptables -X
iptables -Z
```

- 给INPUT链添加规则（开放8080端口）：

```shell
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
```



![展示图片](https://user-gold-cdn.xitu.io/2019/6/13/16b511810ae98aaf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 查找规则所在行号：

```shell
iptables -L INPUT --line-numbers -n
```



![展示图片](https://user-gold-cdn.xitu.io/2019/6/13/16b511810aeee047?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 根据行号删除过滤规则（关闭8080端口）：

```shell
iptables -D INPUT 1
```

![展示图片](https://user-gold-cdn.xitu.io/2019/6/13/16b511810fc14ec7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

