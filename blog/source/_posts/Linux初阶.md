---
title: linux初级
date: 2017-04-26 11:32:30
tags:
- linux
categories:
- 技术
---

#####基本环境：
centos7、centos6.5

deb后缀的软件包是for Debian系的（包括Ubuntu），不是给centos安装的。
rpm后缀的软件包才是for Redhat系的（包括CentOS）。

如果想安装deb包，要安装dpkg命令~
或者是用alien把.deb转化为rpm，然后运行~
yum是操作源组（里面各种官方程序），而rpm是操作自己的rpm包（自己下载过来的）

#####坑一
直接从windows拖到vm可能会导致文件缩小（丢失信息）
#####坑二
使用rpm可能需要依赖包，使用yum安装（yum search ***:查询***的yum源）
#####坑三
如果你的linux采用最小安装（没有图形界面等等），很多命令是默认没有安装的，所以你使用坑二或是在线下载的方式，结果各种错误，此时就需要注意你的网络试试：ip addr；发现你的ip并没有分配，service network start; 开启网络吧!(记得修改配置文件ONBOOT=yes--开机启动网络)
接着yum search \*，查询到位置后，yum install \*。(有的命令需要updatedb,比如locate)

#####坑四
突然有天别人告诉我，所有账户都登录不上（包括root），后来直接通过阿里的控制台登录成功，然后查看原因
一般远程登录不上，先检查端口22(负责远程连接的端口),发现root权限竟然无法开放和关闭端口，脚本的所有权已被修改，而且每个文件的所有权都属于某用户，原来他误操作(chmod *** /)，当时他没发现可怕的事情，也就没放在心上；之后就炸了。
把所有的文件属性都改为root，然后单独修改每个账户的文件夹（包括子文件夹）的权限 (-R：递归修改 子文件)
chmod -R /  权限修改
chgrp 权限组修改
chown -R tangyb.bxGroup ./polaris 修改文件属性（权限、组等等）


#####坑五
个别用户登录不上（显示建立连接通道，然后就和卡机了一样），查看登录日志（文件：/var/log/secure），发现错误，此用户的进程数过多（默认是1024） ，提升进程数（文件：90-nproc.conf）或是关闭此用户下的进程（pkill -u username）。

#####坑六
突然某天vm不能远程了
主机不能ping通vm，vm可以ping通主机并且可以上网（采用的nat模式）
后改为桥接模式，主机可以连接但是vm无法上网（公司绑定了mac地址）,尴尬！
因为以前是可以的，所以不存在网上说的nat不能被远程的情况，仔细想想公司是以10开头的网络，而vm是192开头，根本就不在一个网段,那么原来是怎么通信的？
后来查了下是和两个虚拟网卡有关vmnet1和vmnet8
一个是设置私有网络（Host Only）时，用来和主机通信的，禁用以后就无法正常使用Host-Only模式了，另一个是设置网络地址翻译（NAT）时，和主机通讯使用的。
而我上次直接禁用了这两个，如果你只是远程vm，只需打开vmnet8.

#####坑七
（自家wifi）
A、vm中新建的centos7无法连接网络（桥接模式，没有复制物理网络连接状态），后来ping宿主ip出现network is unreachable（网络不可达）也没有ifcfg-eth0文件（cent7貌似都没有），直接修改ifcfg-eno167777736文件，改为ONBOOT=yes（开机启动此配置），然后service network start。 ok！搞定
B、如果 NAT也存在上面的问题unreachable，首页查看宿主机器的默认网关，接着将v8的网关改为一致即可（或是.2，.1）。

在公司的时候，宿主机器的ip和mac是绑定的，NAT模式是可以的。(有的时候会有问题)，主要就是这个网关IP的原因。也可以使用桥接，指定宿主机器的物理网卡
ps:桥接相当于你的兄弟机器，大家在一个网段；NAT相当于宿主机器做为路由，虚拟机做为局域网中的一台机器（所以IP为192.168**）。


