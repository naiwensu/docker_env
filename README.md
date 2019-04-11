# Docker相关文档


# Winodws、MAC的docker配置文档

### 1. 安装
安装Docker for Windows 或 Dokcer for Mac。网盘里或者自己去docker官网下载。

具体可以参考`文档`文件夹里的`docker从入门到实践.pdf`。

要注意的是你的`localhost`所能访问的文件夹是和`devdocker`平级的`wwwroot`文件夹。也就是典型的目录结构是：

	./devdocker/ ./wwwroot/

然后`./wwwroot/`里是：

	./xy/ ./web/ ./earth/ 等

注意1：如果你已经安装xampp，请把`devdocker`放到`htdocs`平级的地方，也就是`c:\xampp`目录底下。然后你看你的方便，是改`.env`文件里的`APPLICATION`位置还是创建`wwwroot`文件夹（建议用后者）。如果你没有安装xampp，那么理论上你可以随便放。

注意2：在Windows系统下需要新建文件夹`devdocker`，然后将本仓库clone到下级目录。

### 2. 配置加速器镜像
右击托盘上的 小鲸鱼 ，选中 Settings选项，会打开设置窗口，选中 Daemon ，在 Registry mirrors的输入框内，输入：
	
	https://urpvm08q.mirror.aliyuncs.com
	https://registry.docker-cn.com
最后点击 Apply。

注意：此处最好使用下文Linux部分中Chapter 9验证镜像是否可用。
    
### 3. 跑起来我们的镜像devdocker

（注：运行前，请确认类似 xampp 的软件已关闭 80、3306、6379端口未被占用）

（注：执行过程中，docker程序会弹出弹窗，询问是否共享，请选择 share it 选项）


### 4.增加域名映射(laravel开发与TP共存需要)   
    1. 开发环境（本地）:修改host文件（便于nginx访问）
        127.0.0.1       www.dev-tp.com
        127.0.0.1       www.dev-crm.com   
    2. 测试环境与生产环境: 需要修改nginx配置文件，增加二级域名映射 ##TODO 


请执行下文Linux文档中的4、11步骤。（可以使用cmd或者windwos shell或者git bash）

# Linux的docker配置文档

其他系统的安装请查阅：https://www.gitbook.com/book/yeasy/docker_practice/details

### 1. 查看系统版本

   	cat /etc/os-release

### 2. 安装Git
    ****
   	yum -y install git

### 3. 配置Git公钥
1. 设置用户名和邮箱（下面dev处可以换成自己的名字、邮箱）
		
       	git config --global user.name "dev"
       	git config --global user.email "dev@xingyunbooks.com"
	
2. 生成秘钥和公钥（默认目录和空密码，三个回车）（下面dev处可以换成自己的名字、邮箱）
    
        ssh-keygen -t rsa -C " dev@xingyunbooks.com"

3. 查看公钥并复制到阿里git仓库的ssh key中

        cat ~/.ssh/id_rsa.pub

### 4. 将docker-compose file克隆到本地
因习惯，将其克隆到 `/data /` 目录下。

注意：如果是开发环境下，请把`devdocker`放到`htdocs`平级的地方，也就是`c:\xampp`目录底下。
	
	mkdir /data/	
	cd /data/
	git clone git@code.aliyun.com:xy/docker_env.git

### 5. 安装依赖，配置国内的 yum 源，更新yum软件源缓存
docker的前置安装软件
	
	yum install -y yum-utils device-mapper-persistent-data lvm2
	yum-config-manager --add-repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
	yum makecache fast

### 6. 安装 docker
	yum install -y docker-ce            //	若有询问，全部填 y

### 7. TODO：建立 docker 用户组，并把用户添加进docker用户组。

### 8. 添加国内docker镜像加速器，加载配置，并重启docker
	systemctl restart docker
	
	//下面这一段（包含结束于EOF）要一起复制粘贴：
	tee /etc/docker/daemon.json <<-'EOF'
	{
  		"registry-mirrors": ["https://registry.docker-cn.com","https://urpvm08q.mirror.aliyuncs.com"]
	}
	EOF


	systemctl daemon-reload
	systemctl restart docker
### 9. 可选：检查加速器是否生效
配置加速器之后，如果拉取镜像仍然十分缓慢，请手动检查加速器配置是否生效，在命令行执行
	
	docker info
如果从结果中看到了如下内容，说明配置成功。
	
	Registry Mirrors:
	https://registry.docker-cn.com/

