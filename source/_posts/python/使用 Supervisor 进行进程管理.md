---
title: 使用 Supervisor 进行进程管理  
date: 2018-03-02 17:13:53  
tags: [python]  
categories: python
---
##### 简介
&ensp;&ensp;Supervisor是可以在类UNIX系统中进行管理和监控各种进程的小型系统。它自带了客户端和服务端工具。它可以通过用户所定义的配置文件来管理和监控单个或多个进程，并且它可以根据配置来对异常崩溃的进程进行重启操作。supervisor只支持python2，python3需要配合virtualenv使用或用circus代替。  
<!-- more -->
##### 安装配置
###### 安装
```
# 需要在python2环境下运行(也可以通过配置epel源使用yum进行安装)
pip install supervisor
```
###### 配置
###### 主配置文件
&ensp;&ensp;Superviosr通过配置文件来设置被监管的程序。一般配置文件都放置在/etc/supervisor/conf.d路径下面。将每一个应用创建一个conf文件，放在/etc/supervisor/conf.d目录下，再通过主配置文件supervisord.conf包含了conf.d目录下的配置文件。  
```
# 创建supervisor配置文件目录
mkdir /etc/supervisor/conf.d/ -p
# 生成默认配置文件
echo_supervisord_conf > /etc/supervisor/supervisord.conf
# 修改默认配置文件包含conf.d目录下文件
vim /etc/supervisor/supervisord.conf
[include]
files = /etc/supervisor/conf.d/*.conf
```
###### 程序配置文件(以uwsgi为例)
```
vim /etc/supervisor/conf.d/uwsgi.conf
[program:uwsgi]
command=/root/.pyenv/versions/kvmmgr/bin/uwsgi --ini /root/kvmmgr/hxoms/config/uwsgi.ini
user=root
numprocs=1
directory=/root/kvmmgr/hxoms/
stdout_logfile=/var/log/uwsgi_access.log
stderr_logfile=/var/log/uwsgi_error.log
autostart=true
autorestart=true
startsecs=10
stopwaitsecs = 120
priority=998
```
**注：**这里需要注意uwsgi配置为前台运行，因为supervisor只能监控前台程序，以daemon方式运行的程序不能用它监控，否则status会提示BACKOFF  Exited too quickly (process log may have details)。uwsgi.ini文件如下：  
```
[uwsgi]
socket = 0.0.0.0:9001
;socket = /tmp/uwsgi.sock
master = true
pidfile = /var/run/hxoms.pid
processes = 8
chdir = /root/kvmmgr/hxoms
home = /root/.pyenv/versions/kvmmgr
module = hxoms.wsgi
profiler = true
memory-report=true
enable-threads=true
logdate=true
vacuum = true
limit-as=6048
chmod-socket = 664
#daemonize=/var/log/hxoms.log # 此项不能开启
```
###### supervisor常用配置项
```
; * 为必选项
;[program:theprogramname]
;command=/bin/cat              ; * 要执行的命令路径（可以是相对与 PATH 的路径）也可带参数
;process_name=%(program_name)s ; 当进程数为 1 时 为 %(program_name)s 当进程数 >1 时 应配置为 %(program_name)s_%(process_num)02d
;numprocs=1                    ; 进程数量 (默认 1)
;directory=/tmp                ; 在执行运行前切换到目录 (def no cwd)
;umask=022                     ; 掩码 (default None)
;priority=999                  ; 优先级，数值越低越先启动而越后关闭 (default 999)
;autostart=true                ; 在 supervisord 启动时即启动 (default: true)
;startsecs=1                   ; 需要考虑进程启动成功的时间, 当 running 状态超过该值时，表明启动成功 (def. 1)
;startretries=3                ; 在启动时状态为失败时的最大尝试重启次数 (default 3)
;autorestart=unexpected        ; 当进程退出时是否应该重启，可选值为 false true unexpected ，为 false 时表示不重启，为 true 表示重启，为 unexpected 时，如果退出状态码不是 exitcodes 中之一时进行重启 (def: unexpected)
;exitcodes=0,2                 ; 用来重启的状态码 (default 0,2)
;stopsignal=QUIT               ; 进程停止信号，当用设定的信号去干掉进程，退出码会被认为是 expected (default TERM)
;stopwaitsecs=10               ; 这个是当我们向子进程发送 stopsignal 信号后，到系统返回信息给 supervisord，所等待的最大时间。超过这个时间，supervisord会向该子进程发送一个强制 kill 的信号。 (default 10)
;stopasgroup=false             ; 如果设置为 true 那么将会终止该进程下的所有子进程 (default false)
;killasgroup=false             ; 当进程关闭时向该进程的子进程发送的是 kill 信号 (def false)
;user=chrism                   ; 可以管理该 program 的用户
;redirect_stderr=true          ; 如果为 true，那么 stderr 将会被写入 stdout 日志文件中 (default false)
;stdout_logfile=/a/path        ; 日志文件路径， NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; 日志文件的最大大小 (default 50MB)
;stdout_logfile_backups=10     ; 日志文件的最大数量 (default 10)
;stdout_capture_maxbytes=1MB   ; capture 管道的大小，当值不为 0 时，子进程可以从 stdout 发送信息，而 supervisor 可以根据信息，发送相应的 event (default 0)
;stdout_events_enabled=false   ; 当设置为 ture 的时候，当子进程由 stdout 向文件描述符中写日志的时候，将触发 supervisord 发送 PROCESS_LOG_STDOUT 类型的 event (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; 子进程共享的环境变量 (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)
```
##### 启动
配置完成之后使用supervisord命令来启动服务端监控：  
```
supervisord -c /etc/supervisor/supervisord.conf
```
##### 使用supervisorctl管理
启动 supervisorctl 时需要指定配置文件路径，否则会报错：http://localhost:9001 refused connection.  
```
supervisorctl -c /etc/supervisord.conf
```
supervisorctl常用命令：  
```
> status       # 查询进程状态
> stop node    # 关闭 [program:node] 的进程
> start node   # 启动 [program:node] 的进程
> restart node # 重启 [program:node] 的进程
> stop all     # 关闭所有进程
> start all    # 启动所有进程
> reread       # 重新读取配置文件,读取有更新（增加）的配置文件，不会启动新添加的程序
> reload       # 重新载入配置文件
> update       # 重启配置文件修改过的程序
```
##### 启用web管理界面
```
vim /etc/supervisor/supervisord.conf
[inet_http_server]         ; inet (TCP) server disabled by default
port=0.0.0.0:9002        ; (ip_address:port specifier, *:port for all iface)
username=user              ; (default is no username (open server))
password=123               ; (default is no password (open server))
```
reload后访问http://ip:9002可以看到web管理界面。  
##### 参考资料
[使用 Supervisor 进行进程管理](https://www.jianshu.com/p/2a48b2c987e0)  
[supervisor+gunicorn部署python web项目
](http://www.cnblogs.com/weidiao/p/6505346.html)  
[supervisor初体验](https://www.jianshu.com/p/9abffc905645)  
