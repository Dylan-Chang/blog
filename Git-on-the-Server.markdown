原文 [Git 仓库 SSH、HTTP、Gitweb (Nginx) 乱炖 ](http://www.cnblogs.com/wangxiaoqiangs/p/6179610.html)


##简介:

自己搭建 Git 仓库，实现 SSH 协议、配合 Nginx 实现 HTTP 协议拉取、推送代码。

利用 Nginx 实现 Gitweb 在线浏览代码，使用 Gitweb-theme 更新默认 Gitweb 样式。

一、安装 Git

复制代码
shell > yum -y install git

shell > git --version  # yum 安装的 git 版本比较低，所以我选择源码编译的方式安装
git version 1.7.1

shell > yum -y remove git

shell > yum -y install perl cpio autoconf tk zlib-devel libcurl-devel openssl-devel expat-devel gettext-devel perl-ExtUtils-MakeMaker

shell > cd /usr/local/src; wget https://www.kernel.org/pub/software/scm/git/git-2.10.0.tar.gz

shell > tar zxf git-2.10.0.tar.gz && cd git-2.10.0

shell > autoconf && ./configure && make && make install

shell > git --version
git version 2.10.0

##二、Git SSH 协议

1、创建 Git 仓库

shell > mkdir -p /data/git && cd /data/git

shell > git init --bare sample.git
2、客户端 clone 仓库

shell > git config --global color.ui true
shell > git config --global user.name 'wang'
shell > git config --global user.email 'wangxiaoqiang@bftv.com'
### 初始化 git 客户端

复制代码
shell > git clone root@192.168.1.22:/data/git/sample.git  # 输入 1.22 的 root 密码  
root@192.168.1.22's password: 

shell > cd sample
shell > cp /etc/passwd .
shell > git add passwd
shell > git commit -m 'add passwd'
shell > git push -u origin master

shell > git status
 On branch master
nothing to commit (working directory clean)


 已经将新增的文件 passwd 提交到了远程 git 仓库的 master 分支

shell > rm -rf sample
shell > git clone root@192.168.1.22:/data/git/sample.git
root@192.168.1.22's password: 

shell > ls sample
passwd
 这就实现了 SSH 本地用户授权访问 git 仓库
 如果 SSH 非标准端口时，需要这样访问：git clone ssh://root@192.168.1.22:16543/data/git/sample.git
 现在你想把 git 仓库 sample.git 授权别人访问，怎么办？
 首先你得给每个人创建用户，他们才能 clone 代码，但是不能写入。
 其次你要把这些用户加入到一个组，然后把 sample.git 属组改为这个组，并且给这个组写入权限。
 或者创建一个公共用户，修改 git 仓库 sample.git 属主为这个公共用户，然后大家都使用这个用户访问代码库。

##三、Git HTTP 协议

###1、创建、配置 Git 仓库


shell > useradd -r -s /sbin/nologin www-data  # 创建运行用户

shell > mkdir -p /data/git && chown www-data.www-data /data/git && cd /data/git

shell > git init --bare sample.git && chown -R www-data.www-data sample.git

shell > cd sample.git && mv hooks/post-update.sample hooks/post-update

shell > git update-server-info
复制代码
2、安装、配置 Nginx ( 使其支持 CGI )

复制代码
shell > cd /usr/local/src
shell > git clone https://github.com/lighttpd/spawn-fcgi.git
shell > cd spawn-fcgi && ./autogen.sh && ./configure && make && make install

shell > yum -y install fcgi-devel

shell > cd /usr/local/src
shell > git clone https://github.com/gnosek/fcgiwrap.git
shell > cd fcgiwrap && autoreconf -i && ./configure && make && make install

shell > vim /etc/init.d/fcgiwrap  # 配置启动脚本

#! /bin/bash
### BEGIN INIT INFO
# Provides:          fcgiwrap
# Required-Start:    $remote_fs
# Required-Stop:     $remote_fs
# Should-Start:
# Should-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: FastCGI wrapper
# Description:       Simple server for running CGI applications over FastCGI
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
SPAWN_FCGI="/usr/local/bin/spawn-fcgi"
DAEMON="/usr/local/sbin/fcgiwrap"
NAME="fcgiwrap"

PIDFILE="/var/run/$NAME.pid"

FCGI_SOCKET="/var/run/$NAME.socket"
FCGI_USER="www-data"
FCGI_GROUP="www-data"
FORK_NUM=5
SCRIPTNAME=/etc/init.d/$NAME

case "$1" in
    start)
        echo -n "Starting $NAME... "

        PID=`pidof $NAME`
        if [ ! -z "$PID" ]; then
            echo " $NAME already running"
            exit 1
        fi

        $SPAWN_FCGI -u $FCGI_USER -g $FCGI_GROUP -s $FCGI_SOCKET -P $PIDFILE -F $FORK_NUM -f $DAEMON

        if [ "$?" != 0 ]; then
            echo " failed"
            exit 1
        else
            echo " done"
        fi
    ;;

    stop)
        echo -n "Stoping $NAME... "

        PID=`pidof $NAME`
        if [ ! -z "$PID" ]; then
            kill `pidof $NAME`
            if [ "$?" != 0 ]; then
                echo " failed. re-quit"
                exit 1
            else
                rm -f $pid
                echo " done"
            fi
        else
            echo "$NAME is not running."
            exit 1
        fi
    ;;

    status)
        PID=`pidof $NAME`
        if [ ! -z "$PID" ]; then
            echo "$NAME (pid $PID) is running..."
        else
            echo "$NAME is stopped"
            exit 0
        fi
    ;;

    restart)
        $SCRIPTNAME stop
        sleep 1
        $SCRIPTNAME start
    ;;

    *)
        echo "Usage: $SCRIPTNAME {start|stop|restart|status}"
        exit 1
    ;;
