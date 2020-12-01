普通权限
====

**1、权限介绍**

> + 4 / rw  读权限
> + 2 / w   写权限
> + 1 / x    执行权限

**2、划分用户**

```shell
chown -R nginx:nginx /opt/nginx
```

​	以上表示将/opt/nginx目录划分给nginx用户组下的nginx用户，"-R" 表示递归

**3、分配权限**

```shell
chmod -R 500 /opt/nginx
```

​	"-R" 表示递归，为/opt/nginx分配500权限。第一个数字‘5’代表<font color='green'>拥有者</font>有读、可执行权限；第二个数字‘0’表示<font color='green'>群组</font>无权限；第三个数字‘0’表示<font color='green'>其他组</font>无权限。这里说明的是数字授权方式，字母授权方式不在赘述。

特殊权限
====

**1、权限介绍**

> + 
> + 2 / w   写权限
> + 1 / x    执行权限

**2、划分用户**

```shell
chown -R nginx:nginx /opt/nginx
```

​	以上表示将/opt/nginx目录划分给nginx用户组下的nginx用户，"-R" 表示递归

**3、分配权限**

```shell
chmod -R 500 /opt/nginx
```

​	"-R" 表示递归，为/opt/nginx分配500权限。第一个数字‘5’代表<font color='green'>拥有者</font>有读、可执行权限；第二个数字‘0’表示<font color='green'>群组</font>无权限；第三个数字‘0’表示<font color='green'>其他组</font>无权限。

