---
layout:     post
title:      Linux安装php扩展
subtitle:   php扩展
date:       2020-2-7
author:     endy
header-img: upload/20180104163301355.jpg
catalog: true
tags:
    - 扩展
    - php
---

### 介绍

> Linux下安装php扩展,以memcache为例

### 下载

> [可以在php官方扩展搜索](http://pecl.php.net/package-search.php)

```shell
cd /usr/local/src  #进入软件包存放目录
wget http://pecl.php.net/get/memcache-2.2.6.tgz  #下载  （在这里下载 http://pecl.php.net/package-search.php）
tar zxvf memcache-2.2.6.tgz  #解压
cd memcache-2.2.6  #进入安装目录
```

### 运行配置
> - 运行php安装目录下的phpize文件，这时候会在extension目录下生成相应的configure文件。
> - 运行配置，如果你的服务器上只是装了一个版本的php则不需要添加--with-php-config 。后面的参数只是为了告诉phpize要建立基于哪个版本的扩展。

```shell
/usr/local/php/bin/phpize   #用phpize生成configure配置文件（替换成自己的phpize目录）
./configure  --with-php-config=/usr/local/php5/bin/php-config  #配置（替换成自己的php-config目录）
```

### 编译安装
```shell
make  #编译
make install #安装
```
### 添加配置
> 配置php支持memcache

```shell
vi /etc/php.ini  #编辑配置文件，在最后一行添加以下内容（替换自己的目录）
extension="memcache.so"
```

### 重启服务

```shell
service php-fpm restart
```