esac
复制代码
 注意 spawn-fcgi 跟 fcgiwrap 脚本路径及 FCGI_GROUP 跟 FCGI_GROUP
 脚本启动了 5 个 cgi 进程，按需调整

复制代码
shell > chmod a+x /etc/init.d/fcgiwrap

shell > chkconfig --level 35 fcgiwrap on

shell > /etc/init.d/fcgiwrap start

shell > sh auto.sh install nginx  # 安装 nginx

 #!/bin/bash

install_nginx(){
  yum -y install gcc gcc-c++ wget make pcre-devel zlib-devel openssl-devel

  id www-data > /dev/null 2>&1 || useradd -r -s /sbin/nologin www-data

  cd /usr/local/src; wget -qc http://nginx.org/download/nginx-1.10.2.tar.gz || exit 9

  tar zxf nginx-1.10.2.tar.gz; cd nginx-1.10.2
  ./configure --prefix=/usr/local/nginx-1.10.2 \
              --with-http_dav_module \
              --with-http_ssl_module \
              --with-http_realip_module \
              --with-http_gzip_static_module \
              --with-http_stub_status_module \
              --with-http_degradation_module && make && make install
  mkdir /usr/local/nginx-1.10.2/conf/vhost; mkdir -p /data/logs/nginx
  echo "/usr/local/nginx-1.10.2/sbin/nginx" >> /etc/rc.local
}

[ $# -lt 2 ] && exit 9

if [ $1 == 'install' ];then
  case $2 in
    nginx)
      install_nginx ;;
    *)
      echo 'NULL' ;;
  esac
fi

 # --with-http_dav_module  不添加该模块无法 git push，请查找 Nginx WebDAV 模块


shell > vim /usr/local/nginx-1.10.2/conf/vhost/git.server.conf

