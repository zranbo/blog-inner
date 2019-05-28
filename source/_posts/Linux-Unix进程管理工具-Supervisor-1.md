---
title: Linux/Unix进程管理工具 - Supervisor
author: zranbo
abbrlink: 1f6c73c0
tags:
  - Linux
categories: []
date: 2019-05-28 16:40:00
---
<b>Supervisor</b>

[`Supervisor`](http://supervisord.org/) 是一个用 `Python2` 实现的 Unix-like 系统进程管理工具。

<b>安装</b>

 1. Ubuntu : `apt install supervisor`
 2. 其他 Linux 发行版: `pip install supervisor` (只支持 `Python2` 因此请确定是`Python2`的`pip`)

安装完成后会生成三个执行程序：`supervisortd` `supervisorctl` `echo_supervisord_conf` 分别是：守护进程服务、客户端、初始配置文件程序。

<b>配置</b>

运行 `supervisord` 要指定 `supervisor` 配置文件,默认在以下目录查找：
```bash
/etc/supervisor/supervisord.conf
```
并且可以通过 `echo_supervisord_conf` 生成一份 `supervisor` 配置文件
```bash
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```
<b>主程序配置文件说明</b>

注释：以分号 `;` 开头的表示注释行
```ini
[unix_http_server]
file=/tmp/supervisor.sock		; UNIX socket 文件，supervisorctl要用
;chmod=0700				; socket文件的权限，默认是0700
;chown=nobody:nogroup			; socket文件的 owner，格式：uid:gid
;username=user				; 登录管理后台的用户名
;password=123				; 登录管理后台的密码

[inet_http_server]			; HTTP服务，提供Web端管理界面
port=127.0.0.1:9001			; Web管理后台运行的IP和端口
username=user				; 登录管理后台的用户名
password=123				; 登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log 		; 主程序的日志文件
logfile_maxbytes=50MB        		; 主程序日志大小，默认 50MB，0，表示不限制
logfile_backups=10           		; 主程序日志备份数量，默认10，0，表示不备份
loglevel=info                		; 主程序日志级别，默认 info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid 		; supervisord pid 文件; 默认 supervisord.pid
nodaemon=false               		; 是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024                  		; 可以打开的文件描述符的最小值，默认 1024
minprocs=200                 		; 可以打开的进程数的最小值，默认 200
;umask=022                   		; umask 022 创建新文件权限为755
;user=chrism                 		; 启动用户，默认为当前用户
;identifier=supervisor       		; supervisord 标识, 默认 `supervisor`
;directory=/tmp              		; 当 supervisord 守护进程时，切换到此目录
;nocleanup=true              		; 启动时清除子日志文件，默认 false
;childlogdir=/tmp            		; 子日志文件路径 默认为 $TEMP
;environment=KEY="value"     		; 添加环境变量到 supervisord 进程的环境中
;strip_ansi=false            		; 删除日志文件中所有ANSI转义序列


[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock	; 通过UNIX socket 连接supervisord，与[unix_http_server]部分file一致
;serverurl=http://127.0.0.1:9001 	; 通过 HTTP 方式连接 supervisord
;username=chris              		; 用于身份验证的用户名。与 supervisord 服务器配置的用户名相同
;password=123                		; 用于身份验证的密码。与 supervisord 服务器配置的密码的明文相同。
;prompt=mysupervisor         		; 用作 supervisorctl 提示符的字符串
;history_file=~/.sc_history  		; 用作 readline 持久性历史记录文件的路径

[include]
files = /etc/supervisor/conf.d/*.ini 	; 可以指定一个或多个以.ini结束的配置文件
```
<b>配置管理进程</b>

进程管理配置参数，不建议全都写在 `supervisord.conf` 文件中，建议每个进程采用一个配置文件放到 `supervisord.conf` 配置中 `[include]`字段 `files` 指定的路径下

```ini
;[program:programname]			; program:xxx (xxx 为自定义的管理的进程的名称名称，不可重复)
;command=/bin/cat			; 程序启动命令 (该命令只能是前台任务命令)
;process_name=%(program_name)s		; 进程名称
;numprocs=1				; 进程数量 (默认为 1)
;directory=/tmp               		; 在执行 command 之前，supervisord 切换到的目录。
;autostart=true                		; supervisord 启动的时候也自动启动
;startsecs=1                   		; 启动1秒后没有异常退出，就表示进程正常启动了，默认为1秒
;startretries=3                		; 启动失败自动重试次数，默认是3次
;autorestart=unexpected        		; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
;stopasgroup=false             		; 进程被杀死时，是否向这个进程组发送stop信号，包括子进程
;killasgroup=false             		; 进程组发送kill信号，包括子进程
;user=chrism                   		; 启动进程的用户
;redirect_stderr=false          	; 把stderr重定向到stdout，默认false
;stdout_logfile=/a/path        		; stdout 日志文件路径
;stdout_logfile_maxbytes=1MB   		; stdout 日志文件大小 0 表示无限制
;stdout_logfile_backups=10     		; stdout 日志文件备份数量 0 表示不备份
;stdout_capture_maxbytes=1MB   		; stdout 捕获模式 日志文件大小
;stderr_logfile=/a/path        		; stderr 日志文件路径
;stderr_logfile_maxbytes=1MB   		; stderr 日志文件大小 0 表示无限制
;stderr_logfile_backups=10     		; stderr 日志文件备份数量 0 表示不备份
;stderr_capture_maxbytes=1MB   		; stderr 捕获模式 日志文件大小
;environment=A="1",B="2"       		; 添加环境变量到子进程的环境中
```

<b>启动Supervisor服务</b>

```bash
supervisord -c /etc/supervisor/supervisord.conf
```

<b>supervisorctl 管理进程</b>

启动成功后，就可以通过 supervisorctl 管理进程。在 shell 中直接输入 `supervisorctl` 就可以直接进入交互式的环境中
```shell
supervisor> status		# 查看所有进程的状态
supervisor> update		# 重启配置文件修改过的程序
supervisor> reread		# 读取有更新（增加）的配置文件，不会启动新添加的程序
supervisor> start program	# 启动 program 程序
supervisor> stop program	# 停止 program 程序
supervisor> restart program	# 重启 program 程序
```