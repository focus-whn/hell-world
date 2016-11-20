# 阿铭Linux新资源托管地址
https://coding.net/u/aminglinux/p/resource/git/blob/master/README.md

# 【1.1】MySQL安装
MySQL官网下载：http://dev.mysql.com/downloads/  

老的阿铭下载：http://syslab.comsenz.com/downloads/linux/mysql-5.1.40-linux-i686-icc-glibc23.tar.gz

> 类似mysql-5.1.40-linux-i686-icc-glibc23.tar.gz名字的包是**免编译版本**，方便，性能也好，不用下源码包

MySQL 5.7官方安装文档：http://dev.mysql.com/doc/refman/5.7/en/binary-installation.html  

### 安装步骤：
> 以下为**mysql-5.1.73-linux-x86_64-glibc23.tar.gz**版本的安装步骤：

1、初始化
```text
# cd /usr/local/src/
# tar -zxvf mysql-5.1.73-linux-x86_64-glibc23.tar.gz

# useradd -s /sbin/nologin -M mysql
# mkdir -p /data/mysql                                          //数据文件的存放目录
# chown -R mysql /data/mysql/

# mv mysql-5.7.13-linux-glibc2.5-x86_64 /usr/local/mysql        //移动目录并重命名
# cd /usr/local/mysql
# ./scripts/mysql_install_db --user=mysql --datadir=/data/mysql
... ...
Installing MySQL system tables...
OK
Filling help tables...
OK                                                              //初始化MySQL，如果出现2个OK就说明成功了。
... ...
# echo $?
```
2、复制配置
```text
# ll support-files/*.cnf
-rw-r--r--. 1 7155 wheel  4682 Nov  5  2013 support-files/my-huge.cnf
-rw-r--r--. 1 7155 wheel 19731 Nov  5  2013 support-files/my-innodb-heavy-4G.cnf
-rw-r--r--. 1 7155 wheel  4656 Nov  5  2013 support-files/my-large.cnf
-rw-r--r--. 1 7155 wheel  4667 Nov  5  2013 support-files/my-medium.cnf
-rw-r--r--. 1 7155 wheel  2435 Nov  5  2013 support-files/my-small.cnf
# ll /etc/my.cnf                                //系统自带的mysql配置文件
-rw-r--r--. 1 root root 251 May 11  2016 /etc/my.cnf
# cp support-files/my-large.cnf /etc/my.cnf    //拷贝一个新的配置文件
```
3、编辑配置文件

`# vim /etc/my.cnf`  
```bash
# The MySQL server
[mysqld]
port            = 3306		            //监听端口（可通信）
socket          = /tmp/mysql.sock	    //socket通信（可通信）
skip-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 256
sort_buffer_size = 1M
read_buffer_size = 1M
read_rnd_buffer_size = 4M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size= 16M
# Try number of CPU's*2 for thread_concurrency
thread_concurrency = 8
... ...
# Replication Master Server (default)
# binary logging is required for replication
log-bin=mysql-bin		             //二进制bin log，如果打开，则用户在查询、更新、删除数据时，会作为日志记录。主要是mysql主从的时候会用到。不用的话可以注释掉，即关闭。

# binary logging format - mixed recommended
binlog_format=mixed	                    //bin log的格式，不用的话可以注释掉，即关闭。

# required unique id between 1 and 2^32 - 1
# defaults to 1 if master-host is not set
# but will not function as a master if omitted
server-id       = 1		            //不用的话可以注释掉，即关闭。
```
4、复制启动脚本  

`# cp support-files/mysql.server /etc/init.d/mysqld`

5、编辑启动脚本  

`# vim /etc/init.d/mysqld`

```bash
... ...
basedir=/usr/local/mysql		//添加mysql程序的存放位置
datadir=/data/mysql			//添加数据文件的存放位置
... ...
```

6、添加开机启动
```text
# chkconfig --add mysqld
# chkconfig mysqld on
```

7、启动数据库  
```text
# /etc/init.d/mysqld start
# ps -ef |grep mysqld
# netstat -npl |grep mysqld
```


---
# 【1.2】Apache编译安装
> 以下为**httpd-2.2.31.tar.gz**版本的安装步骤：

