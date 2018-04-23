---
layout: post
title: "nginx和PHP7.0 以ROOT身份启动 PDO_MYSQL mbstring"
date: 2018-04-04 19:37:27 +0800
comments: true
categories: PHP7.0
---
本篇文章主要介绍重点实用解决问题的部分

分为四个部分

1 .安装php7.0配置nginx

2 .如何以root身份启动php-fpm

3 .安装扩展 pdo_mysql mbstring

<!--more-->

### 安装php7.0配置nginx

首先安装php7.0

```bash
apt-get install php7.0-fpm
```
然后修改ngixn配置
```bash
location ~ \.php$ {
  include snippets/fastcgi-php.conf;
  # With php7.0-cgi alone:
  #fastcgi_pass 127.0.0.1:9000;
  # With php7.0-fpm:
  fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
}
```
重启nginx就可以了。

### 如何以root身份启动php-fpm

1）编辑/etc/php/7.0/fpm/pool.d/www.conf，把user和group修改为root

```bash
user = root
group = root
```

2）编辑/lib/systemd/system/php7.0-fpm.service，在ExecStart的 --nodaemonize前面加上 --allow-to-run-as-root，就像下面这样：

```bash
ExecStart=/usr/sbin/php-fpm7.0 --allow-to-run-as-root --nodaemonize...
```

3）重载配置：

```bash
systemctl daemon-reload
```

4）重启php-fpm：

```bash
service php7.0-fpm restart
```

5) 查看
```bash
ps auwx | grep php
```
就可以看到 php-fpm已经以root身份运行了

### 安装扩展 pdo_mysql mbstring

```
apt install php7.0-mysql
apt-get install php7.0-mbstring
```

然后修改php.ini 我的目录在/etc/php/7.0/fpm

找到
```
;extension=php_mbstring.dll
// 俩个位置不在一起 单独查找
;extension=php_mysqli.dll
```
把前面的分号;代表注释 给删除 然后重启php-fpm 就可以了