注意：此处可能出现```ERROR:could not read CA certificate```，参照[Windows系统解决方法](https://blog.csdn.net/qq_35852248/article/details/80925154)，Linux系统使用```unset ${!DOCKER*}```重置即可。

### 10. 安装docker-compose 
神奇的问题是下面这句话第一遍一般超时，第二遍才好
	
	sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose;
	sudo chmod +x /usr/local/bin/docker-compose;
	docker-compose --version;

### 11. 运行Laradock
进入docker环境目录

	cd devdocker/
下面的运行会出现warning。（有黄色的、红色的）可以忽略。
第一次运行需要安装环境，需要比较久的时间，请耐心等待
	
	docker-compose up -d nginx mysql redis

注意：这一步安装MySQL应当卸载系统中原有的MySQL以避免冲突。

至此，docker已经安装完毕。接下来开始部署。
### 12. 准备部署脚本

	cd ~;
	rm -rf ~/autoshell/;
	git clone git@code.aliyun.com:xy/auto_shell.git
	chmod u+x ./auto_shell/*;
	cd ~/auto_shell/;

### 13. 运行部署脚本
ALL OVER。




# docker相关知识、命令快速说明

## 启动laradock
可以根据自己需要自行启动 nginx/apache/mysql/phpmyadmin/redis 等

注：

	phpmyadmin 请访问 http://IP:88
	pgadmin请访问 http://IP:5050



## 常用操作
以下操作请确保在laradock 根目录下

#### 启动相关
laradock 默认会启动 php-fpm 和 workspace ，所以参数中无需加这两个。

后台启动
	
	docker-compose up -d xxxxxx

只重启nginx （比如修改了配置文件）
	
	docker-compose restart nginx

停止所有
	
	docker-compose stop

重新构建相应的容器

	docker-compose build nginx worksapce

#### 工作空间

进入工作空间前，请确认环境已经启动
	
	docker-compose exec workspace bash

会进入 /var/www 目录

此时 可以执行composer 和PHP命令。

如果之前env-example 开启了node和yarn 也可执行对应命令。


#### 连接数据库和PHP
请一定注意，数据库连接地址请一定填写为mysql、postgres、mariadb 等。

另外Nginx/Caddy/Apache 如果需要访问PHP容器，请填写:php-fpm

#### 更改laradock 配置
当你再次修改完env-example 后，请一定按照如下方法执行：
	
	cp env-example .env
重新构建相应的容器

	docker-compose build nginx worksapce

如果还修改了 其他容器配置，请在后面一同加上

需要注意的是：

由于数据库的数据是映射到宿主机目录的，所以在env-example中修改数据库密码，即使重新构建也无效。如需强制更改请删除数据存储处里面对应数据库的数据。

日常修改密码，请使用phpmyadmin 或者 pgadmin


# 特别说明（以下基本不用看，因为都已经设置好了)

### 1. 数据库相关：

由于容器之间的互联，数据库配置中的 “DB_HOST”值只能填mysql。

数据库默认账号密码：账号-root；密码-root。

在数据库中新建用户时，用户的登录授权端，需要填%，否则不能没有权限登录

	CREATE USER 'xyName342h'@'%' IDENTIFIED BY 'fds77GHkzH8';

TODO：可以尝试当%换成mysql域会如何？

### 2. 命令解析（以下所有命令都必须在docker配置文档的根目录执行）

1. 进入容器内部
	
		docker-compose exec [mysql/reids/nginx/workspace] bash

2. 操作容器

每次修改了容器的配置文件，都需要重新构建容器，并重新启动

构建：（不指定容器，默认构建全部）

	docker-compose build nginx mysql redis

后台运行：下面的命令，如果指定的容器尚未构建，则会先构建，再启动

	docker-compose up –d nginx

启动容器（不管有这个容器有没有构建，都会启动，但是只有存在实体的容器会被真正启动）。推荐使用 up –d 运行容器
	
	docker-compose start nginx php-fpm workspace

停止容器
	
	docker-compose stop nginx mysql redis

其他命令：可在 http://laradock.io/ 查看

### 3. 文件位置说明（`devdocker`表示docker配置的根目录）

1. Nginx 配置文件 ：`devdocker`\nginx\sites\default.conf
2. MySQL 配置文件：`devdocker`\mysql\my.cnf
3. PHP 配置文档：`devdocker`\php-fpm\php**.ini（由.env文件中的PHP_VERSION=56 确定选择的配置文件）

### 4. `.env` 配置文件说明（如果没有，可以 `cp env-example .env` 复制一份）
1. 设置项目的映射目录，以当前文件所在目录为基础目录。以下设置的意思是：将docker配置文件夹同级的所有文件，都映射到容器中，实现共享
	
		APPLICATION=../
特别说明：nginx 的default.conf 文件，决定nginx 默认访问的路径。
		
		server {
		    listen 80 default_server;
		    listen [::]:80 default_server ipv6only=on;
		    server_name localhost;
		    root /var/www;（表示上面APPLICATION 映射的目录，只可向后添加路径）
		    index index.php index.html index.htm;
			。。。。。。
		}

2. 设置 PHP 的扩展
在 PHP_FPM 中，将根据想要的扩展设置为true
例如：PHP_FPM_INSTALL_PHPREDIS=true

3. 设置 NGINX 映射端口
指定宿主机映射到容器内的接口；其他接口设置，都是如此
	
		NGINX_HOST_HTTP_PORT=80

4. 设置时区
		
		WORKSPACE_TIMEZONE=Asia/Shanghai


 
# 参考文献：
1. <https://zhuanlan.zhihu.com/p/27182216>