1、编译安装
```text
# tar -zxvf httpd-2.2.31.tar.gz
# cd httpd-2.2.31

# ./configure \
--prefix=/usr/local/apache2 \       //指定Apache安装目录
--with-included-apr \               //apr是Apache依赖的一个包，可以支持httpd跨平台运作，不论是Linux、Windows、Unix、Mac都可以运行httpd，这个包就是从底层去支持他
--enable-so \                       //表示启用DSO
--enable-deflate=shared \           //shared表示动态共享的方式编译安装
--enable-expires=shared \
--enable-rewrite=shared \
--with-pcre                         //pcre是正则相关的库

... ...
configure: error: in `/usr/local/src/httpd-2.2.31/srclib/apr':
configure: error: no acceptable C compiler found in $PATH
See `config.log' for more details
configure failed for srclib/apr
... ...

# yum install -y gcc   

...
checking whether to enable mod_deflate... checking dependencies
checking for zlib location... not found
checking whether to enable mod_deflate... configure: error: mod_deflate has been requested but can not be built due to prerequisite failures
...

# yum install -y zlib-devel
# echo $?

# make
# echo $?
# make install
# echo $?
```
> - [x] DSO是Dynamic Shared Objects（动态共享目标）的缩写，它提供了一种在运行时将特殊格式的代码在程序运行需要时，将需要的部分从外存调入内存执行的方法。Apache 支持动态共享模块，也支持静态模块，静态的话，会把需要的目标直接编译进apache的可执行文件中，相比较动态，虽然省去了加载共享模块的步骤，但是也加大了二进制执行文件的空间，变得臃肿。

2、httpd启动、停止、重启 
```text
# /usr/local/apache2/bin/apachectl start
# /usr/local/apache2/bin/apachectl stop
# /usr/local/apache2/bin/apachectl restart

# ps aux |grep httpd
# netstat -npl
```

3、查看所有模块  
```text
# /usr/local/apache2/bin/apachectl -M

httpd: apr_sockaddr_info_get() failed for aming.example.com
httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName
Loaded Modules:
...
 alias_module (static)
 so_module (static)             //静态模块
 deflate_module (shared)        //动态加载模块
 expires_module (shared)
 rewrite_module (shared)
Syntax OK
```

> - [x] 静态模块：都被编译进httpd这个文件。优点是加载快，但静态模块编译进去的太多会造成httpd这个文件非常大
>   ```text
>   # ll /usr/local/apache2/bin/httpd
>   -rwxr-xr-x. 1 root root 2262497 Nov 19 18:50 /usr/local/apache2/bin/httpd
>   ```
> - [x] 动态共享加载模块（.so）：用的时候加载，不用的时候不加载。优点是保持httpd文件比较小  
>   ```text
>   #  ll /usr/local/apache2/modules/
>   total 288
>   -rw-r--r--. 1 root root   9280 Nov 19 18:49 httpd.exp
>   -rwxr-xr-x. 1 root root  71744 Nov 19 18:50 mod_deflate.so
>   -rwxr-xr-x. 1 root root  37502 Nov 19 18:50 mod_expires.so
>   -rwxr-xr-x. 1 root root 164643 Nov 19 18:50 mod_rewrite.so
>   ```

4、只查看静态模块  
```text
# /usr/local/apache2/bin/apachectl -l

Compiled in modules:
... ...
  mod_actions.c
  mod_userdir.c
  mod_alias.c
  mod_so.c
```

**5、检查httpd.conf配置文件语法错误**  

**`# /usr/local/apache2/bin/apachectl -t`**

**6、重载httpd.conf配置文件**  

**`# /usr/local/apache2/bin/apachectl graceful`**


---
# 【1.3】Apache的mpm工作模式  
### 查看Apache的工作模式
```text
# /usr/local/apache2/bin/apachectl -l

Compiled in modules:
... ...
 prefork.c
... ...
```

### 常用的3种工作模式
1. **prefork**：主进程+子进程，即从主进程下派生很多子进程。每个子进程单独占用内存，某一个出现问题不会影响其他的子进程。但性能上高并发情况下比worker模式占用更多的内存。
```text
# ps aux |grep httpd
root      64040  0.0  0.1  39700  2180 ?        Ss   18:57   0:00  /usr/local/apache2/bin/httpd -k start       //Ss，代表主进程
daemon    64616  0.0  0.0  39700  1268 ?        S    19:50   0:00 /usr/local/apache2/bin/httpd -k start       //S，表示子进程
daemon    64617  0.0  0.0  39700  1268 ?        S    19:50   0:00 /usr/local/apache2/bin/httpd -k start
daemon    64618  0.0  0.0  39700  1268 ?        S    19:50   0:00 /usr/local/apache2/bin/httpd -k start
daemon    64619  0.0  0.0  39700  1268 ?        S    19:50   0:00 /usr/local/apache2/bin/httpd -k start
daemon    64620  0.0  0.0  39700  1268 ?        S    19:50   0:00 /usr/local/apache2/bin/httpd -k start
root      64825  0.0  0.0 103312   836 pts/0    S+   20:36   0:00 grep httpd
```

2. **worker**：主进程+子进程+线程，即子进程下还有线程。性能上在高并发情况下具有优势，只占用很少的内存。  

