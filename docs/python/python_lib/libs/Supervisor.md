---
title: Supervisor -> 进程管理工具
tags:
  - Python
---


## 概述：

> 官方文档：[http://supervisord.org/](http://supervisord.org/)

supervisor是用python开发的一款用于linux操作系统的进程管理工具，supervisor可以高效地实现对进程状态的监控及统一管理

- supervisor分为supervisord（服务端）和supervisorctl（客户端）两部分
- supervisord作为服务端负责所有的注册进程的启动及监控，出现崩溃的服务时会自动重启
- supervisorctl是客户端，可以实现对子进程的管理，如启动、停止、重启等

注意：

- supervisord 要求管理的程序是非 daemon 程序， 因此用supervisord 来管理进程，进程需要以非daemon的方式启动。
   - 例如：管理nginx，必须在 nginx 的配置文件里添加一行设置 daemon off 让 nginx 以非 daemon 方式启动。
- 每次修改配置文件后需进入 supervisorctl，执行 reload， 改动部分才能生效。

## 使用：
### 安装
```bash
pip install supervisor

# 生成配置文件到指定路径（此处路径指定为了/etc/supervisord.conf）
echo_supervisord_conf > /etc/supervisord.conf
```

### 常用命令：

- supervisord: 服务端命令：
```bash
# 启动服务
supervisord -c /etc/supervisord.conf

# 查看是否在运行
ps aux | grep supervisord
```

- supervisorctl: 客户端命令：
```bash
supervisorctl status                         # 查看进程状态
supervisorctl start program_name             # 启动program_name进程
supervisorctl stop program_name              # 终止program_name进程
supervisorctl restart program_name           # 重启program_name进程
supervisorctl reread                         # 更新配置，根据最新的配置启动所有程序
supervisorctl update                         # 更新配置，重启配置有变化的进程
```

- supervisorctl: shell命令：
```bash
supervisorctl                # 进入shell命令行
> status                     # 查看进程状态
> start program_name         # 启动program_name进程
> stop program_name          # 终止program_name进程
> restart program_name       # 重启program_name进程
> reread                     # 更新配置，根据最新的配置启动所有程序
> update                     # 更新配置，重启配置有变化的进程
> start fastapi:*            # 启动 fastapi组 程序
> stop fastapi:*             # 停止 fastapi组 程序
```

### 快速使用：

- step1：编辑supervisor配置文件
   - 在配置文件的末尾加上[include]内容
```bash
...
[include]
files=/etc/supervisor.d/*.conf  #若你本地无/etc/supervisor.d目录，请自建
...
```

- step2：新建/etc/supervisor.d/.demo.conf 配置文件
```bash
[program:自定义的服务名称]
command=python3 /xxxx/main.py  #服务的启动命令，在这里用的python环境为supervisor安装的环境，若想指定其他python环境则可指定python包的位置如command=xxxx/xxxx/bin/python3 /xxxx/main.py便可
process_name=%(program_name)s
stdout_logfile=/home/task.log  #输出日志文件路径，可根据需要自定义
stderr_logfile=/home/task.log  #报错日志文件路径，可根据需要自定义
```

- step3:启动
```bash
supervisord -c /etc/supervisor.conf
```
## 配置文件解读：
### 常用配置解读

- 打开配置文件： /etc/supervisord.conf
```bash
[unix_http_server]
file=/tmp/supervisor.sock   ;UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ;socket文件的mode，默认是0700
;chown=nobody:nogroup       ;socket文件的owner，格式：uid:gid

[inet_http_server]         ;HTTP服务器，提供web管理界面
port=0.0.0.0:9001          ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
username=user              ;登录管理后台的用户名
password=123               ;登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10           ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info                ;日志级别，默认info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ;pid 文件
nodaemon=false               ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024                  ;可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ;可以打开的进程数的最小值，默认 200

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ;通过UNIX socket连接supervisord，路径与unix_http_server部分的file一致
;serverurl=http://127.0.0.1:9001 ; 通过HTTP的方式连接supervisord


# 以上东西都不需要改，就加下面这行
[include]
files=/etc/supervisor.d/*.conf  #若你本地无/etc/supervisor.d目录，请自建
```
### 详细配置解读：
```bash
[unix_http_server]            
 
file=/tmp/supervisor.sock   ; socket文件的路径，supervisorctl用XML_RPC和supervisord通信就是通过它进行
 
                              的。如果不设置的话，supervisorctl也就不能用了  
 
                              不设置的话，默认为none。 非必须设置        
 
;chmod=0700                 ; 这个简单，就是修改上面的那个socket文件的权限为0700
 
                              不设置的话，默认为0700。 非必须设置
 
;chown=nobody:nogroup       ; 这个一样，修改上面的那个socket文件的属组为user.group
 
                              不设置的话，默认为启动supervisord进程的用户及属组。非必须设置
 
;username=user              ; 使用supervisorctl连接的时候，认证的用户
 
                               不设置的话，默认为不需要用户。 非必须设置
 
;password=123               ; 和上面的用户名对应的密码，可以直接使用明码，也可以使用SHA加密
 
                              如：{SHA}82ab876d1387bfafe46cc1c8a2ef074eae50cb1d
 
                              默认不设置。。。非必须设置
 
;[inet_http_server]         ; 侦听在TCP上的socket，Web Server和远程的supervisorctl都要用到他
 
                              不设置的话，默认为不开启。非必须设置
 
;port=127.0.0.1:9001        ; 这个是侦听的IP和端口，侦听所有IP用 :9001或*:9001。
 
                              这个必须设置，只要上面的[inet_http_server]开启了，就必须设置它
 
;username=user              ; 这个和上面的uinx_http_server一个样。非必须设置
 
;password=123               ; 这个也一个样。非必须设置
 
[supervisord]                ;这个主要是定义supervisord这个服务端进程的一些参数的
 
                              这个必须设置，不设置，supervisor就不用干活了
 
logfile=/tmp/supervisord.log ; 这个是supervisord这个主进程的日志路径，注意和子进程的日志不搭嘎。
 
                               默认路径$CWD/supervisord.log，$CWD是当前目录。。非必须设置
 
logfile_maxbytes=50MB        ; 这个是上面那个日志文件的最大的大小，当超过50M的时候，会生成一个新的日 
 
                               志文件。当设置为0时，表示不限制文件大小
 
                               默认值是50M，非必须设置。              
 
logfile_backups=10           ; 日志文件保持的数量，上面的日志文件大于50M时，就会生成一个新文件。文件
 
                               数量大于10时，最初的老文件被新文件覆盖，文件数量将保持为10
 
                               当设置为0时，表示不限制文件的数量。
 
                               默认情况下为10。。。非必须设置
 
loglevel=info                ; 日志级别，有critical, error, warn, info, debug, trace, or blather等
 
                               默认为info。。。非必须设置项
 
pidfile=/tmp/supervisord.pid ; supervisord的pid文件路径。
 
                               默认为$CWD/supervisord.pid。。。非必须设置
 
nodaemon=false               ; 如果是true，supervisord进程将在前台运行
 
                               默认为false，也就是后台以守护进程运行。。。非必须设置
 
minfds=1024                  ; 这个是最少系统空闲的文件描述符，低于这个值supervisor将不会启动。
 
                               系统的文件描述符在这里设置cat /proc/sys/fs/file-max
 
                               默认情况下为1024。。。非必须设置
 
minprocs=200                 ; 最小可用的进程描述符，低于这个值supervisor也将不会正常启动。
 
                              ulimit  -u这个命令，可以查看linux下面用户的最大进程数
 
                              默认为200。。。非必须设置
 
;umask=022                   ; 进程创建文件的掩码
 
                               默认为022。。非必须设置项
 
;user=chrism                 ; 这个参数可以设置一个非root用户，当我们以root用户启动supervisord之后。
 
                               我这里面设置的这个用户，也可以对supervisord进行管理
 
                               默认情况是不设置。。。非必须设置项
 
;identifier=supervisor       ; 这个参数是supervisord的标识符，主要是给XML_RPC用的。当你有多个
 
                               supervisor的时候，而且想调用XML_RPC统一管理，就需要为每个
 
                               supervisor设置不同的标识符了
 
                               默认是supervisord。。。非必需设置
 
;directory=/tmp              ; 这个参数是当supervisord作为守护进程运行的时候，设置这个参数的话，启动
 
                               supervisord进程之前，会先切换到这个目录
 
                               默认不设置。。。非必须设置
 
;nocleanup=true              ; 这个参数当为false的时候，会在supervisord进程启动的时候，把以前子进程
 
                               产生的日志文件(路径为AUTO的情况下)清除掉。有时候咱们想要看历史日志，当 
 
                               然不想日志被清除了。所以可以设置为true
 
                               默认是false，有调试需求的同学可以设置为true。。。非必须设置
 
;childlogdir=/tmp            ; 当子进程日志路径为AUTO的时候，子进程日志文件的存放路径。
 
                               默认路径是这个东西，执行下面的这个命令看看就OK了，处理的东西就默认路径
 
                               python -c "import tempfile;print tempfile.gettempdir()"
 
                               非必须设置
 
;environment=KEY="value"     ; 这个是用来设置环境变量的，supervisord在linux中启动默认继承了linux的
 
                               环境变量，在这里可以设置supervisord进程特有的其他环境变量。
 
                               supervisord启动子进程时，子进程会拷贝父进程的内存空间内容。 所以设置的
 
                               这些环境变量也会被子进程继承。
 
                               小例子：environment=name="haha",age="hehe"
 
                               默认为不设置。。。非必须设置
 
;strip_ansi=false            ; 这个选项如果设置为true，会清除子进程日志中的所有ANSI 序列。什么是ANSI
 
                               序列呢？就是我们的\n,\t这些东西。
 
                               默认为false。。。非必须设置
 
; the below section must remain in the config file for RPC
 
; (supervisorctl/web interface) to work, additional interfaces may be
 
; added by defining them in separate rpcinterface: sections
 
[rpcinterface:supervisor]    ;这个选项是给XML_RPC用的，当然你如果想使用supervisord或者web server 这 
 
                              个选项必须要开启的
 
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface 
 
[supervisorctl]              ;这个主要是针对supervisorctl的一些配置  
 
serverurl=unix:///tmp/supervisor.sock ; 这个是supervisorctl本地连接supervisord的时候，本地UNIX socket
 
                                        路径，注意这个是和前面的[unix_http_server]对应的
 
                                        默认值就是unix:///tmp/supervisor.sock。。非必须设置
 
;serverurl=http://127.0.0.1:9001 ; 这个是supervisorctl远程连接supervisord的时候，用到的TCP socket路径
 
                                   注意这个和前面的[inet_http_server]对应
 
                                   默认就是http://127.0.0.1:9001。。。非必须项
 
 
 
;username=chris              ; 用户名
 
                               默认空。。非必须设置
 
;password=123                ; 密码
 
                              默认空。。非必须设置
 
;prompt=mysupervisor         ; 输入用户名密码时候的提示符
 
                               默认supervisor。。非必须设置
 
;history_file=~/.sc_history  ; 这个参数和shell中的history类似，我们可以用上下键来查找前面执行过的命令
 
                               默认是no file的。。所以我们想要有这种功能，必须指定一个文件。。。非
 
                               必须设置
 
; The below sample program section shows all possible program subsection values,
 
; create one or more 'real' program: sections to be able to control them under
 
; supervisor.
 
;[program:theprogramname]      ;这个就是咱们要管理的子进程了，":"后面的是名字，最好别乱写和实际进程
 
                                有点关联最好。这样的program我们可以设置一个或多个，一个program就是
 
                                要被管理的一个进程
 
;command=/bin/cat              ; 这个就是我们的要启动进程的命令路径了，可以带参数
 
                                例子：/home/test.py -a 'hehe'
 
                                有一点需要注意的是，我们的command只能是那种在终端运行的进程，不能是
 
                                守护进程。这个想想也知道了，比如说command=service httpd start。
 
                                httpd这个进程被linux的service管理了，我们的supervisor再去启动这个命令
 
                                这已经不是严格意义的子进程了。
 
                                这个是个必须设置的项
 
;process_name=%(program_name)s ; 这个是进程名，如果我们下面的numprocs参数为1的话，就不用管这个参数
 
                                 了，它默认值%(program_name)s也就是上面的那个program冒号后面的名字，
 
                                 但是如果numprocs为多个的话，那就不能这么干了。想想也知道，不可能每个
 
                                 进程都用同一个进程名吧。
 
 
 
;numprocs=1                    ; 启动进程的数目。当不为1时，就是进程池的概念，注意process_name的设置
 
                                 默认为1    。。非必须设置
 
;directory=/tmp                ; 进程运行前，会前切换到这个目录
 
                                 默认不设置。。。非必须设置
 
;umask=022                     ; 进程掩码，默认none，非必须
 
;priority=999                  ; 子进程启动关闭优先级，优先级低的，最先启动，关闭的时候最后关闭
 
                                 默认值为999 。。非必须设置
 
;autostart=true                ; 如果是true的话，子进程将在supervisord启动后被自动启动
 
                                 默认就是true   。。非必须设置
 
;autorestart=unexpected        ; 这个是设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected
 
                                 和true。如果为false的时候，无论什么情况下，都不会被重新启动，
 
                                 如果为unexpected，只有当进程的退出码不在下面的exitcodes里面定义的退 
 
                                 出码的时候，才会被自动重启。当为true的时候，只要子进程挂掉，将会被无
 
                                 条件的重启
 
;startsecs=1                   ; 这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启
 
                                 动成功了
 
                                 默认值为1 。。非必须设置
 
;startretries=3                ; 当进程启动失败后，最大尝试启动的次数。。当超过3次后，supervisor将把
 
                                 此进程的状态置为FAIL
 
                                 默认值为3 。。非必须设置
 
;exitcodes=0,2                 ; 注意和上面的的autorestart=unexpected对应。。exitcodes里面的定义的
 
                                 退出码是expected的。
 
;stopsignal=QUIT               ; 进程停止信号，可以为TERM, HUP, INT, QUIT, KILL, USR1, or USR2等信号
 
                                  默认为TERM 。。当用设定的信号去干掉进程，退出码会被认为是expected
 
                                  非必须设置
 
;stopwaitsecs=10               ; 这个是当我们向子进程发送stopsignal信号后，到系统返回信息
 
                                 给supervisord，所等待的最大时间。 超过这个时间，supervisord会向该
 
                                 子进程发送一个强制kill的信号。
 
                                 默认为10秒。。非必须设置
 
;stopasgroup=false             ; 这个东西主要用于，supervisord管理的子进程，这个子进程本身还有
 
                                 子进程。那么我们如果仅仅干掉supervisord的子进程的话，子进程的子进程
 
                                 有可能会变成孤儿进程。所以咱们可以设置可个选项，把整个该子进程的
 
                                 整个进程组都干掉。 设置为true的话，一般killasgroup也会被设置为true。
 
                                 需要注意的是，该选项发送的是stop信号
 
                                 默认为false。。非必须设置。。
 
;killasgroup=false             ; 这个和上面的stopasgroup类似，不过发送的是kill信号
 
;user=chrism                   ; 如果supervisord是root启动，我们在这里设置这个非root用户，可以用来
 
                                 管理该program
 
                                 默认不设置。。。非必须设置项
 
;redirect_stderr=true          ; 如果为true，则stderr的日志会被写入stdout日志文件中
 
                                 默认为false，非必须设置
 
;stdout_logfile=/a/path        ; 子进程的stdout的日志路径，可以指定路径，AUTO，none等三个选项。
 
                                 设置为none的话，将没有日志产生。设置为AUTO的话，将随机找一个地方
 
                                 生成日志文件，而且当supervisord重新启动的时候，以前的日志文件会被
 
                                 清空。当 redirect_stderr=true的时候，sterr也会写进这个日志文件
 
;stdout_logfile_maxbytes=1MB   ; 日志文件最大大小，和[supervisord]中定义的一样。默认为50
 
;stdout_logfile_backups=10     ; 和[supervisord]定义的一样。默认10
 
;stdout_capture_maxbytes=1MB   ; 这个东西是设定capture管道的大小，当值不为0的时候，子进程可以从stdout
 
                                 发送信息，而supervisor可以根据信息，发送相应的event。
 
                                 默认为0，为0的时候表达关闭管道。。。非必须项
 
;stdout_events_enabled=false   ; 当设置为ture的时候，当子进程由stdout向文件描述符中写日志的时候，将
 
                                 触发supervisord发送PROCESS_LOG_STDOUT类型的event
 
                                 默认为false。。。非必须设置
 
;stderr_logfile=/a/path        ; 这个东西是设置stderr写的日志路径，当redirect_stderr=true。这个就不用
 
                                 设置了，设置了也是白搭。因为它会被写入stdout_logfile的同一个文件中
 
                                 默认为AUTO，也就是随便找个地存，supervisord重启被清空。。非必须设置
 
;stderr_logfile_maxbytes=1MB   ; 这个出现好几次了，就不重复了
 
;stderr_logfile_backups=10     ; 这个也是
 
;stderr_capture_maxbytes=1MB   ; 这个一样，和stdout_capture一样。 默认为0，关闭状态
 
;stderr_events_enabled=false   ; 这个也是一样，默认为false
 
;environment=A="1",B="2"       ; 这个是该子进程的环境变量，和别的子进程是不共享的
 
;serverurl=AUTO                ; 
 
; The below sample eventlistener section shows all possible
 
; eventlistener subsection values, create one or more 'real'
 
; eventlistener: sections to be able to handle event notifications
 
; sent by supervisor.
 
;[eventlistener:theeventlistenername] ;这个东西其实和program的地位是一样的，也是suopervisor启动的子进
 
                                       程，不过它干的活是订阅supervisord发送的event。他的名字就叫
 
                                       listener了。我们可以在listener里面做一系列处理，比如报警等等
 
                                       楼主这两天干的活，就是弄的这玩意
 
;command=/bin/eventlistener    ; 这个和上面的program一样，表示listener的可执行文件的路径
 
;process_name=%(program_name)s ; 这个也一样，进程名，当下面的numprocs为多个的时候，才需要。否则默认就
 
                                 OK了
 
;numprocs=1                    ; 相同的listener启动的个数
 
;events=EVENT                  ; event事件的类型，也就是说，只有写在这个地方的事件类型。才会被发送
 
 
 
 
 
;buffer_size=10                ; 这个是event队列缓存大小，单位不太清楚，楼主猜测应该是个吧。当buffer
 
                                 超过10的时候，最旧的event将会被清除，并把新的event放进去。
 
                                 默认值为10。。非必须选项
 
;directory=/tmp                ; 进程执行前，会切换到这个目录下执行
 
                                 默认为不切换。。。非必须
 
;umask=022                     ; 淹没，默认为none，不说了
 
;priority=-1                   ; 启动优先级，默认-1，也不扯了
 
;autostart=true                ; 是否随supervisord启动一起启动，默认true
 
;autorestart=unexpected        ; 是否自动重启，和program一个样，分true,false,unexpected等，注意
 
                                  unexpected和exitcodes的关系
 
;startsecs=1                   ; 也是一样，进程启动后跑了几秒钟，才被认定为成功启动，默认1
 
;startretries=3                ; 失败最大尝试次数，默认3
 
;exitcodes=0,2                 ; 期望或者说预料中的进程退出码，
 
;stopsignal=QUIT               ; 干掉进程的信号，默认为TERM，比如设置为QUIT，那么如果QUIT来干这个进程
 
                                 那么会被认为是正常维护，退出码也被认为是expected中的
 
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
 
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
 
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
 
;user=chrism                   ;设置普通用户，可以用来管理该listener进程。
 
                                默认为空。。非必须设置
 
;redirect_stderr=true          ; 为true的话，stderr的log会并入stdout的log里面
 
                                默认为false。。。非必须设置
 
;stdout_logfile=/a/path        ; 这个不说了，好几遍了
 
;stdout_logfile_maxbytes=1MB   ; 这个也是
 
;stdout_logfile_backups=10     ; 这个也是
 
;stdout_events_enabled=false   ; 这个其实是错的，listener是不能发送event
 
;stderr_logfile=/a/path        ; 这个也是
 
;stderr_logfile_maxbytes=1MB   ; 这个也是
 
;stderr_logfile_backups        ; 这个不说了
 
;stderr_events_enabled=false   ; 这个也是错的，listener不能发送event
 
;environment=A="1",B="2"       ; 这个是该子进程的环境变量
 
                                 默认为空。。。非必须设置
 
;serverurl=AUTO                ; override serverurl computation (childutils)
 
; The below sample group section shows all possible group values,
 
; create one or more 'real' group: sections to create "heterogeneous"
 
; process groups.
 
;[group:thegroupname]  ;这个东西就是给programs分组，划分到组里面的program。我们就不用一个一个去操作了
 
                         我们可以对组名进行统一的操作。 注意：program被划分到组里面之后，就相当于原来
 
                         的配置从supervisor的配置文件里消失了。。。supervisor只会对组进行管理，而不再
 
                         会对组里面的单个program进行管理了
 
;programs=progname1,progname2  ; 组成员，用逗号分开
 
                                 这个是个必须的设置项
 
;priority=999                  ; 优先级，相对于组和组之间说的
 
                                 默认999。。非必须选项
 
; The [include] section can just contain the "files" setting.  This
 
; setting can list multiple files (separated by whitespace or
 
; newlines).  It can also contain wildcards.  The filenames are
 
; interpreted as relative to this file.  Included files *cannot*
 
; include files themselves.
 
;[include]                         ;这个东西挺有用的，当我们要管理的进程很多的时候，写在一个文件里面
 
                                    就有点大了。我们可以把配置信息写到多个文件中，然后include过来
 
;files = relative/directory/*.ini

```
## Supervisor控制台

- 在/etc/supervisord.conf中修改[inet_http_server]的参数，具体如下：
```
[inet_http_server]         ; inet (TCP) server disabled by default
port=*:9001                ; ip_address:port specifier, *:port for all iface
username=root              ; default is no username (open server)
password=xxxx              ; default is no password (open server)
```

- 修改后重启supervisor进程，在浏览器访问 http://<host-ip>:9001。

## 分组管理

- Supervisor 同时还提供了另外一种进程组的管理方式，通过这种方式，可以使用 supervisorctl 命令来管理一组进程，通过在配置文件中添加如下的配置，将多个服务添加至分组中
```
[group:group_name]
programs=progname1,progname2
```

- 注意:
   - 当添加了上述配置后，progname1 和 progname2 的进程名就会变成 group_name:progname1 和 group_name:progname2 以后就要用这个名字来管理进程了，而不是之前的 progname1。
   - 以后执行 supervisorctl stop group_name: 就能同时结束 progname1 和 progname2，执行 supervisorctl stop group_name:progname1 就能结束 progname1

## Supervisor配置开机启动
### 方式一：

- Linux环境中：以systemd的方式管理
   - 编写启动脚本：vim /etc/rc.d/init.d/supervisord
```bash
#!/bin/sh
#
# /etc/rc.d/init.d/supervisord
#
# Supervisor is a client/server system that
# allows its users to monitor and control a
# number of processes on UNIX-like operating
# systems.
#
# chkconfig: - 64 36
# description: Supervisor Server
# processname: supervisord

# Source init functions
. /etc/rc.d/init.d/functions

prog="supervisord"

prefix="/usr"
exec_prefix="${prefix}"
prog_bin="${exec_prefix}/bin/supervisord"
PIDFILE="/var/run/$prog.pid"

start()
{
echo -n $"Starting $prog: "
daemon $prog_bin --pidfile $PIDFILE -c /etc/supervisord.conf
[ -f $PIDFILE ] && success $"$prog startup" || failure $"$prog startup"
echo
}

stop()
{
echo -n $"Shutting down $prog: "
[ -f $PIDFILE ] && killproc $prog || success $"$prog shutdown"
echo
}

case "$1" in

start)
start
;;

stop)
stop
;;

status)
status $prog
;;

restart)
stop
start
;;

*)
echo "Usage: $0 {start|stop|restart|status}"
;;

esac
```

- 设置开机启动及systemd方式启动。
```bash
sudo chmod +x /etc/rc.d/init.d/supervisord
sudo chkconfig --add supervisord
sudo chkconfig supervisord on
sudo service supervisord start
```

### 方式二：
> centos-7

- 进入 / lib/systemd/system 目录，并创建 supervisord.service 文件。
```bash
[Unit]
Description=supervisord
After=network.target
 
[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s
 
[Install]
WantedBy=multi-user.target
```

- 加入开机启动
```bash
chmod 766 /lib/systemd/system/supervisord.service
systemctl daemon-reload
systemctl enable supervisord.service
systemctl start supervisord
systemctl status supervisord
```

## 配置案例:
### Django的配置文件
```
#项目名
[program:django]
#脚本目录
directory=/root/social_engineering/
#脚本执行命令
command=/root/anaconda3/bin/python manage.py runserver 0.0.0.0:8000

#supervisor启动的时候是否随着同时启动，默认True
autostart=true
#当程序exit的时候，这个program不会自动重启,默认unexpected，设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected和true。如果为false的时候，无论什么情况下，都不会被重新启动，如果为unexpected，只有当进程的
退出码不在下面的exitcodes里面定义的
autorestart=true
#这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功了。默认值为1
startsecs=1

#脚本运行的用户身份 

#日志输出 
stderr_logfile=/tmp/django_stderr.log
stdout_logfile=/tmp/django_stdout.log
#把stderr重定向到stdout，默认 false
redirect_stderr = true
#stdout日志文件大小，默认 50MB
stdout_logfile_maxbytes = 20
#stdout日志文件备份数
stdout_logfile_backups = 20

; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```

### Celery的配置文件
```
#项目名
[program:celery]
#脚本目录
directory=/root/social_engineering/
#脚本执行命令
command=/root/anaconda3/bin/celery -A tasks worker --loglevel=info

#supervisor启动的时候是否随着同时启动，默认True
autostart=true
#程序崩溃时自动重启
autorestart=true
#这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功了。默认值为1
startsecs=1

#脚本运行的用户身份 

#日志输出 
stderr_logfile=/tmp/celery_stderr.log
stdout_logfile=/tmp/celery_stdout.log
#把stderr重定向到stdout，默认 false
redirect_stderr = true
#stdout日志文件大小，默认 50MB
stdout_logfile_maxbytes = 20
#stdout日志文件备份数
stdout_logfile_backups = 20
```

### Redis的配置文件
```bash
[program:zedis]
command=/opt/zedis/redis-server /opt/zedis/redis.conf
user=redisuser
autostart=true 								
;启动supervisord的时候会将该配置项设置为true的所有进程自动启动
stopasgroup=true							
;使用supervisorctl停止zedis时，子进程也会一起停止
killasgroup=true							
;向进程组发送kill信号，包括子进程
startsecs=10								
;进程从STARING状态转换到RUNNING状态所需要保持运行10s时间
startretries=3								
;启动失败自动重试次数，默认是3
autorestart=true							
;进程停止后自动启动
stdout_logfile=/var/log/zedis-out
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stderr_logfile=/var/log/zedis-err
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
```

- Ø 因为无法监视后台进程，需要把Zedis前台运行（daemonize no），否则会识别不到Zedis启动，重试启动3次,日志提示6379已占用。
- Ø 因为redis.conf指定了logfile，stdout_logfile并无内容输出；如果不指定redis.conf只是启动redis-server，日志会输出到stdout_logfile。

### tomcat的配置文件
```bash
[program:tomcat]
command=/usr/local/tomcat/bin/catalina.sh run
stdout_logfile=/usr/local/tomcat/logs/catalina.out
stderr_logfile=/usr/local/tomcat/logs/catalina.out
environment=JAVA_HOME="/usr/local/jdk1.8.0_144",JAVA_BIN="/usr/local/jdk1.8.0_144/bin"
autorestart=false
startsecs=60
priority=1
stopasgroup=true
killasgroup=true
```

## 分组管理的配置文件

- 新建文件，fastapi_app.conf
```
[group:fastapi]
programs=fastapi-app

[program:fastapi-1]
command=/home/python/scripts/fastapi1_app.sh
directory=/home/python/fastapi-1-backend
user=python
autorestart=true
redirect_stderr=false
loglevel=info
stopsignal=KILL
stopasgroup=true
killasgroup=true

[program:fastapi-2]
command=/home/python/scripts/fastapi-2.sh
directory=/home/python/fastapi-2-backend
user=python
autorestart=true
redirect_stderr=false
loglevel=info
stopsignal=KILL
stopasgroup=true
killasgroup=true
```

## 常见问题：
### 问题1：unix:///tmp/supervisor.sock no such file的问题

- 在supervisor默认配置中，其启动的sock等都会放到tmp目录，而tmp目录会自动清理导致无法使用supervisorctl
```
vim /etc/supervisord.conf

#step1: 将配置文件中的tmp替换掉，否则容易被linux自动清除
/tmp/supervisor.sock 改成 /var/run/supervisor.sock，
/tmp/supervisord.log 改成 /var/log/supervisor.log，
/tmp/supervisord.pid 改成 /var/run/supervisor.pid

# step2:修改权限，如果不修改，启动时会报错无权限
sudo chmod 777 /run
sudo chmod 777 /var/log

# step3: 创建supervisor.sock
sudo touch /var/run/supervisor.sock
sudo chmod 777 /var/run/supervisor.sock

# step4: 杀掉原来的进程，重新启动
supervisord -c /etc/supervisord.conf
```

### 问题2：command中指定的进程已经起来，但supervisor还不断重启

- command中启动方式为后台启动，导致识别不到pid，然后不断重启，这里使用的是elasticsearch，command指定的是$path/bin/elasticsearch -d
```bash
解决办法：supervisor无法检测后台启动进程的pid，而supervisor本身就是后台启动守护进程，因此不用担心这个
```

### 问题3：启动了多个supervisord服务，导致无法正常关闭服务

- 在运行supervisord -c /etc/supervisord.conf之前，直接运行过supervisord -c /etc/supervisord.d/xx.conf导致有些进程被多个superviord管理，无法正常关闭进程
```bash
解决办法：使用ps -fe | grep supervisord查看所有启动过的supervisord服务，kill相关的进程。
```
