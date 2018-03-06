---
title: saltstack(四)——自动安装nginx-1.4.5实例  
date: 2017-05-12 15:39:22  
tags: [saltstack,运维自动化]  
categories: saltstack
---
目录结构
```
[root@linux-node1 prod]# pwd
/srv/salt/prod
[root@linux-node1 prod]# tree
.
├── nginx
│   ├── conf.sls
│   ├── files
│   │   ├── nginx
│   │   ├── nginx-1.4.5.tar.gz
│   │   ├── nginx.conf
│   │   ├── nginx_log_cut.sh
│   │   └── vhost.conf
│   ├── init.sls
│   ├── install.sls
│   └── vhost.sls
└── top.sls

2 directories, 10 files
```
<!-- more -->  
init.sls
```
[root@linux-node1 prod]# cat nginx/init.sls 
include:
  - nginx.install
  - nginx.conf
  - nginx.vhost
```
Install.sls  
&ensp;&ensp;nginx编译安装，涉及文件管理、包管理、用户管理及cmd运用，其中注意的是如果使用cmd，它每次同步客户时都会执行，为了防止这一现象，使用unless可解决。
```
[root@linux-node1 prod]# cat nginx/install.sls 
#nginx.tar.gz
nginx_source:
  file.managed:
    - name: /tmp/nginx-1.4.5.tar.gz
    - unless: test -e /tmp/nginx-1.4.5.tar.gz
    - source: salt://nginx/files/nginx-1.4.5.tar.gz

#extract

extract_nginx:
  cmd.run:
    - cwd: /tmp
    - names:
      - tar zxvf nginx-1.4.5.tar.gz
    - unless: test -d /tmp/nginx-1.4.5
    - require:
      - file: nginx_source


#user

nginx_user:
  user.present:
    - name: nginx
    - uid: 1501
    - createhome: False
    - gid_from_name: True
    - shell: /sbin/nologin

#nginx_pkgs

nginx_pkg:
  pkg.installed:
    - pkgs:
      - gcc
      - openssl-devel
      - pcre-devel
      - zlib-devel

#nginx_compile
nginx_compile:
  cmd.run:
    - cwd: /tmp/nginx-1.4.5
    - names:
      - ./configure --prefix=/usr/local/nginx  --user=nginx  --group=nginx  --with-http_ssl_module  --with-http_gzip_static_module --http-client-body-temp-path=/usr/local/nginx/client/ --http-proxy-temp-path=/usr/local/nginx/proxy/   --http-fastcgi-temp-path=/usr/local/nginx/fcgi/   --with-poll_module  --with-file-aio  --with-http_realip_module  --with-http_addition_module --with-http_random_index_module   --with-pcre   --with-http_stub_status_module
      - make
      - make install
    - require:
      - cmd: extract_nginx
      - pkg:  nginx_pkg
    - unless: test -d /usr/local/nginx

#cache_dir
cache_dir:
  cmd.run:
    - names:
      - mkdir -p /usr/local/nginx/{client,proxy,fcgi} && chown -R nginx.nginx /usr/local/nginx/
    - unless: test -d /usr/local/nginx/client/
    - require:
      - cmd: nginx_compile
```
conf.sls
```
[root@linux-node1 prod]# cat nginx/conf.sls 
include:
  - nginx.install     # 引用安装
{% set nginx_user = 'nginx' + ' ' + 'nginx' %}  # 设置用户变量
nginx_conf:
  file.managed:   # nginx主配置文件管理
    - name: /usr/local/nginx/conf/nginx.conf
    - source: salt://nginx/files/nginx.conf
    - template: jinja
    - defaults:
      nginx_user: {{ nginx_user }}      
      num_cpus: {{grains['num_cpus']}}  # 根据cpu的个数来设置nginx.conf文件
nginx_service:  # nginx服务管理
  file.managed:
    - name: /etc/init.d/nginx
    - user: root
    - mode: 755
    - source: salt://nginx/files/nginx
  cmd.run:    # 将服务由chkconfig管理
    - names:
      - /sbin/chkconfig --add nginx
      - /sbin/chkconfig  nginx on
    - unless: /sbin/chkconfig --list nginx
  service.running:     # nginx是启动状态
    - name: nginx
    - enable: True
    - reload: True
    - watch:
      - file: /usr/local/nginx/conf/*.conf
nginx_log_cut:                 # nginx日志管理
  file.managed:
    - name: /usr/local/nginx/sbin/nginx_log_cut.sh
    - source: salt://nginx/files/nginx_log_cut.sh
  cron.present:             # 将日志切割脚本加入crontab定时执行
    - name: sh /usr/local/nginx/sbin/nginx_log_cut.sh
    - user: root
    - minute: 10
    - hour: 0
    - require:
      - file: nginx_log_cut
```
Nginx启动脚本
```
[root@linux-node1 prod]# cat nginx/files/nginx
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:     /usr/local/nginx/logs/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
 
 
lockfile=/var/lock/subsys/nginx
 
make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -z "`grep $user /etc/passwd`" ]; then
       useradd -M -s /bin/nologin $user
   fi
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```
Nginx主配置文件
```
[root@linux-node1 prod]# cat nginx/files/nginx.conf 
#
user  {{ nginx_user }};     # 这里调用conf.sls中的配置
worker_processes {{grains['num_cpus']}};
error_log  logs/nginx_error.log  notice;
pid        /usr/local/nginx/sbin/nginx.pid;

worker_rlimit_nofile 65535;

events
     {
              use epoll;
              worker_connections 65535;
      }
http
     {
              include       mime.types;
              default_type  application/octet-stream;
              charset  utf-8;
              server_names_hash_bucket_size 128;
              client_header_buffer_size 32k;
              large_client_header_buffers 4 32k;
              client_max_body_size 128m;
              sendfile on;
              tcp_nopush     on;
              keepalive_timeout 60;
              tcp_nodelay on;
              server_tokens off;
              client_body_buffer_size  512k;
              gzip on;
              gzip_min_length  1k;
              gzip_buffers     4 16k;
              gzip_http_version 1.1;
              gzip_comp_level 2;
              gzip_types      text/plain application/x-javascript text/css application/xml;
              gzip_vary on;
	      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    	                        '$status $body_bytes_sent "$http_referer" '
                                '"$http_user_agent" "$http_x_forwarded_for" "$host"' ;

              include vhost*.conf;
       }
```
日志切割脚本
```
[root@linux-node1 prod]# cat nginx/files/nginx_log_cut.sh 
#!/bin/bash

logs_path=/usr/local/nginx/logs
yesterday=`date -d "yesterday" +%F`

mkdir -p $logs_path/$yesterday

cd $logs_path

for nginx_logs in `ls *log`
do
    mv $nginx_logs ${yesterday}/${yesterday}-${nginx_logs}
    kill -USR1  `cat /usr/local/nginx/sbin/nginx.pid`
done
```
Pillar目录
```
[root@linux-node1 pillar]# pwd
/srv/pillar
[root@linux-node1 pillar]# tree
.
├── nginx
│   └── init.sls
└── top.sls

1 directories, 2 files
```
Pillar配置
```
[root@linux-node1 pillar]# cat top.sls 
base:
  '*':
    - nginx
[root@linux-node1 pillar]# cat nginx/init.sls 
vhost:
  {% if 'test8' in grains['id'] %}    //如果id中有test8字符，使用vhost_www.conf配置文件，反之使用vhost_bbs.conf配置文件
  - name: www    //必须加”-“形成列表,不然在后面for引用时会报错
    target: /usr/local/nginx/conf/vhost_www.conf
  {% else %}
  - name: bbs
    target: /usr/local/nginx/conf/vhost_bbs.conf
  {% endif %}
[root@linux-node1 pillar]# salt '*' saltutil.refresh_pillar
```
vhost.sls
```
[root@linux-node1 prod]# cat nginx/vhost.sls 
include:
  - nginx.install

{% for vhostname in pillar['vhost'] %}

{{vhostname['name']}}:
  file.managed:
    - name: {{vhostname['target']}}
    - source: salt://nginx/files/vhost.conf
    - target: {{vhostname['target']}}
    - template: jinja
    - defaults:
      server_name: {{grains['fqdn_ip4'][0]}} 
      log_name: {{vhostname['name']}}
    - watch_in:
      service: nginx    #可以去掉，没有定义nginx这个service，而且前面已经watch了

{% endfor %}
```
vhost.conf
```
[root@linux-node1 prod]# cat nginx/files/vhost.conf 
server
        {
                listen       80;
                server_name {{ server_name }};  # 调用vhost.sls中配置
                index index.html index.htm ;
                root  html;
                #location ~ .*\.(php|php5)?$
                #        {
                #                try_files $uri =404;
                #                fastcgi_pass  unix:/tmp/php-cgi.sock;
                #                fastcgi_index index.php;
                #                include fcgi.conf;
                #        }
                location /status {
                       stub_status on;
                }
                location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
                        {
                                expires      30d;
                        }
                location ~ .*\.(js|css)?$
                        {
                                expires      1d;
                        }
                access_log  logs/{{ log_name }}-access.log  main;
        }
```
执行
```
[root@linux-node1 prod]# salt '*' state.highstate env=prod test=True  #先测试，没错误再执行
[root@linux-node1 prod]# salt '*' state.highstate env=prod
```