3. **event**：升级版worker，主进程+子进程+线程。性能上在保持长连接的时候会更好。
> - [x] 线程与进程的区别：  
线程是进程下派生出的更小的单元，进程的内存会共享给派生出的所有线程使用，也就意味着如果进程下的某一个线程出了故障，这个进程下的所有线程都会受到影响。  
进程

### 编译安装时修改工作模式
```text
# pwd
/usr/local/src/httpd-2.2.31
# ./configure --help
... ...
--with-mpm=MPM          Choose the process model for Apache to use.
                         MPM={beos|event|worker|prefork|mpmt_os2|winnt}
... ...
```

例如添加`--with-mpm=worker`。  
如果不添加的话，**Apache2.2默认**`prefork`，**Apache2.4默认**`event`


----
# 【1.4】php编译安装-安装顺序在最后
PHP官网下载：http://php.net/downloads.php

### 安装步骤
1、编译安装
```text
# tar -jxvf php-5.4.45.tar.bz2
# cd php-5.4.45

# ./configure \
--prefix=/usr/local/php \
--with-apxs2=/usr/local/apache2/bin/apxs \          //apxs，是帮助我们自动安装Apache动态扩展模块的工具。所以要先安装Apache，再安装PHP
--with-config-file-path=/usr/local/php/etc  \
--with-mysql=/usr/local/mysql \                     //php依赖MySQL。所以要先安装MySQL，再安装PHP
--with-libxml-dir \
--with-gd \
--with-jpeg-dir \
--with-png-dir \
--with-freetype-dir \
--with-iconv-dir \
--with-zlib-dir \
--with-bz2 \
--with-openssl \
--with-mcrypt \
--enable-soap \
--enable-gd-native-ttf \
--enable-mbstring \
--enable-sockets \
--enable-exif \
--disable-ipv6

... ... 
checking for xml2-config path...
configure: error: xml2-config not found. Please check your libxml2 installation.
... ...

# yum install -y libxml2-devel

... ...
checking for pkg-config... /usr/bin/pkg-config
configure: error: Cannot find OpenSSL's <evp.h>
... ...

# yum install -y openssl openssl-devel

... ...
checking for BZip2 in default path... not found
configure: error: Please reinstall the BZip2 distribution
... ...

# yum install -y bzip2 bzip2-devel

... ...
configure: error: jpeglib.h not found.
... ...
# yum install -y libjpeg libjpeg-devel

... ...
configure: error: png.h not found.
... ...

# yum install -y libpng libpng-devel

... ...
configure: error: freetype-config not found.
... ...

# yum install -y freetype freetype-devel

... ...
configure: error: mcrypt.h not found. Please reinstall libmcrypt.
... ...

# rpm -ivh 'http://www.lishiming.net/data/attachment/forum/epel-release-6-8_64.noarch.rpm'      //安装第三方yum源，因为官方源没有libmcrypt-devel这个包
# yum install -y libmcrypt-devel
# echo $?

# make
... ...
Build complete.
Don't forget to run 'make test'.
... ...
# echo $?
# make install
... ...
Installing PDO headers:          /usr/local/php/include/php/ext/pdo/
# echo $?
```
> - mcrypt.h not found的错误，要从猿课论坛的帖子中，下载第三方yum源。帖子名称《centos 5/6 epel yum源安装》http://ask.apelearn.com/question/6721  

> - `# make install`一次不成功的话，可以尝试使用命令`# make clean`，然后重新`# ./configure ...`，重新`# make`，`# make install`

> - 如果还遇到其他error，理论上都是缺少某个包，要使用命令 **`# yum list |grep -i xxx`** 搜索，然后安装即可。（-i，不区分大小写）

2、验证安装
```text
# ll /usr/local/apache2/modules/
total 30968
-rw-r--r--. 1 root root     9280 Nov 19 18:49 httpd.exp
-rwxr-xr-x. 1 root root 31413060 Nov 19 22:22 libphp5.so        //apxs帮助我们生成的动态模块
-rwxr-xr-x. 1 root root    71744 Nov 19 18:50 mod_deflate.so
-rwxr-xr-x. 1 root root    37502 Nov 19 18:50 mod_expires.so
-rwxr-xr-x. 1 root root   164643 Nov 19 18:50 mod_rewrite.so
```

3、查看PHP安装的静态模块
```text
# /usr/local/php/bin/php -m

[PHP Modules]
... ...
openssl
pcre
PDO
... ...
[Zend Modules]

```

4、查看PHP相关配置
```text
# /usr/local/php/bin/php -i

... ...
mcrypt

mcrypt support => enabled
mcrypt_filter support => enabled
Version => 2.5.8
Api No => 20021217
... ...
mysql

MySQL Support => enabled
Active Persistent Links => 0
Active Links => 0
Client API version => 5.1.73
... ...
```


----
# 【1.5】测试php解析


---


