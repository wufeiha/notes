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

## 注意点

1. **不要创建大镜像**

   **镜像缩小例子：**https://www.infoq.cn/article/7c3mgbkgrgtuzfleypsr

   **优化工具 dive：**https://github.com/wagoodman/dive

   - 基于[Alpine](https://hub.docker.com/_/alpine)或[BusyBox](https://hub.docker.com/_/busybox)的镜像非常小
   - 确保仅具有运行应用程序/进程所需的文件和库。
   - 不要安装不必要的软件包或运行将许多文件下载到新镜像层的“更新” 。

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

3. **镜像分层**

   有效利用分层文件系统，重新创建，管理和分发镜像将更加容易。

   1. 镜像分层优势
      - 基本上每个软件都是基于某个镜像去运行的，因此一旦某个底层环境出了问题，就不需要去修改全部基于该镜像的软件的镜像，只需要修改底层环境的镜像
      - 这个好处也是最大好处，就是可以共享资源，其他相同环境的软件镜像都共同去享用同一个环境镜像，而不需要每个软件镜像要去创建一个底层环境。
      - 可以复用,节省磁盘空间，相同的内容只需加载一份到内存。 修改dockerfile之后，再次构建速度快
   2. 镜像分层原则。【一条RUN命令就会构建一层镜像，有选择的合并RUN命令，减少不必要的分层】
      - 请始终为操作系统创建自己的基础镜像层。
      - 为用户名定义创建另一层。
      - 为运行时安装创建另一层。
      - 为配置创建另一层。
      - 最后是应用程序的另一层。

4. **不要从正在运行的容器中创建镜像**

   换句话说，不要使用“ docker commit”来创建镜像。这种创建镜像的方法不可复制，应完全避免。始终使用完全可复制的Dockerfile或任何其他S2I（从源到镜像）方法，如果将Dockerfile存储在源代码控制存储库（git）中，则可以跟踪对Dockerfile的更改。

5. **不要只使用“latest”标签**

   由于容器的分层文件系统性质，因此鼓励使用标签。

6. **不要在单个容器中运行多个进程**

   如果有多个进程，则管理起来可能会遇到更多麻烦，检索日志，并分别更新流程。

7. **不要将数据存储在镜像中**

8. **不要将凭据存储在镜像中**

   使用环境变量，你不想对镜像中的任何用户名/密码进行硬编码。使用环境变量从容器外部检索该信息。

9. **不要以root用户身份运行进程**

10. **不要依赖IP地址**

    每个容器都有自己的内部IP地址，如果你启动和停止容器，它可能会更改。如果应用程序或微服务需要与另一个容器通信，请使用环境变量将正确的主机名和端口从一个容器传递到另一个容器。

