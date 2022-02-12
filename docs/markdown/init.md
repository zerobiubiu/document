# init进程

## 一、什么是init进程

在介绍init进程前我们先了解下什么是进程

### 1.进程的概念

所谓进程就是系统中正在运行的程序，进程是操作系统的概念，每当我们执行一个程序时，对于操作系统来讲就是创建了一个进程，在这个过程中操作系统对进程资源的分配和释放，可以认为进程就是一个程序的一次执行过程。

### 2.Linux下的三个特殊进程

Linux下有三个特殊的进程idle进程（PID=0），init进程（PID=1），和kthreadd（PID=2）
**idle进程由系统自动创建，运行在内核态**
idle进程其pid=0，其前身是系统创建的第一个进程，也是唯一一个没有通过fork或者kernel_thread产生的进程。完成加载系统后，演变为进程调度、交换。
**kthreadd进程由idle通过kernel_thread创建，并始终运行在内核空间，负责所有内核进程的调度和管理。**
它的任务就是管理和调度其他内核线程kernel_thread, 会循环执行一个kthread的函数，该函数的作用就是运行kthread_create_list全局链表中维护的kthread, 当我们调用kernel_thread创建的内核线程会被加入到此链表中，因此所有的内核线程都是直接或者间接的以kthreadd为父进程 。
**init进程由idle通过kernel_thread创建，在内核空间完成初始化后，加载init程序**
在这里我们就主要讲解下init进程，init进程由0进程创建，完成系统的初始化，是系统中所有其他用户进程的祖先进程
Linux中的所有进程都是由init进程创建并运行的。首先Linux内核启动，然后在用户空间中启动init进程，再启动其他系统进程。在系统启动完成后，init将变成为守护进程监视系统其他进程。
所以说init进程是Linux系统操作中不可缺少的程序之一，如果内核找不到init进程就会试着运行/bin/sh，如果运行失败，系统的启动也会失败。

## 二、运行级别

init服务的配置文件是/etc/inittab
在centos7之前inittab的配置文件是这样的

```html
# inittab is only used by upstart for the default runlevel.
#
# ADDING OTHER CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# System initialization is started by /etc/init/rcS.conf
#
# Individual runlevels are started by /etc/init/rc.conf
#
# Ctrl-Alt-Delete is handled by /etc/init/control-alt-delete.conf
#
# Terminal gettys are handled by /etc/init/tty.conf and /etc/init/serial.conf,
# with configuration in /etc/sysconfig/init.
#
# For information on how to write upstart event handlers, or how
# upstart works, see init(5), init(8), and initctl(8).
#
# Default runlevel. The runlevels used are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
# 
id:5:initdefault:
1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.
```

里面介绍了init的6个运行级别
0是关机

1是单用户

2是多用户，不联网

3是多用户

4是不使用的

5是xwindows，也就是有界面的

6是重启

init命令很简单。直接输入init + 你想要的模式 回车就行。
我们可以使用runlevel命令来查询当前系统的运行级别。
比如 输入 ： init 0 就是关机

init 3 就是切换到多用户

init 5 就是切换到界面

init 6 就是重启
但是千万不要把initdefault设置为0或者6
但是在centos7之后有了一个新的服务systemd取代了init，
systemd 是 Linux 下一个与 SysV 和 LSB 初始化脚本兼容的系统和服务管理器。在这里我也不过多介绍了，大家有兴趣可以自行研究下。

## 三、init运行级别的定义

init的运行级别配置是在/etc/init，而这些级别的定义是在/etc/rc.d目录内的如下：

```html
[root@centos6 rc.d]# ll
total 60
drwxr-xr-x. 2 root root  4096 Jan  9 02:30 init.d
-rwxr-xr-x. 1 root root  2617 Mar 23  2017 rc
drwxr-xr-x. 2 root root  4096 Jan  9 02:30 rc0.d
drwxr-xr-x. 2 root root  4096 Jan  9 02:30 rc1.d
drwxr-xr-x. 2 root root  4096 Jan  9 18:39 rc2.d
drwxr-xr-x. 2 root root  4096 Jan  9 18:39 rc3.d
drwxr-xr-x. 2 root root  4096 Jan  9 18:39 rc4.d
drwxr-xr-x. 2 root root  4096 Jan  9 18:39 rc5.d
drwxr-xr-x. 2 root root  4096 Jan  9 02:30 rc6.d
-rwxr-xr-x. 1 root root   220 Mar 23  2017 rc.local
-rwxr-xr-x. 1 root root 20199 Mar 23  2017 rc.sysinit

1.2.3.4.5.6.7.8.9.10.11.12.13.14.
```

