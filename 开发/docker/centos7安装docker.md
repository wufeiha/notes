Docker从1.13版本之后采用时间线的方式作为版本号，分为社区版CE和企业版EE。

社区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件等。

社区版按照stable和edge两种方式发布，每个季度更新stable版本，如17.06，17.09；每个月份更新edge版本，如17.09，17.10。

##  安装docker

1、Docker 要求 CentOS 系统的内核版本高于 3.10 ，通过 **uname -r** 命令查看你当前的内核版本

```shell
 $ uname -r
```

2、使用 `root` 权限登录 Centos。确保 yum 包更新到最新。

```shell
$ yum update
```

3、卸载旧版本(如果安装过旧版本的话)

```shell
$ yum remove docker  docker-common docker-selinux docker-engine
```

4、安装epel扩展软件源

```shell
$ yum -y install epel-release
```
5、安装需要的软件包， yum-utils提供yum-config-manager功能。device Mapper是Linux2.6内核中支持逻辑卷管理的通用设备映射机制，它为实现用于存储资源管理的块设备驱动提供了一个高度模块化的内核架构。LVM是对磁盘分区进行管理的一种机制，建立在硬盘和分区之上的一个逻辑层，用来提高磁盘管理的灵活性。通过LVM可将若干个磁盘分区连接为一个整块的卷组(Volume Group)，形成一个存储池。可以在卷组上随意创建逻辑卷(Logical Volumes)，并进一步在逻辑卷上创建文件系统，与直接使用物理存储在管理上相比，提供了更好灵活性。<font color=red>一般不需要安装，系统自带</font>。

```shell
$ yum install -y yum-utils device-mapper-persistent-data lvm2
```

6、添加docker源

```shell
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
7、安装docker

```shell
$ yum install docker-ce
```