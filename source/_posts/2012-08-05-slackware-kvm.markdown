---
layout: post
title: 服务器基于slackware的KVM布署详解
categories:
- server
tags:
- bridge
- kvm
- slackware
- tap
- tun
- virtio
- server
- 虚拟机
published: true
comments: true
---

### 一、背景 

首先呢，我发现自己有半年时间没有写技术类的文章了，其次，关于这个话题，我一直都想整理一篇比较好的文章出来，一来可以做个备忘，更重要是起到分享交流，共同进步。

之前一直可能都有一些零零散散关于这部分细节的，但我觉得都看得不是很爽。

另外，网上搜到大多数关于KVM的都是debian系或redhat系的，有部分内容和slackware有差异，并且由于slackware的文化，在KVM虚拟化这块，相比其它发行版要麻烦一点点，而slackware的简洁、稳定、快速是我一直所爱，所以更希望写下这篇文章。

### 二、相关准备
* 装有slackware的主机一台，内存最好不要太少吧，如果4G或以上的，就装slackware64，当前slackware最新稳定版是13.37（37这个命名方式是和内核2.6.xx一致），因为是做物理主机的系统，所以稳定至上。至于如何安装slackware，可以看[slackbook中文译本](http://www.linuxsir.org/bbs/thread383703.html)，反正基本流程是：光碟或U盘启动 -> cfdisk分区 -> setup安装 -> 重启……，下载slackware可以到[这里](http://taper.alienbase.nl/mirrors/slackware/)
* KVM是什么，可以了解，随便一搜就有，还要确定你的CPU是否支持虚拟化技术；（[什么是虚拟化技术？](http://baike.baidu.com/view/13605.htm)）
* 因为我建立两个kvm，一个是slackware-current，一个是windows 2003，所以你也得要有2003的iso，安装2003请选择手动安装，最好是MSDN版本；
* 虚拟机访问外网要使用网桥（package bridge-utils），但听说用VDE更方面，老实说，VDE我弄了一下，好像很麻烦就没弄了。

{% img /images/blog/servers.jpg %}

### 三、软件准备 
物理服务器机器安装好slackware64
13.37后，那得得要准备相关的软件环境（均运行在root下面）：**qemu-kvm**和**tunctl**（用来tap通讯）

* qemu-kvm

首先到slackbuilds.org下载[qemu-kvm](http://slackbuilds.org/repository/13.37/system/qemu-kvm/)相关的编译准备文件（还有源码包），运行qemu-kvm.SlackBuild进行编译，OK后会在tmp生成一个名为qemu-kvm-1.0.1-x86_64-1_SBo.tgz的包，installpkg之。如果在编译过程中出现错误，那就得先排错，一般都是相关的依赖问题。

* tunctl

同样地，下载[tunctl](http://slackbuilds.org/repository/13.37/network/tunctl/)并编译之，得到tunctl-1.5-x86_64-2_SBo.tgz并installpkg之
这部分完毕

### 四、设置网桥和Tap 
KVM虚拟机通过tap来访问互联网，那么物理主机得设置好网桥，下面是我/etc/rc.d/rc.local里设置网桥和tap的代码 
```bash
# create brige 
ifconfig eth0 0.0.0.0 
brctl addbr br0 
ifconfig br0 x.x.x.x(物理主机IP) netmask 255.255.255.0(物理主机掩码) up 
route add default gw x.x.x.x(网关) br0 
brctl addif br0 eth0
```
```bash
# create tap0 for windows 2003 
tunctl -t tap0 
ifconfig tap0 up 
brctl addif br0 tap0
# create tap1 for slackware-current 
tunctl -t tap1 
ifconfig tap1 up 
brctl addif br0 tap1
```
### 五、安装windows server 2003 
windows方面我们使用virtio来提高磁盘和网络的IO性能，那么这里得准备virtio的两个文件：[virtio-win-0.1-30.iso](http://alt.fedoraproject.org/pub/alt/virtio-win/latest/images/bin/virtio-win-0.1-30.iso)和[virtio-win-1.1.16.vfd](http://www.linuxwind.org/download/virtio-win-1.1.16.vfd)

来说明一下，virtio-win-0.1-30.iso里面放有windows的virtio磁盘控制器和网卡的驱动，是安装2003成功后，进入系统后进行相关驱动安装或更新用的。而virtio-win-1.1.16.vfd是一个软盘镜像，是用来安装2003时，通过F6来指定磁盘驱动用的。
在使用qemu-kvm前，需要载入kvm内核模块： 
```
$ modprobe kvm_intel
```
创建大小为20G的win2003.qcow2磁盘文件： 
```
$ qemu-img create -f qcow2 win2003.qcow 20G
```
启动KVM进行2003的安装： 
```
$ qemu-kvm -m 768 -boot d -drive file=/xxx/win2003.qcow2,cache=writeback,if=virtio -fda /xxx/virtio-win-1.1.16.vfd -cdrom /xxx/Windows.Server.2003.R2.iso -vnc :1 
```
命令运行后，没有提示任何错误就应该是没有问题了，打开tightVNC，登录到该远程桌面进行安装的操作（VNC登录地址是IP1:1）。启动安装是按F6，之后再安“S”，进入后选择“Redhat I/O for windows 2003”相应的驱动，32或64位。之后的安装过程是和平时装2003的过程一样。
安装成功并启动一次，之后将2003关机，换用另外一条指令来启动虚拟机： 
```
$ qemu-kvm -m 768 -net nic,model=virtio -net tap,ifname=tap0,script=no -drive file=/xxx/win2003.qcow2,cache=writeback,if=virtio -cdrom /xxx/virtio-win-0.1-30.iso -vnc :1 
```
此次启动将载入virtio-win-0.1-30.iso镜像，包含有磁盘控制器和网卡的驱动
启动后，同样使用VNC远程接入，把2003里的网卡安装好，也可以把磁盘控制器也升级一下，因为之前软盘镜像带的是2010年的，现在iso里包含的是最新的驱动

{% img /images/blog/tm1.png %}

到这里，windows 2003安装步骤就完毕了，之后就像平时新装系统设定IP，网关什么的就好，并开启windows远程桌面方便后续的远程操作
六、安装slackware-current 另一台虚拟机我装的是slackware-current，安装步骤要比windows 2003简单一些
创建大小为20G的slackware.qcow2磁盘文件： 
```
$ qemu-img create -f qcow2 slackware.qcow 20G
```
启动KVM进行slackware的安装： 
```
$ qemu-kvm -m 768 -boot d -drive file=/xxx/slackware.qcow2,cache=writeback -cdrom /xxx/slackware-13.37-install-d1.iso -vnc :1
```
用VNC连接，像平时一样来安装，current的安装方法就略去了
安装成功后就换另外一条指令来启动： 
```
$ qemu-kvm -m 768 -net nic -net tap,ifname=tap1,script=no -drive file=/xxx/slackware.qcow2,cache=writeback -vnc :1
```
之后进行一些必要的设定，之后就可以用ssh来连接

### 六、加入rc.local
两台KVM虚拟机都安装设置成功了，所以就需要将其加入到rc.local脚本，使两个虚拟机在开机时随系统启动，下面是我的rc.local： 
```bash
# create brige 
ifconfig eth0 0.0.0.0 
brctl addbr br0 
ifconfig br0 x.x.x.x(物理主机IP) netmask 255.255.255.0(物理主机掩码) up 
route add default gw x.x.x.x(网关) br0 
brctl addif br0 eth0
# create tap0 for windows 2003 
tunctl -t tap0 
ifconfig tap0 up 
brctl addif br0 tap0
# create tap1 for slackware-current 
tunctl -t tap1 
ifconfig tap1 up 
brctl addif br0 tap1
# macaddr 
macaddr1=`printf ‘DE:AD:BE:EF:%02X:%02X\n’ $((RANDOM%256)) $((RANDOM%256))` 
macaddr2=`printf ‘DE:AD:BE:EF:%02X:%02X\n’ $((RANDOM%256)) $((RANDOM%256))`
# start kvm
qemu-kvm -m 768 -drive file=/home/qcow2/win2003.qcow2,cache=writeback,if=virtio -net nic,model=virtio,macaddr=$macaddr1 -net tap,ifname=tap0,script=no -daemonize -nographic -localtime 
qemu-kvm -m 768 -drive file=/home/qcow2/slackware.qcow2,cache=writeback -net nic,macaddr=$macaddr2 -net tap,ifname=tap1,script=no -daemonize -nographic -localtime 
```
启动参数多了个mac地址，是用随机数生成的，建议生成mac地址后将其固定，不然每次启动机器都会有不同的mac地址……
另外，因为不需要用vnc，所以参数里也没有vnc的启动参数，这样也是保证安全的一个做法

* -daemonize：让KVM在后台运行
* -nographic：关闭图形输出
* -localtime：让虚拟机的时间显示正常

### 七、总结与展望
到这里，文章也快要结束了。
文中没有使用libvirt来进行虚拟机管理，这个后续可以进行改进，并且也没有相应的watchdog，这也是一个不足。
文中所讲述是一个相对为通用的方法，希望读者能有所启发。
如果你按照这些步骤进展顺利，那么恭喜你:)
