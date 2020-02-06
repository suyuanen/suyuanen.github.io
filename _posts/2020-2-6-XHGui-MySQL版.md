---
layout:     post
title:      XHGui（MySQL版）的安装、配置和使用
subtitle:   xhgui
date:       2020-2-6
author:     endy
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - xhgui
---

## 前言

本文介绍XHGui（MySQL版）的安装、配置和使用。

XHGui基于XHProf，但是较XHpro更加便捷直观，因为它不需要修改项目代码，而且以图形化方式显示结果。

### 1. 安装XHprof
#### 1.1 获取XHprof源码
对于本地开发环境来说，进行性能分析xdebug是够用了，但如果是线上环境的话，xdebug消耗较大，配置也不够灵活，因此线上环境建议使用xhprof进行PHP性能追踪及分析。

下载xhprof，我们这里选择的是通过git clone的方式，当然你也可以从 http://pecl.php.net/package/xhprof 这里下载。

cd /usr/local/src
git clone https://github.com/phacility/xhprof.git
 



注意：
php5.4及以上版本不能在pecl中下载，不支持。需要在github上下载hhttps://github.com/phacility/xhprof.git。
另外xhprof已经很久没有更新过了，截至目前还不支持php7，php7可以试使用https://github.com/tideways/php-profiler-extension。

#### 1.2 安装XHprof
```
cd xhprof/extension
/usr/local/php5.6/bin/phpize
./configure --with-php-config=/usr/local/php5.6/bin/php-config --enable-xhprof
make
make install
```


最后如果出现类似的提示信息，就表示编译安装成功
```
stalling shared extensions:     /usr/local/php-5.6.14/lib/php/extensions/no-debug-non-zts-20131226/
```


#### 1.3 配置XHprof
修改配置文件php.ini，在最后增加如下配置
```
[xhprof]

extension=xhprof.so

xhprof.output_dir=/www/xhprof/output
```


重启php-fpm后通过phpinfo查看，或者在命令行通过php -m | grep xhprof查看是否安装成功。

注意：
需要创建output_dir
mkdir -p /www/xhprof/output

### 2. 安装XHGui
#### 2.1 获取XHGui源码
使用git工具克隆XHGui（MySQL版）到本地：



git clone https://github.com/preinheimer/xhprof.git



当然，也可以在github上下载源码压缩包，再在本地解压。

假设下载后XHGui源码地址为:/usr/local/nginx/html/xhgui.yourdomain.com。

#### 2.2 Nginx配置
因为XHGui的数据要显示在浏览器上，所以必须配置一个能够访问的地址。

在服务器上新增一个站点，指向XHGui源码下面的xhprof_html目录。

Nginx配置如下：
```
server {

    listen 80;
    server_name xhgui.yourdomain.com;

    location / {

        root /usr/local/nginx/html/xhgui.yourdomain.com/xhprof_html;

        index index.html index.htm index.php;

        if ( !-e $request_filename ) {

            rewrite ^/(.*)? /index.php?/$1 last;

            break;
        }
    }

    location ~ \.php$ {
    root /usr/local/nginx/html/xhgui.yourdomain.com/xhprof_html;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

    error_log /www/logs/xhgui.yourdomain.com_error.log;
    access_log /www/logs/xhgui.yourdomain.com_access.log main;
}
```

 
#### 2.3  配置XHProf
复制文件 xhprof_lib 目录下的 config.sample.php 为config.php。编辑 config.php 文件，进行配置。

配置数据库和URL选项：
```
$_xhprof['dbhost'] = '127.0.0.1';
$_xhprof['dbuser'] = 'root';
$_xhprof['dbpass'] = '123456';
$_xhprof['dbname'] = 'xhprof';
$_xhprof['url'] = 'http://xhgui.yourdomain.com';
```

对于开发环境，设置IP控制为false，并将其他行注释，如下：

```
$controlIPs = false; //Disables access controlls completely.
//$controlIPs = array();
//$controlIPs[] = "127.0.0.1"; // localhost, you'll want to add your own ip here
//$controlIPs[] = "::1"; // localhost IP v6
```


