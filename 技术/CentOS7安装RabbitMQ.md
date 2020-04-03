# CentOS7 安装 RabbitMQ

### 安装Erlang 22.x
	# In /etc/yum.repos.d/rabbitmq-erlang.repo
	[rabbitmq-erlang]
	name=rabbitmq-erlang
	baseurl=https://dl.bintray.com/rabbitmq-erlang/rpm/erlang/22/el/7
	gpgcheck=1
	gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
	repo_gpgcheck=0
	enabled=1

	yum install erlang

### 安装RabbitMQ
	rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
	wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.3/rabbitmq-server-3.8.3-1.el7.noarch.rpm
	yum install rabbitmq-server-3.8.3-1.el7.noarch.rpm

### 启动RabbitMQ
	systemctl start rabbitmq-server.service
	systemctl enable rabbitmq-server.service

### 安装管理插件
	rabbitmq-plugins enable rabbitmq_management

### 添加管理账号
	rabbitmqctl add_user admin user_pass
	rabbitmqctl set_user_tags admin administrator
	rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"

### 修改防火墙规则
	firewall-cmd --zone=public --permanent --add-port=15672/tcp
	firewall-cmd --reload