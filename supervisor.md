# Supervisor

### 概述

Supervisor是一个客户端/服务器系统，它允许用户在类UNIX操作系统上控制多个进程，它的灵感来源于：

* 方便性
    通常来说，为 每一个单进程编写 `rc.d` 脚本是不方便的。`rc.d` 脚本是初始化/自动启动/管理进程的最低层的方式，但是写入和维护会很痛苦。此外，`rc.d` 脚本不能自动重启一个奔溃的进程，并且许多程序在奔溃时不能正确启动。Supervisor将进程作为其子进程启动，同时可以配置为奔溃时自动重启。它还可以配置为在自己的调用上启动进程。

* 准确性
    通常来说，准确地在UNIX上获取进程的状态是困难的。Pidfiles文件总是不可靠的。Supervisor通过将启动的进程作为子进程，所以它总是可以知晓子进程的真实运行状态，并且可以方便地被查询数据。

* 代理
    需要控制进程状态的用户通常只需要这样做。他们不希望或不需要对运行进程的机器进行完全shell访问。在“低”TCP端口上侦听的进程通常需要以root用户的身份启动和重新启动。通常来说，允许人为地停止或重新启动这样的进程是完全正常的，但是提供shell访问通常是不切实际的，并且为他们提供root访问或sudo访问更是不可能的。 `Supervisorctl` 允许非常有限方式来访问机器，基本上只允许用户查看进程状态，并且可以通过从shell或web UI发出“停止”，“开始”和“重新启动”命令来控制子进程。

* 进程组
    进程总是需要被成组地启动和停止，有时甚至以“优先级顺序”启动和停止。通常很难向人们解释如何做到这一点。Supervisor允许您为进程分配优先级，并允许用户通过supervisorctl客户端发出命令，如“start all”和“restart all”，以预先分配的优先级顺序启动它们。另外，可以将进程分组为“进程组”，并且可以停止并开始一组逻辑相关的进程作为一个单元。

### 特点

* 简单
    Supervisor通过一个简单的ini风格的配置文件完成配置，它提供了许多进程选项，例如重启失败进程和自动日志轮换。

* 集中
    Supervisor为您提供一个地方来启动，停止和监视您的进程。进程可以单独或成组控制。您可以配置Supervisor以提供本地或远程命令行和Web界面。

* 高效
    Supervisor通过fork/exec启动其子进程，并且子进程不进行守护进程。操作系统在进程终止时立即向Supervisor发出信号，这与依赖于麻烦的PID文件和定期轮询来重新启动失败进程的解决方案不同。

* 可扩展
    Supervisor有一个简单的事件通知协议，用任何语言编写的程序都可以使用它来监视它，还有一个XML-RPC接口用于控制。它也构建有可以被Python开发人员利用的扩展点。

* 兼容
    Supervisor只适用于除Windows之外的一切。它在Linux，Mac OS X，Solaris和FreeBSD上进行测试和支持。它完全用Python编写，因此安装不需要C编译器。

### Supervisor 组件

* Supervisord
    Supervisor的服务部分名称为 supervisord。它负责在自己的调用中启动子程序，响应来自客户端的命令，重新启动崩溃或退出的子进程，记录其子进程 stdout 和 stderr 输出，以及生成和处理对应于子进程生命周期中的点的“事件”。服务器进程使用配置文件。这通常位于 /etc/supervisord.conf。此配置文件是一个“Windows-INI”样式的配置文件。通过适当的文件系统权限保持此文件安全是很重要的，因为它可能包含未加密的用户名和密码。

* Supervisorctl
    Supervisor的命令行客户端名称为 supervisorctl。它为 supervisord 提供的功能提供了一个shell样界面。从 supervisorctl，用户可以连接到不同的 supervisord 进程，在由子进程控制的子进程上获取状态，停止和启动子进程，并获取 supervisord 的运行进程列表。

* Web Server
    可以通过浏览器访问具有与 supervisorctl 相当的功能的Web用户界面从而查看和控制进程状态，需要先激活配置文件的 [inet_http_server] 部分。


### 使用

> echo_supervisord_conf > /etc/supervisord.conf

如果您没有root访问权限，或者您不想将 supervisord.conf 文件放在 /etc/supervisord.conf` 中，则可以将其放在当前目录（echo_supervisord_conf > supervisord.conf）中，并使用 -c 标志启动 supervisord 以指定配置文件位置。

### 配置

* [program:x]
    supervosord.conf文件必须包含一个或多个program段，以便supervisord知道应该启动和控制哪些程序。
    
    | 选项 | 作用 |
    | - | - | - |
    | command | 此程序启动时将运行的命令。命令可以是绝对路径，也可以是相对路径。如果是相对的，将在supervisord的环境变量 `$PATH` 中搜索可执行文件。 
    | process_name | 一个用于组成此进程名称的Python字符串表达式，默认值是 `%(program_name)s`  |
    | numprocs | Supervisor将启动该数量的进程。如果numprocs> 1，则 process_name 表达式必须包含 %(process_num)s   |
    | numprocs_start | 用于计算 numprocs 开始的数字的整数偏移量。  |
    | priority |  进程的优先级，较低优先级的进程最先启动和晚关闭，较高优先级的进程晚启动早关闭。 |
    | autostart | 进程是否自动启动  |
    | startsecs | 启动后，程序需要保持运行以考虑启动成功（将进程从 STARTING 状态移动到 RUNNING 状态）的总秒数 |
    | startretries | 错误启动尝试次数 |
    | autorestart | 控制 supervisord 是否会在程序成功启动后退出时自动重新启动程序 |
    | exitcodes | 与 autorestart 一起使用的此程序的“预期”退出代码列表。如果 autorestart 参数设置为 unexpected，并且进程以除了Supervisor停止请求的结果之外的任何其他方式退出，则如果进程退出并且未在此列表中定义退出代码，则 supervisord 将重新启动该进程。|
    | stopsignal | Supervisor请求停止时用于终止程序的信号。这可以是TERM，HUP，INT，QUIT，KILL，USR1或USR2中的任何一个。|
| stopwaitsecs | 在程序发送一个停止信号后等待操作系统将SIGCHLD返回到 supervisord 的秒数。如果在 supervisord 从进程接收到SIGCHLD之前经过了这个秒数，supervisord 将尝试使用最终的SIGKILL来杀死它。|
| stopasgroup | |
| killasgroup | |
| user | 指示 supervisord 将此UNIX用户帐户用作运行程序的帐户。只有在以root用户身份运行 supervisord 时，才能切换用户。如果 supervisord 无法切换到指定的用户，程序将不会启动。|
| **redirect_stderr** | 如果为true，则进程的stderr输出在其stdout文件描述符上被发送回 supervisord |
| stdout_logfile | 将进程stdout输出放在此文件中（如果redirect_stderr为true，还将stderr输出放在此文件中） |
| stdout_logfile_maxbytes | stdout_logfile 日志文件最大字节数（可以在值中使用类似“KB”，“MB”和“GB”的后缀乘法器）。将此值设置为0表示无限日志大小。 |
| environment | 设置子进程的环境变量中 |

    #### command 示例

    > /path/to/program foo bar 
    > /path/to/program -p "foo bar" 
    > /path/to/program --port=80%(process_num)02d
    
    可用的变量有 `group_name`, `host_node_name`, `process_num`, `program_name` 以及所有以 `ENV_` 为前缀的环境变量。

* [group:x] 
    将“同类”进程组组合在一起，可以作为一个单元并通过Supervisor的各种控制器接口进行控制。
