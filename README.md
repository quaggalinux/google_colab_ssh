# google_colab_ssh
使用ssh连接google colab不依赖第三方服务中转的配置方法  
  
前阵子有一个项目需要用到google colab的环境，但是只用它的web操作界面的话，总是觉得差了点什么，  
于是研究了google colab自己的介绍，发现它是可以配置成ssh登录的，但是它需要借助第三方的服务进行中转，  
这个第三方中转服务毕竟还是不放心，于是折腾一番，研究出自力更生的方法，完全不依赖第三服务，  
当然这个是有一些前提条件准备的，首先你自家的出口路由器必须有端口映射的功能，然后也必须有DDNS的功能，  
这个DDNS功能可以是路由器具有，也可以是内网的NAS、树莓Pi、OpenWRT、linux真机/虚拟机等等可以做DDNS功能或运行DDNS更新脚本的机器  
  
下面是各位需要准备的自己内网环境：  
你自己DDNS申请的域名假设是home.hopto.org，当然建议大家可以找合适自己的DDNS服务商  
内网连接colab映射端口的机器IP地址假设是10.0.0.8  
自己的出口路由器映射11111端口给10.0.0.8机器  
  
另外还需要在github下载运行一个开源的软件gost  
  
如果10.0.0.8这台机器是windows7或10的，使用下面的连接下载64位版：  
https://github.com/ginuerzh/gost/releases/download/v2.11.1/gost-windows-amd64-2.11.1.zip  
下载后解压到随便一个目录，如c:\gost目录等等，改名字为gost.exe  
然后使用管理员权限打开“命令提示符”窗口，在命令行打入下面命令：  
c:\>c:\gost\gost -L=ssh://:11111  
然后就放着这个运行窗口千万不要关闭！！！  
  
如果10.0.0.8这台机器是linux的，使用下面的连接下载64位版：  
转用root用户，并下载运行  
$su  
#cd /tmp  
#wget https://github.com/ginuerzh/gost/releases/download/v2.11.1/gost-linux-amd64-2.11.1.gz  
#gunzip gost-linux-amd64-2.11.1.gz  
#mv gost-linux-amd64-2.11.1 \usr\bin\gost  
#chmod +x \usr\bin\gost  
#gost -L=ssh://:11111  
然后就放着这个运行窗口千万不要关闭！！！如果这时想做其他操作，可以在终端软件打开另外一个新的ssh会话，  
后面也可以把gost这个运行命令创建成systemctl的服务，这样开机就可以自动监听11111端口了  
  
其他系统如NAS、树莓Pi、OpenWRT等等的，各位自己去上面github连接下载相应的操作系统及CPU架构运行版本  
  
下面是google colab的配置部分，当然你得登录colab了:  
https://colab.research.google.com  
  
登录后 文件->新建笔记本  
  
点击左上角的“代码”创建代码单元格  
  
在新创建的代码单元格里面粘贴下面的代码，下面的代码除了创建root登录密码外，  
还创建了一个一般用户登录名及密码（这个没有需要的话可以不要，删除相应语句即可）：  
  
!apt install ssh openbsd-inetd telnetd htop iftop nload nethogs net-tools mtr-tiny traceroute virt-what lsof nano iproute2 iproute2-doc -y  
!/etc/init.d/openbsd-inetd restart  
!useradd -m -s /bin/bash 创建你的一般登录用户名  
!echo "root:创建你的root登录密码"|chpasswd  
!echo "创建你的一般登录用户名:创建你的一般用户登录密码"|chpasswd  
!echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config  
!echo "PermitRootLogin yes" >> /etc/ssh/sshd_config  
!/etc/init.d/ssh restart  
!wget https://github.com/ginuerzh/gost/releases/download/v2.11.1/gost-linux-amd64-2.11.1.gz  
!gunzip gost-linux-amd64-2.11.1.gz  
!mv gost-linux-amd64-2.11.1 gost  
!chmod +x gost  
!nohup /content/gost -L=rtcp://:20022/127.0.0.1:22 -F=ssh://home.hopto.org:11111 > /dev/null 2>&1 &  
!netstat -tunap  
  
粘贴完成后点击菜单 代码执行程序->全部运行  
  
等代码全部运行完毕，检查最后的输出信息是否有sshd进程正在监听22端口，  
然后gost进程连接到你自己家当前出口公网IP地址的11111端口，  
有的话就表示运行正常  
  
做完上面的步骤就可以在本地打开一个终端软件SSH登录到google colab了，putty、xshell、MobaXterm等等都可以，  
这时的ssh连接信息如下：  
  
ssh服务器IP地址为：10.0.0.8  
连接端口为：20022  
登录用户名：root  
登录密码：“创建你的root登录密码”上面代码里面你自己设的那个  
  
当然也可以用上面代码你创建的一般用户名登录  
  
这个SSH连接端口是没有暴露到公网的，所以相对是安全的，而且gost的连接隧道本身也使用了开源的协议来加密连接  
  
至此各位就可以愉快地使用SSH登录google colab而不需要求助于第三方服务了  
  





