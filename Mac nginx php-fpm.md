mac nginx php-fpm （转） 

---------------这两天碰到的事。

毫无疑问，这两天碰到蛋疼的事。

No 1、起因 mac下的apache服务瘫痪，各种调试不起作用，最后一次log还是三天前的，给出error更是奇怪

error_log中的报错：

[Tue Sep 03 15:13:31 2013] [warn] Init: Session Cache is not configured [hint: SSLSessionCache]
httpd: Could not reliably determine the server's fully qualified domain name, using LiangdeMacBook-Air.local for ServerName
[Tue Sep 03 15:13:31 2013] [notice] Digest: generating secret for digest authentication ...
[Tue Sep 03 15:13:31 2013] [notice] Digest: done
[Tue Sep 03 15:13:31 2013] [notice] Apache/2.2.22 (Unix) DAV/2 PHP/5.3.15 with Suhosin-Patch mod_ssl/2.2.22 OpenSSL/0.9.8x configured -- resuming normal operations
[Tue Sep 03 18:58:45 2013] [notice] caught SIGTERM, shutting down

 

sites-error_log 中的报错：

[Tue Sep 03 14:36:10 2013] [error] [client ::1] PHP Warning:  strtotime(): It is not safe to rely on the system's timezone settings. You are *required* to use the date.timezone setting or the date_default_timezone_set() function. In case you used any of those methods and you are still getting this warning, you most likely misspelled the timezone identifier. We selected 'Asia/Chongqing' for 'CST/8.0/no DST' instead in /Users/liangzhongyuan/Sites/Classes/PHPExcel/Reader/Excel2007.php on line 407

 

在网上各种查资料，各种方法尝试，借鉴国内外高手的解答，可就是没解决掉。蛋疼，胸痒啊~

现在想想，还是不太确定，只能说有可能是，php-fpm没启动得了，cgi没跑起来。请求过来，apache能拿到，却解析不了。不过一般这种情况应该会报connect() failed错误。所以一直蛋疼着。

后来决定，重装apache，遂，去官网，下最新的。解压，./configure ---> make --->啪，又报错，c 编译器执行不了。哇嘎嘎，火大。

决定重装gcc，第一种方法，xcode 下载command line tools，速度太慢了。第二种方法，网上http://hi.baidu.com/chaseshu/item/15f8531083d934707b5f258d 没尝试，太烦，担心依赖包不够（有兴趣的可以试试第二种方法）。只能磨磨唧唧等第一种方法了。

装完gcc，再次编译apache，真他丫的蛋疼，还是不行。索性算了，换nginx。（期间还装了macports和howbrew，macports有pkg文件，直接运行安装。howbrew有中文官网，一句ruby就装好了。）

mac 安装 nginx

第一步：安装macports。(目的为了安装pcre)

http://www.macports.org/install.php  ，下载对应版本的pkg包，直接运行安装。

第二步：安装pcre (目的是满足nginx需要)

$ port install pcre
第三步：解压编译安装

$ tar xvzf nginx-1.2.0.tar.gz
$ cd nginx-1.2.0
$ sudo ./configure --prefix=/usr/local/nginx --with-http_ssl_module
$ sudo make
$ sudo make install
第四步：启动nginx 

$ sudo /usr/local/nginx/sbin/nginx
与linux不同，service命令没有，sudo service nginx restart 无效。

第五步：nginx加入环境变量

cd ~

sudo vim .bash_profile

export PATH="/usr/local/nginx/sbin:$PATH"  //在最前面加上这句话，保持退出后，重启终端。

现在就可以不用那么长命令了，直接在终端中输入nginx -h 就可以看到参数表。比如nginx -v 查看版本。sudo nginx -s reload 是重启nginx。

不过对于重启我更建议用 sudo  /usr/local/nginx/sbin/nginx -s reload 。 之所以写这么长命令地址，因为在实际测试中，发现直接用sudo nginx 会报找不到nginx.pid的错。只有用长命令系统才会自动生成nginx.pid。这点要注意。对于重启php-fpm也有同样建议，用sudo /usr/sbin/php-fpm。

