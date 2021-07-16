---
layout:     post
title:      gitlab webhooks的使用
subtitle:   webhooks
date:       2021-7-16
author:     endy
header-img: upload/20180104163301355.jpg
catalog: true
tags:
    - git webhooks
    - php
---

### 介绍

> 利用gitlab的webhooks的功能, 实现代码自动部署, 下面以develop为例

### 用户www用户创建ssh公钥


```shell
vi /etc/passwd  # 把www后面的 /sbin/nologin   改为 /bin/bash   
www:x:1000:1000:www:/home/www:/bin/bash
vi /etc/sudoers  # 把www设置有sudo的权限
www    ALL=(ALL) NOPASSWD:ALL

# 设置完上面切换到www用户生成ssh公钥, 并把生成的公钥配置到gitlab
ssh-keygen -t rsa -C "test@mail.com"  # 直接全部回车, 这样就pull的时候不用输密码
cat ~/.ssh/id_rsa.pub 
```
> 把打印出来的公钥,放到gitlab  ssh keys 配置上即可
![](/upload/5k6ea7idcihmlz1j8fua5ddervnmdwts1.png)
![](/upload/5k6ea7idcihmlz1j8fua5ddervnmdwts.png)

### clone代码并切换到dev分支

```shell
cd /www
git clone git@0.0.0.0.0:xxx/test.git   #克隆下来是默认分支
git fetch origin develop:develop  #获取develop分支
git checkout develop
git pull origin develop   # 不提示报错,应该就没问题了


# 改为root用户为禁止登录
vi /etc/passwd  # 把www后面的 /bin/bash   改为 /sbin/nologin
www:x:1000:1000:www:/home/www:/sbin/nologin
```

### 配置webhooks
![](/upload/5k6ea7idcihmlz1j8fua5ddervnmdwts.png)

### 编写接收webhooks请求的脚步
> 脚本内容很简单, 需要加ip限制, 和记录日志的可独自添加

```shell
<?php

error_reporting(1);

// 刚才上面www用户克隆出来的web目录
$web_path = '/www/test';


//作为接口传输的时候认证的密钥, gitlab上面配置的token
$valid_token = '999999'; 

//调用接口被允许的ip地址
// $valid_ip = array('192.168.14.2','192.168.14.1','192.168.14.128');
// $client_ip = $_SERVER['REMOTE_ADDR'];

$json_content = file_get_contents('php://input');
$data = json_decode($json_content, true);

// 判断token是否正确
if (empty($_SERVER['HTTP_X_GITLAB_TOKEN']) || $_SERVER['HTTP_X_GITLAB_TOKEN'] != $valid_token) {
    exit('token异常');
}

// 判断是否是测试测试分支
if ($data['ref'] == 'refs/heads/develop') {


    $cmd = "cd $web_path && /usr/bin/git pull origin develop";
    echo $cmd;
    $a = exec($cmd, $out, $status);
    var_dump($a, $out, $status);
}
```

### 注意

1. PHP需要修改配置, 禁用函数里面把exec函数去除