server {
    listen      80;
    server_name git.server.com;

    client_max_body_size 100m;

    auth_basic "Git User Authentication";
    auth_basic_user_file /usr/local/nginx-1.10.2/conf/pass.db;

    location ~ ^.*\.git/objects/([0-9a-f]+/[0-9a-f]+|pack/pack-[0-9a-f]+.(pack|idx))$ {
        root /data/git;
    }    
    
    location ~ /.*\.git/(HEAD|info/refs|objects/info/.*|git-(upload|receive)-pack)$ {
        root          /data/git;
        fastcgi_pass  unix:/var/run/fcgiwrap.socket;
        fastcgi_connect_timeout 24h;
        fastcgi_read_timeout 24h;
        fastcgi_send_timeout 24h;
        fastcgi_param SCRIPT_FILENAME   /usr/local/libexec/git-core/git-http-backend;
        fastcgi_param PATH_INFO         $uri;
        fastcgi_param GIT_HTTP_EXPORT_ALL "";
        fastcgi_param GIT_PROJECT_ROOT  /data/git;
        fastcgi_param REMOTE_USER $remote_user;
        include fastcgi_params;
    }
}

 # 自己按需修改 nginx.conf，user www-data www-data; 不要忘记加入 include vhost/*.conf;
 # 注意 认证文件 pass.db 路径
 # 注意 git-http-backend 路径 
 # 第一个 location 用于静态文件直接读取
 # 第二个 location 用于将指定动作转给 cgi 执行
 # 根目录指向 git 仓库目录

复制代码
shell > /usr/local/nginx-1.10.2/sbin/nginx

shell > yum -y install httpd-tools  # 安装 htpasswd 命令

shell > cd /usr/local/nginx-1.10.2/conf

shell > htpasswd -c pass.db wang  # 添加用户时执行 htpasswd pass.db username
复制代码
3、客户端 clone

shell > git config --global color.ui true
shell > git config --global user.name 'wnag'
shell > git config --global user.email 'wangxiaoqiang@bftv.com'
 # 初始化 git 客户端

复制代码
shell > git clone http://git.server.com/sample.git  # 输入用户名、密码即可
正克隆到 'sample'...
Username for 'http://git.server.com': wang
Password for 'http://wang@git.server.com': 
warning: 您似乎克隆了一个空仓库。

shell > cd sample.git
shell > cp /etc/passwd .
shell > git add passwd
shell > git commit -m 'add passwd'
shell > git push
Username for 'http://git.server.com': wang
Password for 'http://wang@git.server.com': 
对象计数中: 3, 完成.
Delta compression using up to 24 threads.
压缩对象中: 100% (2/2), 完成.
写入对象中: 100% (3/3), 826 bytes | 0 bytes/s, 完成.
Total 3 (delta 0), reused 0 (delta 0)
To http://git.server.com/sample.git
 * [new branch]      master -> master
复制代码
 # htpasswd 创建新用户，就可以协作了。

四、Gitweb 在线代码预览

# 按照我的方法安装 Git ，Gitweb 已经集成了，我们要做的就是找到并配置 Gitweb 。

复制代码
shell > /usr/local/share/gitweb  # Gitweb 路径

shell > ls /usr/local/share/gitweb  # gitweb.cgi 脚本、static 这是个目录，里面是静态文件 css 、js 、logo 等
gitweb.cgi  static

shell > vim /etc/gitweb.conf  # 生成配置文件

# path to git projects (<project>.git)
$projectroot = "/data/git";

# directory to use for temp files
$git_temp = "/tmp";

# target of the home link on top of all pages
$home_link = $my_uri || "/";

# html text to include at home page

$home_text = "indextext.html";

# file with project list; by default, simply scan the projectroot dir.
$projects_list = $projectroot;

# javascript code for gitweb
$javascript = "static/gitweb.js";

# stylesheet to use
$stylesheet = "static/gitweb.css";

# logo to use
$logo = "static/git-logo.png";

# the 'favicon'
$favicon = "static/git-favicon.png";

# 注意 Git 仓库路径、静态文件路径

shell > /usr/local/share/gitweb/gitweb.cgi  # 手动执行开是否报错，Status: 200 OK 正常，以下为报错及解决方法
Status: 200 OK
Content-Type: text/html; charset=utf-8
.....
复制代码
报错 1：