然后，

nginx跑起来了，没有问题，很健康的和我打招呼。Welcome nginx。(配置与ubuntu下稍有不同，多个server都写在nginx.conf里面，最好自己单独将其抽离出来，我的处理是再建一个vhost-default文件，即sudo vim  vhost-default ,然后把nginx.conf里面那些server配置全部切过来，不过要记得在nginx.conf里加一句，include /usr/local/nginx/conf/vhost-default;)

 

But 配置好环境后，php文件还是运行不了。蛋疼，胸更痒~ 

估计不是nginx的问题了，应该是php或者cgi了。于是写一个小php文件，在终端直接用：$ php  index.php 运行成功，有输出。说明php解析也没有问题。

那唯一的可能就是，nginx或者apache都没有调到cgi。于是测试php-fpm，终端，$ sudo php-fpm ;啪，果然出错。php-fpm初始化失败。

原来一切症结都在这里啊~该死的php-fpm~

本想重装php-fpm，但php-fpm本是跟这php一起装来的，如果重装成本太大。先且看看到底什么问题吧~

一个：目录/private/etc/下php-fpm.conf配置文件没找到。于是到那下面，一看有个php-fpm.conf.default,确实没有.conf的，于是cp一个出来。然后重启nginx再sudo /usr/sbin/php-fpm，这次又说log目录不存在，于是又按要求在usr下面mkdir var，cd var，mkdir logs，cd logs，sudo vim php-fpm-error.log。结束。再重启，再运行php-fpm成功。占用端口127.0.0.1:9000。(有时在/usr/下建目录var不行，mkdir: var: Operation not permitted那是因为，mac EI Captian系统权限收紧了，不让创建了）。解决方式：找到/etc/php-fpm.conf，修改error_log = /usr/local/var/log/nginx.log，保存退出即ok了。

 

至此，php-fpm跑成功了，这时跑http://localhost/index.php 空白页。嘿嘿，我笑了，知道是跑成功了，只是fastcgi_param的问题，于是到vhost-default下，修改成：

fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;  //$document_root 就是你的虚拟主机目录

 

Mac下设置开机启动nginx和php-fpm:

第一步：

cd /Library/LaunchDaemons

sudo vim com.yourname.nginx.plist //名字随便起

粘贴：

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>KeepAlive</key>
<true/>
<key>Label</key>
<string>com.yourname.nginx</string>
<key>Program</key>
<string>/usr/local/nginx/sbin/nginx</string>    //应用地址，写自己的实际情况。
<key>RunAtLoad</key>
<true/>
<key>WorkingDirectory</key>
<string>/usr/local/var</string>
</dict>
</plist>

sudo vim com.yourname.php-fpm.plist 

粘贴：

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>KeepAlive</key>
<true/>
<key>Label</key>
<string>com.yourname.php-fpm</string>
<key>Program</key>
<string>/usr/sbin/php-fpm</string>   //应用地址，写自己的实际情况。
<key>RunAtLoad</key>
<true/>
<key>WorkingDirectory</key>
<string>/usr/local/var</string>
</dict>
</plist>

有时whereis php-fpm查到的是这个/usr/sbin/php-fpm，但不好使，我用的是/usr/local/sbin/php-fpm这个，不需要root也能运行。

 

第二步：添加到开机启动任务列表：

launchctl load -w /Library/LaunchDaemons/com.yourname.php-fpm.plist

launchctl load -w /Library/LaunchDaemons/com.yourname.nginx.plist

第三步：重启电脑，查看端口：

netstat -nat | grep LISTEN

是不是发现9000、80等端口都起来啦~

 

或者用ps命令

ps aux | grep php      //very good

 

至此，蛋疼几天的问题over~

 

 相关链接：

http://www.cnblogs.com/allen8807/archive/2010/11/10/1873843.html  //ps命令
