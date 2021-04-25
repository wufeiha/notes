1. 官网下载yum源

   官网地址  https://dev.mysql.com/downloads/repo/yum/

   选择合适的版本下载

   ```shell
   wget https://dev.mysql.com/get/mysql57-community-release-el6-9.noarch.rpm
   ```

2. 安装yum源

   ```shell
   rpm -Uvh mysql57-community-release-el6-9.noarch.rpm
   ```

3. 修改yum mysql源默认安装版本**

   修改`/etc/yum.repos.d/mysql-community.repo`文件实现，将对应资源的enabled=1,其余enabled=0即可

   ```shell
   [mysql-connectors-community]
   name=MySQL Connectors Community
   baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/6/$basearch/
   enabled=1
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
   ```

4. 安装mysql**

   ```shell
   yum install mysql-community-server
   ```

5. 修改密码**

   5.1、找到原始密码

   ```shell
   cat /var/log/mysqld.log | grep password
   ```

   ![image-20201207190841373](https://i.loli.net/2020/12/07/Wgbq8PapdjoZe2c.png)

   5.2、mysql -u root -p 输入密码登录并修改密码

   ```shell
   mysql>  ALTER USER 'root'@'localhost' IDENTIFIED BY 'newPassword';
   ```

6. 修改编码**

   6.1、查看编码

   ```
   mysql> status;
   ```

   ![image-20201207191530231](https://i.loli.net/2020/12/07/JxayhwDpUGcVLKX.png)

   注意标红两项，起初一般默认都是latin,

   6.2、修改编码     编辑/etc/my.cnf   依据具体情况修改

   ![image-20201207191906901](https://i.loli.net/2020/12/07/5C7qUYQw83vtfTW.png)

7. 创建远程连接账户

   ```shell
   mysql> GRANT ALL PRIVILEGES ON *.* TO'remote'@'%'IDENTIFIED BY'yourPassword'WITH GRANT OPTION;
   mysql> FLUSH PRIVILEGES;
   ```

   

