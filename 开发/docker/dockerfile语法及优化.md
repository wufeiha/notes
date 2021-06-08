## 基本使用

```dockerfile
#基于centos镜像
FROM centos
#维护人的信息
LABEL maintainer = "The CentOS Project <303323496@qq.com>"
#安装httpd软件包
RUN yum -y update
RUN yum -y install httpd
#暴露端口
EXPOSE 80
#复制网站首页文件至镜像中web站点下
ADD index.html /var/www/html/index.html
#复制该脚本至镜像中，并修改其权限
ADD run.sh /run.sh
RUN chmod 775 /run.sh
#当启动容器时执行的脚本文件
CMD ["/run.sh"]
```

- RUN：构建镜像时执行的命令，<font color=red>每一个 RUN 命令都是新建的一层，可以使用&&将多个命令组合成一个命令，缩小层数</font>
- CMD：启动容器时执行的命令，<font color=red>如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效</font>
- WORKDIR：制定容器工作目录
- ENV：设置环境变量
- ADD：拷贝文件，支持自动解压及远程文件，COPY命令的超集功能强大，但是比较复杂
- COPY：拷贝文件，如果没有自动解压及拷贝远程文件的需求，推荐优先使用
- ENTRYPOINT：与CMD相同，只是由ENTRYPOINT启动的程序**不会被docker run命令行指定的参数所覆盖**，<font color=red>Dockerfile文件中也可以存在多个ENTRYPOINT指令，但仅有最后一个会生效</font>
- VOLUME：设置匿名卷
- USER：为RUN、CMD、ENTRYPOINT指定运行用户,用户 必须已经存在
- ARG：构建镜像时制定参数,构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量

## 优化

**优化例子：**https://www.infoq.cn/article/7c3mgbkgrgtuzfleypsr

**优化工具 dive：**https://github.com/wagoodman/dive

1. **使用轻量化的基础镜像**

   基于[Alpine](https://hub.docker.com/_/alpine)或[BusyBox](https://hub.docker.com/_/busybox)的镜像非常小

2. **多阶段构建**

   两个阶段：第一个阶段中，我们编译项目，第二个阶段中，我们在 web 服务器上部署应用程序

   <font color=red>第一阶段的镜像不会被自动删除，Docker 将它保存在 cache 中(其实就是中间镜像，docker images列出的)，如果我们在另一个构建镜像过程中执行了相同的阶段，就可以使镜像构建更快。如果不需要，你必须手动删除第一阶段镜像</font>

   ```dockerfile
   FROM node:10-alpine AS build
   WORKDIR /app
   COPY app /app
   RUN npm install && npm run build
    
   
   FROM nginx:stable-alpine
   COPY --from=build /app/build /usr/share/nginx/html
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   ```
   
3. **有选择的组合RUN命令减少镜像的层数**
   
   镜像分层有以下好处，所以不可过分减少层数
   
   1. 基本上每个软件都是基于某个镜像去运行的，因此一旦某个底层环境出了问题，就不需要去修改全部基于该镜像的软件的镜像，只需要修改底层环境的镜像。
   2. 这个好处也是最大好处，就是可以共享资源，其他相同环境的软件镜像都共同去享用同一个环境镜像，而不需要每个软件镜像要去创建一个底层环境。
   3. 可以复用,节省磁盘空间，相同的内容只需加载一份到内存。 修改dockerfile之后，再次构建速度快