这里的rc{0…6}.目录对应相应的级别里面放的都是要启动和关闭的进程我们进去看一下

```html
[root@centos6 rc3.d]# ls
K01smartd          K69rpcsvcgssd      K95firstboot     S13irqbalance        S26udev-post
K02oddjobd         K73winbind         K99rngd          S13rpcbind           S28autofs
K05wdaemon         K74ntpd            S01sysstat       S15mdmonitor         S50bluetooth
K10psacct          K75ntpdate         S02lvm2-monitor  S22messagebus        S55sshd
K10saslauthd       K75quota_nld       S05rdma          S23NetworkManager    S80postfix
K15htcacheclean    K76ypbind          S08ip6tables     S24nfslock           S82abrtd
K15httpd           K84wpa_supplicant  S08iptables      S24rpcgssd           S83abrt-ccpp
K30spice-vdagentd  K87restorecond     S10network       S25blk-availability  S90crond
K50dnsmasq         K88sssd            S11auditd        S25cups              S95atd
K50kdump           K89netconsole      S11portreserve   S25netfs             S99certmonger
K60nfs             K89rdisc           S12rsyslog       S26acpid             S99local
K61nfs-rdma        K92pppoe-server    S13cpuspeed      S26haldaemon

1.2.3.4.5.6.7.8.9.10.11.12.13.14.
```

这里以K开头的都是要关闭的进程，而以S开头的则是要启动的进程

```html
[root@centos6 rc6.d]# ls
K01certmonger    K25sshd            K74haldaemon         K84wpa_supplicant  K90network
K01smartd        K30postfix         K74ntpd              K85mdmonitor       K92ip6tables
K02oddjobd       K30spice-vdagentd  K75blk-availability  K85messagebus      K92iptables
K05atd           K50dnsmasq         K75netfs             K87irqbalance      K92pppoe-server
K05wdaemon       K50kdump           K75ntpdate           K87restorecond     K95firstboot
K10cups          K60crond           K75quota_nld         K87rpcbind         K95rdma
K10psacct        K60nfs             K75udev-post         K88auditd          K99cpuspeed
K10saslauthd     K61nfs-rdma        K76ypbind            K88rsyslog         K99lvm2-monitor
K15htcacheclean  K69rpcsvcgssd      K83bluetooth         K88sssd            K99rngd
K15httpd         K72autofs          K83nfslock           K89netconsole      K99sysstat
K16abrt-ccpp     K73winbind         K83rpcgssd           K89portreserve     S00killall
K16abrtd         K74acpid           K84NetworkManager    K89rdisc           S01reboot

1.2.3.4.5.6.7.8.9.10.11.12.13.14.
```

像rc6.d目录中基本都是要关闭的进程，只有S00killall和S01reboot这两个要启动的进程他们分别是结束所有进程和重启系统。这里文件中的数字代表了他们的优先级，数字越小优先启动。所以我们自己做的服务放在这个目录中时要谨慎以免因为他所需的关联程序没有启动而导致进程无法启动。
PS:如果真的不小心把init默认运行级别设置为0或6的解决办法
我们知道init0和6级别分别对应的是关机和重启，如果把这两个设为默认运行级别我们是无法进入系统的，所以我们就要借助救援系统了，在开机GRUB界面按e如下:
![浅谈init进程_进程](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/229e25153e9c4b03d9cda3355ce1216a.png)
选择kernel这行接着按e
![浅谈init进程_init_02](http://i2.51cto.com/images/blog/201804/19/e13e274dbfe5ac2ce41029fe595bac93.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
在命令行最后输入1（进入单用户模式），回车退后到上个界面
![浅谈init进程_Linuxinit_03](http://i2.51cto.com/images/blog/201804/19/b76c1a8f9a165f85dd806588ebabbf4a.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
接着按b进入单用户模式，我们这就进入到单用户模式了
![浅谈init进程_进程_04](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/77f56f7eeaf8d757cfd4d815c8091588.png)
我们只需要进入/etc/inittab配置文件中把最后的0或6改为3，重启系统就可以啦
![浅谈init进程_进程_05](http://i2.51cto.com/images/blog/201804/19/3b63444b848949446682e5548962dd3c.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)