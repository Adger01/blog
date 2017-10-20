## 新建目录
>下载的软件都存放在这个目录   

<pre>
# mkdir /opt/soft
# cd /opt/soft
</pre>

## 安装mysql
- 安装依赖

<pre># yum -y install gcc-c++ ncurses-devel cmake make perl gcc autoconf automake zlib libxml libgcrypt libtool bison
</pre>


- 下载mysql5.6

<pre>
# cd /opt/soft 
# wget http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.30.tar.gz
</pre>

- 创建用户组 用户

<pre>
# groupadd mysql
# useradd -r -g mysql mysql
</pre>

- 创建目录 （安装目录&存放数据目录）

<pre>
 # mkdir -p /usr/local/mysql 
 # mkdir -p /data/mysqldb
</pre>

- 编译安装

<pre>
 # tar zxvf mysql-5.6.30.tar.gz
 # cd mysql-5.6.30
 # cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql 
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DMYSQL_DATADIR=/data/mysqldb -DMYSQL_TCP_PORT=3306 -DENABLE_DOWNLOADS=1 -DEXTRA_CHARSETS=all -DENABLED_LOCAL_INFILE=1 -DWITH_READLINE=1 
 # make 
 # make install
</pre>

>注：重新运行配置，需要删除CMakeCache.txt文件 rm CMakeCache.txt ) 

- 修改两个目录的权限  

<pre>
 # cd /usr/local/mysql
 # chown -R mysql:mysql .  
 
 # cd /data/mysqldb
 # chown -R mysql:mysql .
</pre>
 
- 初始化mysql数据库

<pre>
 # cd /usr/local/mysql
 # /scripts/mysql_install_db --user=mysql --datadir=/data/mysqldb
</pre>

- 复制mysql服务启动配置文件

<pre>
# cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
</pre>

>（注：如果/etc/my.cnf文件存在，则覆盖。）

- 复制mysql服务启动脚本及加入PATH路径

<pre>
# cp support-files/mysql.server /etc/init.d/mysqld
</pre>

- 启动mysql服务并加入开机自启动

<pre>
# service mysqld start 
# chkconfig --level 35 mysqld on
</pre> 

- 检查mysql服务是否启动

<pre># netstat -tulnp | grep 3306 </pre>

- 需要把mysql加入环境变量 

<pre>
# vim /etc/profile
</pre>

- 在最后加入如下命令 

><pre>MYSQL_HOME=/usr/local/mysql
PATH=$MYSQL_HOME/bin:$PATH 
export PATH MYSQL_HOME </pre> 

<pre># source /etc/profile 
# mysql -u root -p` （密码为空，如果能登陆上，则安装成功）
</pre>

- 修改MySQL用户root的密码

<pre># mysql -u root -p 
# use mysql 
# update user set password=password(‘MySQL13456~’) where user=’root’   
# flush privileges`(刷新权限)   
</pre>



### 可能造成启动失败的原因
> MySQL提示:The server quit without updating PID file问题的解决办法

- 可能是/usr/local/mysql/data/mysql.pid文件没有写的权限
解决方法 ：给予权限，执行 “chown -R mysql:mysql /var/data” “chmod -R 755 /usr/local/mysql/data”  然后重新启动mysqld！

- 可能进程里已经存在mysql进程
解决方法：用命令“ps -ef|grep mysqld”查看是否有mysqld进程，如果有使用“kill - 进程号”杀死，然后重新启动mysqld！

- 可能是第二次在机器上安装mysql，有残余数据影响了服务的启动。
解决方法：去mysql的数据目录/data看看，如果存在mysql-bin.index，就赶快把它删除掉吧，它就是罪魁祸首了。本人就是使用第三条方法解决的 ！

- mysql在启动时没有指定配置文件时会使用/etc/my.cnf配置文件，请打开这个文件查看在[mysqld]节下有没有指定数据目录(datadir)。
解决方法：请在[mysqld]下设置这一行：datadir = /usr/local/mysql/data

- skip-federated字段问题
解决方法：检查一下/etc/my.cnf文件中有没有没被注释掉的skip-federated字段，如果有就立即注释掉吧。

- 错误日志目录不存在
解决方法：使用“chown” “chmod”命令赋予mysql所有者及权限

- selinux惹的祸，如果是centos系统，默认会开启selinux
解决方法：关闭它，打开/etc/selinux/config，把SELINUX=enforcing改为SELINUX=disabled后存盘退出重启机器试试。
 
##  nginx 安装
- 安装依赖  

<pre>
# yum -y install gcc automake autoconf libtool make gcc-c++ glibc libxslt-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5-devel libidn libidn-devel openssl openssl-devel pcre pcre-devel libmcrypt libmcrypt-devel cmake
</pre>

- 下载并解压  

<pre>
# wget  http://nginx.org/download/nginx-1.12.2.tar.gz
# tar -zvxf nginx-1.12.2.tar.gz
</pre>

- 创建用户组和用户 
 
<pre>
groupadd www
useradd -g www www
</pre>


- 编译并安装  

<pre>
# cd nginx-1.12.2
# ./configure --prefix=/usr/local/nginx --sbin-path=/usr/sbin/nginx --user=www --group=www --with-http_ssl_module --with-http_gzip_static_module
# make && make install  
</pre>

- 启动   

<pre>
# nginx
</pre>  
    
## php安装
<pre>
# mkdir /usr/local/php71
# yum -y install php-mcrypt libmcrypt libmcrypt-devel  autoconf  freetype gd jpegsrc libmcrypt libpng libpng-devel libjpeg libxml2 libxml2-devel zlib curl curl-devel
# wget http://cn2.php.net/distributions/php-7.1.10.tar.gz  
# tar -zvxf php-7.1.10.tar.gz 
# cd php-7.1.10
</pre>

- 配置编译参数

<pre>
# ./configure --prefix=/usr/local/php71/ --enable-mbstring --with-curl --with-gd --enable-fpm --enable-mysqlnd  --with-pdo-mysql=mysqlnd --with-config-file-path=/usr/local/php71/etc/ --with-mysqli=mysqlnd 
#make && make install
# cp /usr/local/php71/etc/php-fpm.conf.default /usr/local/php71/etc/php-fpm.conf
# cp /usr/local/php71/etc/php-fpm.d/www.conf.default /usr/local/php71/etc/php-fpm.d/www.conf
# /usr/local/php71/sbin/php-fpm    
</pre>