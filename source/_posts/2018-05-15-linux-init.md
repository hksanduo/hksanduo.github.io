---
title: "init进程"
date:   2018-05-15 12:00 +0800
layout: post
tag: 
- Kernel
categories:
- Linux
---

# init进程
    
init的进程号为1,是所有进程的父进程，内核初始化完毕之后，init程序开始运行。其他软件也同时开始运行。init程序通过/etc/inittab文件进行配置。inittab文件的内容如下：
引用内容：

    # inittab       This file describes how the INIT process should set up
    #               the system in a certain run-level.
    #
    # Author:       Miquel van Smoorenburg, <miquels@drinkel.nl.mugnet.org>
    #               Modified for RHS Linux by Marc Ewing and Donnie Barnes
    #
                                                                                                                             
    # Default runlevel. The runlevels used by RHS are:
    #   0 - halt (Do NOT set initdefault to this)
    #   1 - Single user mode
    #   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
    #   3 - Full multiuser mode
    #   4 - unused
    #   5 - X11
    #   6 - reboot (Do NOT set initdefault to this)
    #    
    id:3:initdefault:
                                                                                                                             
    # System initialization.
    si::sysinit:/etc/rc.d/rc.sysinit
                                                                                                                             
    l0:0:wait:/etc/rc.d/rc 0
    l1:1:wait:/etc/rc.d/rc 1
    l2:2:wait:/etc/rc.d/rc 2
    l3:3:wait:/etc/rc.d/rc 3
    l4:4:wait:/etc/rc.d/rc 4
    l5:5:wait:/etc/rc.d/rc 5
    l6:6:wait:/etc/rc.d/rc 6

    # Things to run in every runlevel
    ud::once:/sbin/update
                                                                                                                             
    # Trap CTRL-ALT-DELETE
    ca::ctrlaltdel:/sbin/shutdown -t3 -r now
                                                                                                                             
    # When our UPS tells us power has failed, assume we have a few minutes
    # of power left.  Schedule a shutdown for 2 minutes from now.
    # This does, of course, assume you have powerd installed and your
    # UPS connected and working correctly.
    pf::powerfail:/sbin/shutdown -f -h +2 "Power Failure; System Shutting Down"
                                                                                                                             
    # If power was restored before the shutdown kicked in, cancel it.
    pr:12345:powerokwait:/sbin/shutdown -c "Power Restored; Shutdown Cancelled"


    # If power was restored before the shutdown kicked in, cancel it.
    pr:12345:powerokwait:/sbin/shutdown -c "Power Restored; Shutdown Cancelled"
                                                                                                                             
                                                                                                                             
    # Run gettys in standard runlevels
    1:2345:respawn:/sbin/mingetty tty1
    2:2345:respawn:/sbin/mingetty tty2
    3:2345:respawn:/sbin/mingetty tty3
    4:2345:respawn:/sbin/mingetty tty4
    5:2345:respawn:/sbin/mingetty tty5
    6:2345:respawn:/sbin/mingetty tty6                                                                                                                             
    # Run xdm in runlevel 5
    x:5:respawn:/etc/X11/prefdm -nodaemon


Runlevel 0是让init关闭所有进程并终止系统。

Runlevel 1是用来将系统转到单用户模式，单用户模式只能有系统管理员进入，在该模式下处理那些在有登录用户的情况下不能进行更改的文件，改runlevel的编号1也可以用S代替。

Runlevel 2是允许系统进入多用户的模式，但并不支持文件共享，这种模式很少应用。

Runlevel 3是最常用的运行模式，主要用来提供真正的多用户模式，也是多数服务器的缺省模式。

Runlevel 4一般不被系统使用，用户可以设计自己的系统状态并将其应用到runlevel 4阶段，尽管很少使用，但使用该系统可以实现一些特定的登录请求。

Runlevel 5是将系统初始化为专用的X Window终端。对功能强大的Linux系统来说，这并不是好的选择，但用户如果需要这样，也可以通过在runlevel启动来实现该方案。

Runlevel 6是关闭所有运行的进程并重新启动系统。

