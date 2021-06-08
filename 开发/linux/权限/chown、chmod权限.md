普通权限
====

**1、权限介绍**

> + 4 / r  读权限
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

> + suid  |  4
> + sgid  |  2
> + sbit   |  1

**2、suid**

​	2.1 作用对象

​	可执行文件

​	2.2 特点

> + 如果执行者对于该二进制可执行文件<font color='red'>具有 x 的权限</font>，执行者将具有该文件的所有者的权限
> + 本权限仅在执行该二进制可执行文件的过程中有效

​	2.3 过程
![image-20201214114230816](https://i.loli.net/2020/12/14/sg7JI6xGb2OcT9D.png)
​	2.4 授权

```shell
chmod 4775 /usr/bin/passwd
```
​		执行ls -l 显示 
```shell
-rwsrwxr-x   /usr/bin/passwd
```
**3、sgid**

3.1、作用对象
可执行文件、目录
3.2、作用

> + 作用在可执行文件上与SUID 相同,执行者将具有该文件的所有组的权限	
> + 作用在目录上
>   >+ 若用户在此目录下拥有 w 权限，则用户所创建的新文件的用户组与该目录的用户组相同
>

3.3 授权

```shell
chmod 2775 /test/a
```
​		执行ls -l 显示 
```shell
-rwxrwsr-x   /test/a
```

**4、sbit**

​	4.1、作用对象
​	目录
​    4.2、作用
​	当用户在该目录下创建新文件或目录时，仅有自己和 root 才有权力删除。
​    4.3、授权

```shell
chmod 1777 /test/a
```
执行ls -l 显示 
```shell
-rwxrwxrwt   /test/a
```