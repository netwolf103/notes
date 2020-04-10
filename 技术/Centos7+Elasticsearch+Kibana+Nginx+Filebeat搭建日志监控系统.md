# Centos7 Elasticsearch + Kibana + Nginx + Filebeat 搭建日志监控系统

### 更新YUM源
	yum -y update

### Java安装
	yum -y install java-openjdk-devel java-openjdk

### 添加ELK YUM源
	cat <<EOF | sudo tee /etc/yum.repos.d/elasticsearch.repo
	[elasticsearch-7.x]
	name=Elasticsearch repository for 7.x packages
	baseurl=https://artifacts.elastic.co/packages/7.x/yum
	gpgcheck=1
	gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
	enabled=1
	autorefresh=1
	type=rpm-md
	EOF

### 添加GPG key
	rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

### 更新YUM缓存
	yum clean all
	yum makecache

### 安装 Elasticsearch
	yum -y install elasticsearch

	# 验证安装
	rpm -qi elasticsearch

	Name        : elasticsearch
	Epoch       : 0
	Version     : 7.5.0
	Release     : 1
	Architecture: x86_64
	Install Date: Sat 07 Dec 2019 05:40:08 AM UTC
	Group       : Application/Internet
	Size        : 490194716
	License     : Elastic License
	Signature   : RSA/SHA512, Tue 26 Nov 2019 03:19:16 AM UTC, Key ID d27d666cd88e42b4
	Source RPM  : elasticsearch-7.5.0-1-src.rpm
	Build Date  : Tue 26 Nov 2019 01:20:07 AM UTC
	Build Host  : packer-virtualbox-iso-1559162487
	Relocations : /usr 
	Packager    : Elasticsearch
	Vendor      : Elasticsearch
	URL         : https://www.elastic.co/
	Summary     : Distributed RESTful search engine built for the cloud
	Description :
	Reference documentation can be found at
	https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
	and the 'Elasticsearch: The Definitive Guide' book can be found at
	https://www.elastic.co/guide/en/elasticsearch/guide/current/index.htm

### 启动Elasticsearch
	systemctl enable elasticsearch
	systemctl start elasticsearch

	# 验证Elasticsearch是否启动
	curl http://127.0.0.1:9200
	{
	  "name" : "x.members.linode.com",
	  "cluster_name" : "elasticsearch",
	  "cluster_uuid" : "0LxmV-29TPW4C-QFVJYHlg",
	  "version" : {
	    "number" : "7.5.0",
	    "build_flavor" : "default",
	    "build_type" : "rpm",
	    "build_hash" : "e9ccaed468e2fac2275a3761849cbee64b39519f",
	    "build_date" : "2019-11-26T01:06:52.518245Z",
	    "build_snapshot" : false,
	    "lucene_version" : "8.3.0",
	    "minimum_wire_compatibility_version" : "6.8.0",
	    "minimum_index_compatibility_version" : "6.0.0-beta1"
	  },
	  "tagline" : "You Know, for Search"
	}

### 安装Kibana
	yum -y install kibana

	# 配置Kibana
	vim /etc/kibana/kibana.yml
	server.host: "localhost"
	server.name: "kibana.example.com"
	elasticsearch.hosts: ["http://localhost:9200"]
	csp.rules: 
	  - "script-src 'self' 'unsafe-eval' 'unsafe-inline'"
	  - "worker-src blob:"
	  - "child-src blob:"
	  - "style-src 'unsafe-inline' 'self'"
	# 启动Kibana
	systemctl enable kibana
	systemctl start kibana

### 以Nginx作为反向代理
	yum -y install nginx httpd-tools

	# 配置nginx
	htpasswd -c /etc/nginx/htpasswd.users kibanaadmin

	vi /etc/nginx/conf.d/kibana.conf
	server {
	    listen 80;

	    server_name example.com;

	    auth_basic "Restricted Access";
	    auth_basic_user_file /etc/nginx/htpasswd.users;

	    location / {
	        proxy_pass http://localhost:5601;
	        proxy_http_version 1.1;
	        proxy_set_header Upgrade $http_upgrade;
	        proxy_set_header Connection 'upgrade';
	        proxy_set_header Host $host;
	        proxy_cache_bypass $http_upgrade;        
	    }
	}

	# 启动Nginx
	systemctl enable nginx
	systemctl start nginx
	# 开启防火墙80端口
	firewall-cmd --add-port=80/tcp --permanent
	firewall-cmd --reload

### 以Apache作为反向代理
	yum -y install httpd

	# 配置httpd
	htpasswd -c /etc/httpd/conf/htpasswd.users kibanaadmin

	vi /etc/httpd/conf.d/kibana.conf
	<VirtualHost *:80>
	    ServerName kibana.example.com
	    ErrorLog "logs/kibana.example.com-error_log"
	    CustomLog "logs/kibana.example.com-access_log" combined

	    <Location />
	        AuthType Basic
	        AuthName "Restricted Content"
	        AuthUserFile /etc/httpd/conf/htpasswd.users
	        Require valid-user
	    </Location>

	    <IfModule mod_proxy.c>
		ProxyRequests On

		ProxyPass / http://localhost:9200/
		ProxyPassReverse / http://localhost:9200/
	    </IfModule>
	</VirtualHost>

### 安装Filebeat
	yum -y install filebeat

	# 启用filebeat apache配置
	filebeat modules enable apache

	# 配置apache filebeat
	vim /etc/filebeat/modules.d/apache.yml
	- module: apache
	  access:
	    enabled: true
	    var.paths: ["/var/log/httpd/*access_log*"]

	  # Error logs
	  error:
	    enabled: true
	    var.paths: ["/var/log/httpd/*error_log*"]
	    
	# 启动filebeat
	systemctl enable filebeat
	systemctl start filebeat