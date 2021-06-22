# Centos7安装RabbitMQ

检查是否安装过`erlang`

```shell
yum list | grep erlang
```

卸载`erlang`

```shell
yum -y remove erlang-*
rm -rf /usr/lib64/erlang
```

安装`erlang`

```shell
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
yum install -y erlang
```

检查`erlang`版本

```shell
erl
```

安装`socat`

```shell
yum -y install epel-release
yum -y install socat
```

安装`RabbitMQ`

安装前可以查看官方文档的要求以及安装包的下载地址：[Installing on RPM-based Linux (RedHat Enterprise Linux, CentOS, Fedora, openSUSE)](https://www.rabbitmq.com/install-rpm.html)

```shell
# import the new PackageCloud key that will be used starting December 1st, 2018 (GMT)
rpm --import https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey

# import the old PackageCloud key that will be discontinued on December 1st, 2018 (GMT)
rpm --import https://packagecloud.io/gpg.key

curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash

rpm --import https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc

# 下载系统对应的RabbitMQ版本
# RPM for RHEL Linux 8.x, CentOS 8.x, Fedora 28+ (supports systemd)
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.9/rabbitmq-server-3.8.9-1.el8.noarch.rpm

# RPM for RHEL Linux 7.x, CentOS 7.x, Fedora 24+ (supports systemd)
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.9/rabbitmq-server-3.8.9-1.el7.noarch.rpm

# RPM for RHEL Linux 6.x, CentOS 6.x, Fedora prior to 19
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.9/rabbitmq-server-3.8.9-1.el6.noarch.rpm

# 安装(我的是CentOS 7.x系统)
rpm -ivh rabbitmq-server-3.8.9-1.el7.noarch.rpm
```

启用管理平台插件，启用插件后，可以可视化管理RabbitMQ

```shell
rabbitmq-plugins enable rabbitmq_management
```

启动/关闭/重启`RabbitMQ`

```shell
# CentOS 7.x
systemctl start/stop/restart rabbitmq-server

# CentOS 6.x
service strat/stop/restart rabbitmq-server
```

RabbitMQ配置

```shell
# 创建用户
rabbitmqctl add_user admin 123456

# 设置用户为超管
rabbitmqctl set_user_tags admin administrator

# 授权远程访问
rabbitmqctl set_permissions -p / admin "." "." ".*"

# 创建完成后重启RabbitMQ
systemctl restart rabbitmq-server
```

访问地址: 127.0.0.1:15672	账号(admin)	密码(123456)

更多请阅读官方文档: [https://www.rabbitmq.com/install-rpm.html](https://www.rabbitmq.com/install-rpm.html)