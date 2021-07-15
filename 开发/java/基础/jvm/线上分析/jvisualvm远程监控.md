**jmx链接   (监控cpu，内存，线程，mbean等功能)**

 1. 准备 jmxremote.access 文件

    参考文件位置 ${JAVA_HOME}\jre\lib\management\jmxremote.access

 2. 准备 jmxremote.password 文件

    参考文件位置 ${JAVA_HOME}\jre\lib\management\jmxremote.password

 3. 配置

    1. 将jmxremote.access，jmxremote.password，放到tomcat conf目录下，<font color='red'>并赋予只读权限</font>

    2. 修改bin 下 catalina.bat 添加如下配置   linux版去掉“^”

       window版本，“^” 代表换行

       > set CATALINA_OPTS= ^
       > -Dcom.sun.management.jmxremote ^
       > -Dcom.sun.management.jmxremote.ssl=false ^
       > -Dcom.sun.management.jmxremote.port=1099 ^
       > -Dcom.sun.management.jmxremote.authenticate=true ^
       > -Dcom.sun.management.jmxremote.password.file=../conf/jmxremote.password ^
       > -Dcom.sun.management.jmxremote.access.file=../conf/jmxremote.access
    
 4. 启动

    随tomcat或者jar进程自动启动，每个tomcat或者jar进程需要单独配置

**jstatd链接   (virtualGC 展示堆内存各部分详情)**

1. 在  ${JAVA_HOME}\jre\lib\security\java.policy  文件中加入如下配置，<font color='red'>注意要写绝对路径，两个系统写法一致，路径斜杠方向一样</font>

   linux版/window版
   
   > grant codebase "file:/usr/java/jdk1.8.0_181-cloudera/lib/tools.jar" {
   >         permission java.security.AllPermission;
   > };
   
2. 启动    -p 表示指定端口

   linux  

   ~~~shell
   . jstatd -p 12345 &
   ~~~

   window
   ~~~bash
   jstatd -p 12345
   ~~~

