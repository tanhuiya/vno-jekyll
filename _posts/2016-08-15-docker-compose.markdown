---
layout: post
title: Docker docker-compose 配置lnmp开发环境
date: 2016-08-15 15:32:24.000000000 +09:00
---

###1 安装Docker
本机在CentOs7 下安装Docker，其他平台也一样 
首先查看当前内核版本是否高于 3.10
 
	$ uname -r
	3.10.0-327.el7.x86_64
	//安装docker	
	yum -y install docker
	//启动docker
	service docker start
	//查看版本信息
	docker info
	
安装hello-world 镜像
	
	docker run hello-world

运行时的输出可以看到docker 是从本地镜像开始找，如果没有该镜像则去Docker Hub 去下载并运行。


###2 安装MySQL

获取MySQL 我直接用的官方的镜像，输入以下命令

	docker pull mysql:5.6
	
运行MySQL

	docker run -p 3306:3306 --name test_mysql -v $PWD/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d --privileged=true mysql:5.6
	
命令说明：

	-p 3306:3306：将容器的3306端口映射到主机的3306端口
	-v PWD/mysql/data:/var/lib/mysql：将主机当前目录下的mysql/data文件夹挂载到容器的/var/lib/mysql 下，在mysql容器中产生的数据就会保存在本机mysql/data目录下
	-e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码
	-d 后台运行容器
	--name 给容器指定别名
	--privileged=true centos7 可能会碰到权限问题，需要加参数
	
执行以下命令进入mysql 运行环境

	docker exec -it test_mysql bash
	
	这样就进入mysql容器了，可以查看mysql 命令
	mysql －u root -p 
	输入密码就可以操作数据库
	mysql> show databases;
	+--------------------+
	| Database           |
	+--------------------+
	| information_schema |
	| mysql              |
	| performance_schema |
	+--------------------+
	4 rows in set (0.07 sec)
	mysql> create database if not exists test_db;
	Query OK, 1 row affected, 1 warning (0.00 sec)
	
	mysql> use test_db
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A
	
	Database changed
	mysql> create table people (name varchar(10), age int);
	Query OK, 0 rows affected (0.13 sec)
	
	mysql> insert into people values('liming',10);
	Query OK, 1 row affected (0.03 sec)
	

###3 安装PHP
同样使用官方的php镜像，不过需要支持mysql扩展。所以我们这次用Dockerfile 来构建一个镜像，以下是Dockerfile

	 FROM  php:5.6-fpm
	 RUN apt-get update && apt-get install -y \
	 libfreetype6-dev \
	 libjpeg62-turbo-dev \
	 libpng12-dev \
	 vim \
	 && docker-php-ext-install pdo_mysql \
	 && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
	 && docker-php-ext-install gd \

我们在原本的php5.6上安装了一些工具以及扩展。

build 我们新建的镜像

	docker build -t="php-fpm5.6/v2" .

然后用这个镜像跑一个php环境的容器

	docker run -d -p 9000:9000 -v /home/tanhui/composeTest/code/:/var/www/html/ --name php-with-mysql --link test_mysql:mysql  --volumes-from test_mysql --privileged=true php-fpm5.6/v2
	
参数解析

	－v 将本地磁盘上的php代码挂载到docker 环境中，对应docker的目录是 /var/www/html/ 
	--name 新建的容器名称 php-with-mysql
	--link 链接的容器，链接的容器名称：在该容器中的别名，运行这个容器是，docker中会自动添加一个host识别被链接的容器ip
	--privileged=true 权限问题
	
进入容器

	docker exec -it php-with-mysql bash
	cd /var/www/html && ls
	