在inittab文件中以#开头的所有行都是注释行。注释行有助于用户理解inittab文件，inittab文件中的值都是如下格式：

	label:runlevel:action:process
    
label是1~4个字符的标签，用来标示输入的值。一些系统只支持2个字符的标签。鉴于此原因，多数人都将标签字符的个数限制在2个以内。该标签可以是任意字符构成的字符串，但实际上，某些特定的标签是常用的，在Red Hat Linux中使用的标签是：
代码:

    id 用来定义缺省的init运行的级别
    si 是系统初始化的进程
    ln 其中的n从1~6,指明该进程可以使用的runlevel的级别
    ud 是升级进程
    ca 指明当按下Ctrl+Alt+Del是运行的进程
    pf 指当UPS表明断电时运行的进程
    pr 是在系统真正关闭之前，UPS发出电源恢复的信号时需要运行的进程
    x  是将系统转入X终端时需要运行的进程
    
runlevel字段指定runlevel的级别。可以指定多个runlevel级别，也可以不为runlevel字段指定特定的值。
    
action字段定义了该进程应该运行在何种状态下：
代码:
    
    boot        在系统启动时运行，忽略runlevel
    bootwait    在系统启动时运行，init等待进程完成。忽略runlevel
    ctrlaltdel    当Ctrl+Alt+Del三个键同时按下时运行，把SIGINT信号发送给init。忽略    runlevel
    initdefault    不要执行这个进程，它用于设置默认runlevel
    kbrequest    当init从键盘中收到信号时运行。这里要求键盘组合符合KeyBoardSigral(参见/usr/share/doc/kbd-*关于键盘组合的文档)
    off        禁止进入，因此该进程不运行
    once        每一个runlevel级别运行一次
    ondemand    当系统指定特定的运行级别A、B、C时运行
    powerfail    当init收到SIGPWR信号时运行
    powerokwait    当收到SIGPWD信号且/etc/文件中的电源状态包含OK时运行
    powerwait    当收到SIGPWD信号，并且init等待进程结束时运行
    respawn        不管何时终止都重新启动进程
    sysinit        在运行boot或bootwait进程之前运行
    wait        运行进程等待输入运行模式
    process 字段包含init执行的进程，该进程采用的格式与在命令行下运行该进程的格式一样，因此process字段都以该进程的名字开头，紧跟着是运行时，紧跟着是运行时要传递给该进程的参数。比如/sbin/shutdown -t3 -r now，该进程在按下Ctrl+Alt+Del时执行，在命令行下也可以直接输入来重新启动系统。
    
特殊目的的记录   
    仔细学习例子文件，学习应用其中关于inittab的语法格式。该文件的大多数内容都可以忽略，因为超过一半的内容都是注释，剩余的一些文件内容主要是用来实现某些特殊的功能：
    
    id 的值表明缺省的runlevel是3。
    ud 的值可以唤醒/sbin/update进程，该进程为保持磁盘的完整性，将在对磁盘进行I/O操作之前清空整个I/O缓冲区。
    pf、pr和ca的值只被特定的中断所调用。

如果系统是专用的X终端，则只需x的输入值。
getty进程来提供虚拟终端设备的服务，例如：

	3:2345:respawn:/sbin/mingetty tty3
    
标签字段的值是3,3是设备tty3的数字后缀,tty3与相应的进程相关联，该getty进程可以启动的runlevel是2、3、4和5,当该进程终止时，init马上就重新启动它。启动进程的路径名是/sbin/mingetty，该进程是实现虚拟终端支持的最小版本的getty，为tty3提供启动虚拟设备的进程。

	si::sysinit:/etc/rc.d/rc.sysinit

该值告诉init程序运行/etc /rc.d/rc.sysinit脚本文件来初始化系统，该脚本文件与所有启动的脚本类似，它只是一个包含Linux的 shell命令的可执行文件，注意输入的字符串必须包括该脚本的完整路径。不同版本的Linux存放该脚本的位置也不相同，但不用刻意去记忆这些位置，只需查看/etc/inittab 文件即可，该文件中包含启动脚本文件的确切位置。
