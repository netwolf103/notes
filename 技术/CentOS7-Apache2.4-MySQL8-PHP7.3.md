# LAMP - CentOS7+Apache2.4+MySQL8+PHP7.3环境搭建

### YUM源配置 & 更新
	yum -y install epel-release
	yum -y install yum-utils
	yum update

### Apache安装
	yum -y install httpd
	systemctl enable httpd
	systemctl start httpd
	firewall-cmd --permanent --zone=public --add-service=http
	firewall-cmd --permanent --zone=public --add-service=https
	firewall-cmd --reload

	# 隐藏版本号
	vim /etc/httpd/conf.d/secure.conf
	TraceEnable off
	ServerSignature Off
	ServerTokens Prod


	# 配置vhost
	vim /etc/httpd/conf.d/vhosts.conf
	<VirtualHost *:80>
	    DocumentRoot "/var/www/html"
	    ServerName example.com
	    ServerAlias www.example.com
	    ErrorLog "logs/example.com-error_log"
	    CustomLog "logs/example.com-access_log" combined
	</VirtualHost>

	# HTTPS证书安装
	yum-config-manager --enable rhui-REGION-rhel-server-extras rhui-REGION-rhel-server-optional
	yum install certbot python2-certbot-apache
	certbot --apache
	systemctl restart httpd

### MySQL8安装
	# 安装MySQL8 YUM源
	rpm -Uvh https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
	yum install mysql-community-server
	systemctl enable mysqld
	systemctl start mysqld
	# 查看自动生成的MySQL密码
	grep "password" /var/log/mysqld.log
	# 登录MySQL后修改密码
	ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
	# 刷新MySQL权限表
	FLUSH PRIVILEGES;

### PHP7.3安装
	yum-config-manager --disable remi-php54
	yum-config-manager --enable remi-php73
	yum -y install php php-cli php-fpm php-mysqlnd php-zip php-devel php-gd php-mcrypt php-mbstring php-curl php-xml php-pear php-bcmath php-json php-opcache php-redis php-soap
	# 隐藏php版本号
	vim /etc/php.ini
	expose_php = off

### 系统参数设置
	# 文件打开数设置
	ulimit -n 65535
	vim /etc/security/limits.d/nofile.conf
	* soft nofile 65535
	* hard nofile 65535
	# 内核网络优化
	vim /etc/sysctl.conf
	# 启用timewait 快速回收
	net.ipv4.tcp_tw_recycle = 1