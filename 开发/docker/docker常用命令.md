### 镜像仓库

#### 搜索镜像

```shell
docker search imageName
```

![image-20210603021136485](https://i.loli.net/2021/06/03/AUohbHzXLskavB4.png)

- **OFFICIAL：**官方源
- **AUTOMATED：**自动构建

#### 获取镜像

```shell
docker pull name:tag
```

- **name :**镜像源名字
- **tag :**版本，不指定则是最新版本

#### 推送镜像

```shell
docker push name:tag
```

### 本地镜像

#### 列出本地镜像

```shell
docker images
```

![image-20210602213938176](https://i.loli.net/2021/06/02/2w3F48NQgAPfBEG.png)

- **REPOSITORY：**表示镜像的仓库源
- **TAG：**镜像的标签，也就是版本

#### 删除本地镜像

```shell
docker rmi imageID
```

#### 保存镜像

```shell
docker save name:tag > aa.tar
```
#### 自镜像包恢复镜像

```shell
docker load < aa.tar
```

#### 自容器包恢复镜像

```shell
docker import bb.tar name:tag
```

#### 查看本地镜像创建历史

```shell
docker history name:tag
```

#### dockerfile创建镜像

```shell
docker build -t name:tag --rm -m 1000 -f dockerfilePath .
```

- **-f :**指定要使用的Dockerfile路径；
- **-m :**设置内存最大值；
- **--no-cache :**创建镜像的过程不使用缓存；
- **--pull :**尝试去更新镜像的新版本；
- **--rm :**设置镜像成功后删除中间容器；
- **--squash :**将 Dockerfile 中所有的操作压缩为一层。
- **-t:** 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。
- **.** 代表当前目录

### 容器

#### 创建一个新容器并启动

```shell
docker run --name containerName -p port1:port2 -v dir1:dir2 -d name:tag
```

- **-d:** 后台运行容器，并返回容器ID；
- **-P:** 随机端口映射，容器内部端口**随机**映射到主机的端口
- **-p:** 指定端口映射，格式为：**主机(宿主)端口:容器端口**
```shell
docker run --name containerName -p port1:port2 -v dir1:dir2 -ti name:tag /bin/bash
```

- **-i:** 以交互模式运行容器，通常与 -t 同时使用；
- **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- **/bin/bash **交互方式使用的脚本解释器

#### 创建容器但不启动

```shell
docker create
```

语法同 run

#### 在一个运行中的容器打开命令行

```
docker exec -ti containerName /bin/bash
```

#### 列出所有容器

```shell
docker ps -a
```

#### 列出运转中的容器

```shell
docker ps
```

#### 删除容器

```shell
docker rm containerID
```

#### 列出指定的容器的端口映射

```shell
docker port containerName/containerID
```

#### 导出容器

```
docker export -o bb.tar containerID
```

#### 容器创建镜像

```shell
docker commit -a authorName -m message containerID name:tag
```

- **-a :**提交的镜像作者；
- **-c :**使用Dockerfile指令来创建镜像；
- **-m :**提交时的说明文字；
- **-p :**在commit时，将容器暂停
#### 主机与容器间的文件拷贝(双向都可以拷贝)

```shell
docker cp dir1 containerID:dir2
```
```shell
docker cp containerID:dir2 dir1
```
### 存储

#### Bind mount方式

```shell
docker run --name containerName -v dir1:dir2 -d name:tag
```

- **dir1:**宿主机文件或文件夹
- **dir2:**容器文件或文件夹

#### Volume方式

```shell
docker run --name containerName -v volumeName:dir2 -d name:tag
```

- **dir2:**容器文件夹

#### tmpfs方式（挂载在内存中）
```shell
docker run --name containerName --tmpfs dir2 -d name:tag
```

### 网络

#### 以后再研究

```shell

```

### 不常用

#### 查看容器或镜像配置信息

```shell
docker inspect imageName:tag/containerName/containerID
```