#### 2.4 导入数据库
在MySQL中新建一个名为 xhprof 的数据库，用如下的语句创建一个 details 表：
```mysql
CREATE TABLE `details` (
    `id` char(17) NOT NULL,
    `url` varchar(1000) default NULL,
    `c_url` varchar(1000) default NULL,
    `timestamp` timestamp NOT NULL default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP,
    `server name` varchar(64) default NULL,
    `perfdata` BLOB,
    `type` tinyint(4) default NULL,
    `cookie` BLOB,
    `post` BLOB,
    `get` BLOB,
    `pmu` int(11) unsigned default NULL,
    `wt` int(11) unsigned default NULL,
    `cpu` int(11) unsigned default NULL,
    `server_id` char(3) NOT NULL default 't11',
    `aggregateCalls_include` varchar(255) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `url` (`url`),
    KEY `c_url` (`c_url`),
    KEY `cpu` (`cpu`),
    KEY `wt` (`wt`),
    KEY `pmu` (`pmu`),
    KEY `timestamp` (`timestamp`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```


要获取最新语句，请参考XHGui源码下 xhprof_lib/utils/xhprof_runs.php 文件大约 109行的内容。

### 3. 被分析网站配置
打开需要分析的网站的Nginx配置文件，加入如下一行：
```
location ~ .*\.(php|php5)?$ {
    #...
    fastcgi_param PHP_VALUE "auto_prepend_file=/usr/local/nginx/html/xhgui.yourdomain.com/external/header.php";
}
```


如果被分析网站用的是Apache，则这样配置：
```
<VirtualHost *:80>
    ...
    php_admin_value auto_prepend_file "/usr/local/nginx/html/xhgui.yourdomain.com/external/header.php"
    ...
</VirtualHost>
```
这样header.php文件会在目标脚本执行之前自动解析执行。

简单分析一下代码header.php，关健代码：
```
// 开始分析
xhprof_enable();

// 运行一些函数
foo();

// 停止分析，得到分析数据
$xhprof_data = xhprof_disable();
$xhprof_data中记录了程序运行过程中所有的函数调用时间及CPU内存消耗，具体记录哪些指标可以通过xhprof_enable的参数控制，目前支持的参数有：
```
HPROF_FLAGS_NO_BUILTINS 跳过所有内置（内部）函数。
XHPROF_FLAGS_CPU 输出的性能数据中添加 CPU 数据。
XHPROF_FLAGS_MEMORY 输出的性能数据中添加内存数据
之后的处理已经与xhprof扩展无关，大致是编写一个存储类XHProfRuns_Default，将$xhprof_data序列化并保存到某个目录，可以通过XHProfRuns_Default(__DIR__)将结果输出到当前目录，如果不指定则会读取php.ini配置文件中的xhprof.output_dir，仍然没有指定则会输出到/tmp。

xhprof_html/index.php将记录的结果整理并可视化，默认的UI里列出了：

funciton name ： 函数名
calls: 调用次数
Incl. Wall Time (microsec)： 函数运行时间（包括子函数）
IWall%：函数运行时间（包括子函数）占比
Excl. Wall Time(microsec)：函数运行时间（不包括子函数）
EWall%：函数运行时间（不包括子函数）


### 4. 开始分析
问网站地址，假设要分析的网站是：localhost，则通过GET传入_profile=1变量，也就是访问：

http://xhgui.yourdomain.com?_profile=1
这样网站加载完成的同时，会把分析数据保存在MySQL数据库中。
再访问分析结果地址:

http://xhgui.yourdomain.com/index
就会看到如下的报告结果，点击对应的TIMESTAMP就能看到详细的报告。



