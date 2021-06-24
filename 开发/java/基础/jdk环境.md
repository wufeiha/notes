1. **安装**

   ```shell
   yum install java-1.8.0-openjdk.x86_64
   ```

2. **配置环境变量**

   编辑    /etc/profile  添加如下内容

   ```shell
   export JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
   export PATH=$JAVA_HOME/bin:$PATH
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   ```

3. **启用配置文件**

   ```shell
   source /etc/profile
   ```

   

