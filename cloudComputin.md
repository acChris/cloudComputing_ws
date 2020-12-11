## 背景

Linux一开始出现是在学OS这门课的时候，当时学习热情不高，对于作业的完成度仅仅到能重现老师所给的指导文档。自己却是没真正去学习那个重点，也缺乏OS相关书籍的积累。

现在的一门云计算期末考核，一部分成绩是技术实操，所以该填的坑还是要填的。。。



## 环境配置

`VMware Workstation/Oracle VM VirtualBox`

`CentOS7`



## 云计算实验：Zookeeper+Hadoop+Hbase整合平台搭建（完全分布模式）

### 配置五台虚拟机



#### 1. 名字连续，

#### 2. 虚拟机配置统一

* **内存：**`2G`（不够的话1G也可）

* **操作系统：**`Red Hat（64-bit）      `

* **光驱：**`CentOs-7-x86_64-Minimal-1908.iso`(或其他版本也可)

* **硬盘容量：**`20GB（8GB也可）`

如果不知道虚拟机的具体安装步骤，请看参考教程：https://gitee.com/aroming/course_os/blob/master/Experiments/Expt_Guides/OS-E01_Guide/OS-E01_guide.md#%E5%AE%9E%E9%AA%8C%E5%86%85%E5%AE%B9%E6%8C%87%E5%BC%95

#### 3. 创建新用户admin

```
useradd admin
passwd admin
```

#### 4. 配置虚拟机网卡：

**1.查看网卡状态：**

```
ifconfig

cd /etc/sysconfig/network-scripts,cat ifcfg-enp0s3(网卡根据自己的,如enp0s3)

vi ifcfg-ens33
```


**2.修改文件内容为如下（有则修改，无则添加）：**

```
BOOTPROTO=static  
NM_CONTROLLED=no
ONBOOT=yes
IPADDR=192.168.10.111（往后是112,113...）
NATMASK=255.255.255.0 
GATEWAY=192.168.10.1
```

**保存退出，Esc，:wq**

**3.重启网卡服务，并测试能否ping通：**
```
service network restart

ping www.baidu.com
```

可以ping通代表网卡配置完成。

<font size="3" color=red>另：如果虚拟机要连接其他主机，需要修改网卡为桥接模式</font>

**4. 配置主机名**

```
cd /etc/

vi hostname

把原来的一行直接删掉，
在末尾添加修改后的主机名Cluster-01(根据所在主机配置)
```


**5. 配置防火墙**

```
firewall-cmd --zone=public --add-port=端口号/tcp --permanent

firewall-cmd --reload
```

关闭防火墙（了解即可，实际操作请勿关闭防火墙）：

```
systemctl　stop　firewalld.service  #关闭防火墙服务

systemctl　disable　firewalld.service  #禁止防火墙服务的开机自动启动

systemctl　status　firewalld.service  #验证是否关闭
```

Zookeeper的常用端口：2181、2888、3888。
Hadoop的常用端口：8019、8020、8030、8031、8032、8033、8040、8041、8042、8088、8480、8485、9000、10020、19888、50010、50020、50070、50075、50470、50475。
HBase的常用端口：2181、2888、3888、60000、60010、60020、60030。HBase的常用端口和Zookeeper有重复是因为HBase自带Zookeeper组件，使用独立Zookeeper时这些端口不会被启用，也就不会造成端口冲突。
Hive的常用端口：9083、10000。
MySQL Cluster的常用端口：1186、2202、3306。

#### 4. 免密码登录配置(全程admin)：

**1. 新建用于集群的admin用户，每台主机都建一个**

```
useradd admin
passwd admin 
```

**2. 免密码设置**

**生成当前登录用户的公钥和私钥文件**

在admin@Cluster-01中:

```
$ ssh-keygen -t rsa
```

**查看是否已有ssh（确定已有可跳过）**
```
$ l.

$ cd .ssh

$ ls
```

**将公钥文件拷贝给需要进行免密码登录的目标主机和目标用户**

（前提是已把hosts发给目标主机）`#scp -r /etc/hosts root@192.168.10.112:/etc`

**$ ssh-copy-id -i ~/.ssh/id_rsa.pub admin@Cluster-02**

**3. 验证免密码登录：**

在root@Cluster-01中:

**ssh admin@Cluster-02**

如果没有提示让你输入Cluster-02的
admin用户的密码，则说明配置正确。



### 相关操作

#### 1. 切换当前用户(假设当前用户为root)：

`su admin`

#### 2. 挂载光盘：

```
mkdir　~/cdrom

sudo　mount　-r　/dev/cdrom　~/cdrom
```

**报错：**`no medium found on /dev/sr0`

**解决方法：**在linux最下面一栏的光驱右击一下，选择已有的`centos`即可。如图左二：
![](https://img2020.cnblogs.com/blog/2191525/202012/2191525-20201204235622741-460077983.png)

#### 3. 卸载光盘：

```
sudo　umount　-v　/dev/cdrom
或
sudo　umount　-v　~/cdrom
```

#### 4. Linux系统中远程传输文件

scp  -r   /home/admin/要传输的文件  root@IP地址/home/

![](https://img2020.cnblogs.com/blog/2191525/202012/2191525-20201205001912122-2135777898.png)




## FAQ（常见问题）

### 进入linux系统后，想移除到win界面移不出来：
      
      很简单，直接shift+ctrl即可。

### 如何把已创建的文件复制到另一路径下：

      sudo cp test.txt dir/test.txt

### 输入ifconfig无反应：
    
      1. 先确认一下是否缺少了ifconfig
            cd /sbin 
            ls

      2.如果确实，安装（sudo只有root可以运行）：
            (sudo) yum install net-tools 
   
      3.如果没网络下载不了，检查网卡是否配置为dhcp，最好是dhcp（linux自动配置），然后重启一下即可

     

### linux中查看IP地址：

```
ip addr
或
vi etc/config/network-scripts/ifcfg-enp0s3
```
#### ping www.baidu.com失败，linux上不了网：

[root@localhost ~]# ping www.baidu.com
ping: www.baidu.com: Name or service not known

**检查网卡设置是否正确:**

        1.可以查看网卡是否启动、IP地址是否存在：ip addr
        如果有enp0s3和inet 地址...代表网卡配置成功，可
        service network start

        2.改变网卡为桥接模式

#### 在关闭防火墙到时候，出现：

`
Redirecting to /bin/systemctl stop  iptables.service
Failed to stop iptables.service: Unit iptables.service not loaded.`

解决方法：

    yum install iptables-services

#### 清除秘钥方式

    ssh-keygen -R 接目标地址  即可清除秘钥文件

![](https://img2020.cnblogs.com/blog/2191525/202012/2191525-20201208222158214-1613692676.png)

<!-- #### sudo执行命令时提示找不到该命令

1、切换到root用户， 以root用户身份来编辑文件/etc/sudoers：

vim /etc/sudoers
1
找到Defaults env_reset, 将其改为Defaults !env_reset，
然后wq!强制保存退出。
2、 切换回普通用户如用户名为xx, 编辑/etc目录下的配置文件bashrc：

vim bashrc
1
在文件内最后追加：

alias sudo='sudo env PATH=$PATH'
1
是修改后的配置文件生效：

source bashrc

参考自https://blog.csdn.net/panchang199266/article/details/81543335


可以使用 ssh-keygen -R 接目标地址  即可清除秘钥文件

[参考资料](https://blog.csdn.net/zhou_438/article/details/86761398)
      -->