默认XHProf UI不会对PHP应用收集分析数据，在请求任意URL时，需添加GET参数_profile=1来启用。
external/header.php脚本会检查_profile参数，并将参数值写到cookie中setcookie('_profile',$_GET['_profile']);，这样就不用每次请求都带GET参数_profile=1，并且cookie是针对域名的，这样也就同域名下的其他URL请求启用了性能分析,然后对目标URL去掉参数_profile后发起重定向。对于不带GET参数_profile的URL请求，header.php会继续检查是否存在名为_profile的cookie，如果存在且值为布尔真，则设置条件变量启用性能分析，否则不启用。
若想要对已启用性能分析的域名禁用性能分析，则可以通过对URL请求添加GET参数_profile=0来禁用，因为header.php在检查cookie时发现_profile值为布尔假（0），所以不会启用性能分析。

### 5. 图形化支持
在报告详情页面，有一个按钮“View Callgraph”，点击可以看到方法的调用关系，以及花费时间最多（红色）的方法，但是需要安装graphviz和libpng（若php版本是5.6的，强烈建议使用：graphviz-2.24.0 + libpng-1.5.19，亲测可行）。

安装方法：

 apt  install graphviz


安装完成后，会生成文件：/usr/bin/dot，编辑config.php文件，确保 $_xhprof['dot_binary'] 的值是这个文件的位置。

然后点击“View Callgraph”，就能看到调用关系图了。如下。

![](https://img-blog.csdn.net/20180104175720168)


## 6. 术语说明
在查看 Xhprof 或 XHGUI 性能数据时，会遇到以下几个术语，其含义对应如下：

Calls / CallCount：函数被调用的次数
Incl. Wall Time / Wall Time：执行该函数（包括子函数）耗费的时间
Incl. MemUse / Memory Usage：该函数（包括子函数）占用的内存
Incl. PeakMemUse / Peak Memory Usage：函数（包括子函数）占用内存的峰值
Incl. CPU / CPU：执行该函数（包括子函数）花费的CPU时间
Excl. Wall Time / Exclusive Wall Time：函数本身（不包括子函数）耗费的时间
Excl. MemUse / Exclusive Memory Usage：函数本身（不包括子函数）占用的内存
Excl. PeakMemUse / Exclusive Peak Memory Usage：函数本身（不包括子函数）耗费内存的峰值
Exclusive CPU：函数本身（不包括子函数）花费的CPU时间
Inclusive简写Incl，表示测量到的数据是函数本身及所有调用的子函数总共耗费占用的资源。
Exclusive简写Excl，则表示不包含调用的子函数耗费占用的资源。

另外，所有测量值都是每个函数调用在次数上的叠加。

参考资料：

>- [Xhprof安装与使用](http://blog.xiayf.cn/2015/09/15/xhprof-installation-and-usage/)
>- [PHP性能优化工具–xhprof安装](http://www.chenglin.name/php/optimization/439.html)
>- [How To Set Up XHProf and XHGui for Profiling PHP Applications on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-xhprof-and-xhgui-for-profiling-php-applications-on-ubuntu-14-04)
>- [歪麦博客](https://www.awaimai.com/)


> 注意事项：

1、如果不安装 Graphviz，点击链接“View Full Callgraph”，会报如下错误：
failed to execute cmd: " dot -Tpng". stderr: Format: "png" not recognized. Use one of: canon cmap cmapx cmapx_np dot eps fig gv imap imap_np ismap pic plain plain-ext pov ps ps2 svg svgz tk vml vmlz xdot xdot1.2 xdot1.4 '

2、如果看到有提示：Warning: date(): It is not safe to rely on the system's timezone settings. You are *required* to use t...的错误，那么，试着在：/usr/local/nginx/html/xhgui.yourdomain.com/external/header.php文件处的第二行添加：
date_default_timezone_set('PRC');

3、如果nginx的站点域名配置完成并重启nginx后， 被测试站点一言不合就给你来个：502 Bad Gateway。那么，还是这个文件：/usr/local/nginx/html/xhgui.yourdomain.com/external/header.php 文件底下（不是最底下）找到这个方法：
call_user_func($_xhprof['ext_name'].'_enable', $flagsCpu , $flagsMemory);更改成：call_user_func($_xhprof['ext_name'].'_enable', XHPROF_FLAGS_NO_BUILTINS | $flagsCpu | $flagsMemory);
