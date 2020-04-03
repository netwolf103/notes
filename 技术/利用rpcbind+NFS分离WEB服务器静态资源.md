# 利用rpcbind+NFS分离WEB服务器静态资源

### YUM源配置 & 更新
	yum -y install epel-release
	yum -y install yum-utils
	yum update

### 主WEB服务器NFS安装(假定IP192.168.0.2)
	# NFS 安装
	yum -y install nfs-utils
	systemctl enable rpcbind
	systemctl start rpcbind
	systemctl enable nfs
	systemctl start nfs
	firewall-cmd --zone=public --permanent --add-service={rpc-bind,mountd,nfs}
	firewall-cmd reload


	vi /etc/exports
	# IP地址可为(*)全部，IP网段，具体IP
	/var/www/html/img 192.168.0.3(rw,sync,no_root_squash,no_all_squash)
	systemctl restart nfs
	# 测试
	showmount -e localhost

### IMG WEB服务器NFS安装(假定IP192.168.0.3)
	yum -y install nfs-utils
	systemctl enable rpcbind
	systemctl start rpcbind
	# 测试
	showmount -e 192.168.0.2
	mount -t nfs 192.168.0.2:/var/www/image /var/www/html/img
	vim /etc/fstab
	192.168.0.2:/var/www/img /var/www/html/img nfs defaults 0 0

### Nginx安装
	yum install nginx
	systemctl enable nginx
	systemctl start nginx
	firewall-cmd --permanent --zone=public --add-service=http
	firewall-cmd --permanent --zone=public --add-service=https
	firewall-cmd --reload


	# 配置Vhosts
	vim /etc/nginx/conf.d/vhosts.conf
	server {
	    listen 80;
	    server_name img.example.com;
	    access_log /var/log/nginx/access.img.example.com.log main;
	    error_log /var/log/nginx/error.img.example.com.log;
	    root /var/www/html;

	    error_page 404 /404.html;
	        location = /40x.html {
	    }

	    error_page 500 502 503 504 /50x.html;
	        location = /50x.html {
	    }

	    location ~ .*\.(css|js|svg|ico|gif|jpe?g|png|bmp|swf)$ {
	        expires 30d;
	        add_header Pragma public;
	        add_header Cache-Control "public";
	        break;
	    }
	    
	    location / {
	        index index.html index.htm;
	    }
	}

	# 开启GZIP压缩
	vim /etc/nginx/conf.d/gzip.conf
	gzip on;
	gzip_disable "msie6";
	gzip_comp_level 6;
	gzip_min_length 1100;
	gzip_buffers 16 8k;
	gzip_proxied any;
	gzip_types text/plain text/css text/js text/xml text/javascript application/javascript application/json application/xml application/rss+xml image/svg+xml;

### HTTPS证书安装
	yum-config-manager --enable rhui-REGION-rhel-server-extras rhui-REGION-rhel-server-optional
	yum install certbot python2-certbot-apache
	certbot --nginx
	systemctl restart nginx