这里就会看到你本地磁盘下所挂载的文件,在该目录下可以添加一个 mysql.php 文件，如下

	try {
	    $dbh = new PDO('mysql:host=mysql;dbname=test_db', 'root','12
	3456');
	    foreach($dbh->query('SELECT * from people') as $row) {
	        print_r($row);
	    }
	    $dbh = null;
	} catch (PDOException $e) {
	    print "Error!: " . $e->getMessage() . "<br/>";
	    die();
	}

退出编辑
	
	root@f5c9b982690a:/var/www/html# php mysql.phpphp mysql.php
	Array
	(
	    [name] => liming
	    [0] => liming
	    [age] => 10
	    [1] => 10
	)
	
输出结果如上，这样我们php链接mysql就成功了。

###4 安装Nginx镜像
> 本地编辑nginx配置文件 default.conf 绝对路径为（/home/tanhui/composeTest/nginx/conf/default.conf）
文件内容如下

```
	server {
	    listen       80;
	    server_name  localhost;
	
	    location / {
	        root   /var/www/html;
	        index  index.html index.htm index.php; # 增加index.php
	    }
	
	    #error_page  404              /404.html;
	
	    # redirect server error pages to the static page /50x.html
	    #
	    error_page   500 502 503 504  /50x.html;
	    location = /50x.html {
	        root   /var/www/html;
	    }
	    location ~ \.php$ {
	        root           /var/www/html; # 代码目录
	        fastcgi_pass   phpfpm:9000;    # 修改为phpfpm容器
	        fastcgi_index  index.php;
	        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name; # 修改为$document_root
	        include        fastcgi_params;
	    }
	}
```
> 运行容器

	docker run -d --link php-with-mysql:phpfpm --volumes-from php-with-mysql -p 80:80 -v /home/tanhui/composeTest/nginx/conf/default.conf:/etc/nginx/conf.d/default.conf --name nginx-php --privileged=true  nginx
	
> 参数解析
	
	--link php-with-mysql:phpfpm 将php容器链接到nginx容器里来，phpfpm是nginx容器里的别名。
	--volumes-from php-with-mysql 将php-with-mysql 容器挂载的文件也同样挂载到nginx容器中
	-v /home/tanhui/composeTest/nginx/conf/default.conf:/etc/nginx/conf.d/default.conf  将nginx 的配置文件替换，挂载本地编写的配置文件
	
	docker exec -it nginx-php bash
	root@32de01dbee49:/# cd /var/www/html/&&ls
	index.php  mysql.php  testmysql.php  webview
	
> 我们可以看到挂载在php－mysql容器里的文件夹同样也被挂载在nginx容器里，这时在本机方案127.0.0.1/mysql.php,数据库中的数据就在浏览器上输出了。
> 这样 nginx＋php＋mysql 的连接就基本完成了。

###5 docker-compose

接下来介绍另一种方法`docker-compose`

上面介绍了用纯docker 命令启动容器，链接容器，但是每次启动容器时都要填写一堆参数，难免容易出错，出错了之后还要删除该容器才能重新跑。接下来就介绍一下 docker-compose.

一个使用Docker容器的应用，通常由多个容器组成。使用Docker Compose，不再需要使用shell脚本来启动容器。在配置文件中，所有的容器通过services来定义，然后使用docker-compose脚本来启动，停止和重启应用，和应用中的服务以及所有依赖服务的容器。

> 首先安装docker-compose

	// 安装pip
	sudo yum install epel-release
	sudo yum install -y python-pip
	//使用pip 安装
	sudo pip install -U docker-compose

> Compose的默认配置文件是docker-compose.yml。让我们看看下面这个文件：

	nginx:
	  build: ./nginx
	  ports:
	    - "80:80"
	  links:
	    - "phpfpm"
	  volumes:
	    - /home/tanhui/composeTest/code/:/var/www/html/
	    - /home/tanhui/composeTest/nginx/conf/default.conf:/etc/nginx/conf.d/default.conf
	phpfpm:
	  build: ./phpfpm
	  ports:
	    - "9000:9000"
	  volumes:
	    - ./code/:/var/www/html/
	  links:
	    - "mysql"
	mysql:
	  build: ./mysql
	  ports:
	    - "3306:3306"
	  volumes:
	    - /home/tanhui/composeTest/mysql/data/:/var/lib/mysql/
	  environment:
	    MYSQL_ROOT_PASSWORD : 123456
	    
> 以上的配置文件路径有绝对路径，有相对路径，build 参数代表该路径下的Dockerfile
先看一下大概的目录结构

	$ tree composeTest
	composeTest
	├── code
	│   ├── index.php
	│   ├── mysql.php
	│   └── testmysql.php
	│       
	├── docker-compose.yml
	├── index.php
	├── mysql
	│   ├── data
	│   │   ├── auto.cnf
	│   │   ├── ibdata1
	│   │   ├── ib_logfile0
	│   │   ├── ib_logfile1
	│   │   ├── mysql [error opening dir]
	│   │   ├── performance_schema [error opening dir]
	│   │   └── test_db [error opening dir]
	│   └── Dockerfile
	├── nginx
	│   ├── conf
	│   │   └── default.conf
	│   └── Dockerfile
	└── phpfpm
	    └── Dockerfile
	
	10 directories, 23 files
	
	// Dockerfile 如下
	$ cat composeTest/mysql/Dockerfile
	FROM mysql:5.6
	
	# tanhui @ localhost in ~ [20:57:51]
	$ cat composeTest/phpfpm/Dockerfile
	 FROM  php:5.6-fpm
	 RUN apt-get update && apt-get install -y \
	 libfreetype6-dev \
	 libjpeg62-turbo-dev \
	 libpng12-dev \
	 vim \
	 && docker-php-ext-install pdo_mysql \
	 && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
	 && docker-php-ext-install gd \
	 
	# tanhui @ localhost in ~ [20:58:19]
	$ cat composeTest/nginx/Dockerfile
	FROM nginx:latest
	RUN apt-get update && apt-get install -y vim
	
> 现在运行这三个容器只要使用命令  docker-compose up -d