Can't locate CPAN.pm in @INC (@INC contains: /usr/local/lib/perl5 /usr/local/share/perl5 /usr/lib/perl5/vendorperl /usr/share/perl5/vendorperl /usr/lib/perl5 /usr/share/perl5 .) BEGIN failed--compilation aborted.
解决方法:

shell > yum -y install perl-CPAN
报错 2:

Can't locate CGI.pm in @INC (@INC contains: /usr/local/lib/perl5 /usr/local/share/perl5 /usr/lib/perl5/vendorperl /usr/share/perl5/vendorperl /usr/lib/perl5 /usr/share/perl5 .) BEGIN failed--compilation aborted.
解决方法:

shell > yum -y install perl-CGI
报错 3:

Can't locate Time/HiRes.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at /usr/local/share/gitweb/gitweb.cgi line 20.
解决方法:

shell > yum -y install perl-Time-HiRes
复制代码
shell > vim /usr/local/nginx-1.10.2/conf/vhost/git.server.conf  # 添加 Gitweb 配置

server {
    listen      80;
    server_name git.server.com;
    root        /usr/local/share/gitweb;

    client_max_body_size 100m;

    auth_basic "Git User Authentication";
    auth_basic_user_file /usr/local/nginx-1.10.2/conf/pass.db;

    location ~ ^.*\.git/objects/([0-9a-f]+/[0-9a-f]+|pack/pack-[0-9a-f]+.(pack|idx))$ {
        root /data/git;
    }

    location ~ /.*\.git/(HEAD|info/refs|objects/info/.*|git-(upload|receive)-pack)$ {
        root          /data/git;
        fastcgi_pass  unix:/var/run/fcgiwrap.socket;
        fastcgi_connect_timeout 24h;
        fastcgi_read_timeout 24h;
        fastcgi_send_timeout 24h;
        fastcgi_param SCRIPT_FILENAME     /usr/local/libexec/git-core/git-http-backend;
        fastcgi_param PATH_INFO           $uri;
        fastcgi_param GIT_HTTP_EXPORT_ALL "";
        fastcgi_param GIT_PROJECT_ROOT    /data/git;
        fastcgi_param REMOTE_USER $remote_user;
        include fastcgi_params;
    }

    try_files $uri @gitweb;

    location @gitweb {
        fastcgi_pass  unix:/var/run/fcgiwrap.socket;
        fastcgi_param GITWEB_CONFIG    /etc/gitweb.conf;
        fastcgi_param SCRIPT_FILENAME  /usr/local/share/gitweb/gitweb.cgi;
        fastcgi_param PATH_INFO        $uri;
        include fastcgi_params;
    }
}
复制代码
# 全局中指定了 gitweb 根目录，最后配置区域加入了 gitweb 的相关配置。
# 现在浏览器访问 git.server.com 就可以预览 Git 代码库及其中的代码了。

五、Gitweb-theme 样式

# 如果觉得 gitweb 默认样式不好看，可以拿该样式替换

shell > cd /usr/local/src
shell > git clone https://github.com/kogakure/gitweb-theme.git
shell > cd gitweb-theme
shell > ./setup -vi -t /usr/local/share/gitweb --install  # -t 指定 gitweb 根目录，一路 y 即可
# 这时刷新浏览器，就会发现界面的变化


2016-12-14 git push 报错：

复制代码
error: RPC failed; HTTP 413 curl 22 The requested URL returned error: 413 Request Entity Too Large
fatal: The remote end hung up unexpectedly
fatal: The remote end hung up unexpectedly

解决方法：

shell > vim /usr/local/nginx-1.10.2/conf/vhost/git.server.conf  # 或者 nginx.conf 也可以

client_max_body_size 100m;

####补充
2017/05/18 11:59:12 [crit] 39207#0: *69 connect() to unix:/var/run/fcgiwrap.socket failed (13: Permission denied) while connecting to upstream, client: 1.180.215.84, server: git.server.com, request: "GET /favicon.ico HTTP/1.1", upstream: "fastcgi://unix:/var/run/fcgiwrap.socket:", host: "git.server.com", referrer: "http://git.server.com/“

解决

chmod 0666 /var/run/fcgiwrap.socket