<img src="http://upload-images.jianshu.io/upload_images/3650492-b77ea2470a92bb8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240"/>
###常识
#####1、下载***.sh文件，并直接运行，会下载相关组件及软件（如google浏览器）
  rpm 文件：rpm -ivh **.rpm
(rpm安装后有些需要yum 真正安装软件及其依赖)

#####2、解压文件
  cp -r 源位置 目标位置
  tar -xvf **.tar.gz  (-v/x:压缩/解压，-v:详细过程，-f:输出/输入文件)

#####3、循环删除此目录及子文件、文件夹：rm -rf ****

4.默认安装路径：usr/java
   环境配置： etc/profile
   通过export只是临时性，需要在profile加上路径，然后source /etc/profile使环境立即生效

#####5、查询安装了哪些jdk
  rpm -qa|grep jdk

#####6、卸载相应jdk版本
rpm -qa | grep java #查询
rpm -e --nodeps **** #卸载相关版本
（找不到openjdksource可以：yum -y remove java **）--貌似没用，先备着(只是删除软件包)

#####7、文件属性
　 r 表示文件可以被读（read）
　　w 表示文件可以被写（write）
　　x 表示文件可以被执行（如果它是程序的话）
　　其中：rwx也可以用数字来代替
　　r  ------------4
　　w ------------2
　　x ------------1
　　- ------------0
-rw-r--r-- (644) 只有所有者才有读和写的权限，组群和其他人只有读的权限
4755：drwsr-xr-s d:文件类型（目录文件） d/rws/r-x/r-x
这个4表示其他用户执行文件时，具有与所有者相当的权限,所以写成rws（原本是rwx）


#####8、wget 下载想要的rpm或是其他文件

#####9、sudoers文件的作用是 控制用户可以执行哪些指令

#####10、开放3306端口（新安装的mysql可能无法远程，如果用户表已经赋权好了，远程连接报10038错误）
firewall-cmd --zone=public --add-port=3306/tcp --permanent #开放3306
firewall-cmd --reload #在不改变状态的条件下重新加载防火墙
上面的指令完全ok，下面的备用
systemctl stop firewalld.service #停止
systemctl disable firewalld.service #禁用
（centos6.5 service iptables stop）
神坑：直接stop防火墙，依旧访问不了，必须要先开发端口

#####11、vi编辑器
gg:第一行
G：最后一行
ctrl+b:上翻一页
ctrl+f:下翻一页 

/* 从光标开始处向文件尾搜索
?* 从光标开始处向文件首搜索
n：在同一方向重复上一次搜索命令(/?的方向决定方向)

shift+v:块选择
yy:块复制
dd:剪切当前行
p:粘贴

#####12、epel-release
安装docker的时候看教程需要安装epel-release包，我记得以前安装过，查询之后才知道这是第三库源，里面有很多程序方便下载而且很多扩展包比官网更新较快（官方以稳定为主）。
yum install docker（centos7下直接安装，不同版本会不同）


#####13、进程操作
ctrl-c 发送 SIGINT 信号给前台进程组中的所有进程。常用于终止正在运行的程序。
ctrl-z 发送 SIGTSTP 信号给前台进程组中的所有进程，常用于挂起一个进程

crtl+z 将进程挂起，然后fg/bg把刚刚挂在的程序放在前台/后台运行

#####14、查看版本
系统版本：cat  /etc/redhat-release
内核版本：uname -r

#####15、查看端口占用
ps -anp |grep 8080

#####16、golang的install是默认在/usr/local/bin/ 如果想自己配置，需要在profile添加指定路径。
export GOROOT=/usr/tangyb/go
export GOPATH=$PATH:$GOROOT/bin

#####17、设置tomcat的https
a）生成文件 jdk1.7.0_79/bin/keytool -genkey -alias polaris -keyalg RSA -keystore ./polaris.keystore 
b）导出证书 jdk1.7.0_79/bin/keytool  -export -alias polaris -keystore ./polaris.keystore -storepass ******* -rfc -file polaris.cer
c) 在server.xml文件的ssl/tls端口 加入keystoreFile="/usr/baxia/users/tangyb/tomcat_https/polaris.keystore" keystorePass="huoying3138266"
****一定要记得在输入name的时候是写入 网站域名，否则会出现无法通过身份验证的红叉
参考文章http://www.cnblogs.com/green-hand/p/6514597.html



#####18、rm命令的修改（改为mv） 及 定时清除文件
rm更改
a、创建回收站，存放删除的文件
b、创建mv脚本文件
c、在bashrc将rm捕获并改为调用mv脚本
d、定时清除回收站的文件

用户级别：
crontab -e 编辑crontab定时文件
crontab -l 查看明细列表（不包括系统级别的）
service crond restart 重启crontab的服务

系统级别：
/etc/crontab ##注意格式，需要添加指定用户

所有的crontab的执行过程 都可以在 /var/log/cron 查看
(如果任务执行了，没有结果，多注意权限、任务本身等等)


profile：登录会读取
bashrc：每次执行bash会读取

/etc/profile与/etc/bashrc的区别  
前一个主要用来设置一些系统变量,比如JAVA_HOME等等,后面一个主要用来保存一些bash的设置.   
/etc/profile:此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行. 
并从/etc/profile.d目录的配置文件中搜集shell的设置.  
/etc/bashrc:为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.  
~/.bash_profile:每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该  
文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件.  
~/.bashrc:该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该  
该文件被读取.  
~/.bash_logout:当每次退出系统(退出bash shell)时,执行该文件.

/etc/下的很多文件都是系统级别的
比如crontab、bashrc、profile（相当于全局）
而这些文件 每个用户也会 单独拥有

#####19、locate
locate速度很快，是因为有一个检索库，所以用之前最好updatedb下

#####20、邮件，广播等
w: 查看在线账户
wall: 群发消息
write: 个人私发
mail： 邮件发送

#####21、VMware Tools安装（转载：http://www.epinv.com/post/5217.html）
虚拟机-安装VMware Tools
mkdir /media/mnt    #新建挂载目录
mount /dev/cdrom    /media/mnt/      #挂载VMware Tools安装盘到/media/mnt/目录
可以移动到安装目录在解压安装
tar zxvf VMwareTools-9.6.2-1688356.tar.gz #解压
./vmware-install.pl  #安装
成功后重启服务器

#####22、系统监控
空间大小：du -sm ./logs/   （s:指定的目录，m：按照MB来显示大小）
日志ERROR高亮：tail -f xxx.log | perl -pe 's/(ERROR)/\e[1;31m$1\e[0m/g'

#####23. 集群网络基本配置（centos 7）
a. 设置静态ip(NAT模式)
修改配置文件 /etc/sysconfig/network-scripts/ifcfg-eno16777736
BOOTPROTO=static ##默认dhcp，改为静态分配
ONBOOT=yes  ##自启动，默认no
IPADDR=192.168.118.129   ##ip地址，其中192.168.118和NAT模式的子网IP相同 
GATEWAY=192.168.118.2   ##网关，和NAT模式设置的网关相同
NETMASK=255.255.255.0   
DNS1=192.168.118.2     ##和网关相同 或是 114.114.114.114（电信的DNS）或是223.5.5.5（阿里），可在NAT模式菜单中修改

service network start 重启ok, ping百度测试

b. 端口开放
firewall-cmd --zone=public --add-port=3306/tcp --permanent #永久开放3306
firewall-cmd --reload #重新加载防火墙规则
systemctl stop firewalld.service #停止
systemctl disable firewalld.service #禁用

#####24.集群资源共享
采用scp传送，之前需要准备ssh免密登录
scp -r 源文件夹  username@ip(或者映射名):目标文件夹
-r：传输文件夹
