## P4 Language Specification

[P4 Specs](https://p4.org/p4-spec/docs/P4-16-v1.2.2.html "P4 Specs")是最正宗的理论教材了。从概念到架构到语法，一步一步地介绍P4语言。如果遇到讲解不清楚的专有名词或statement，可以查一查C语言的相关介绍，因为P4有很多地方都借鉴了C，少量借鉴了Java。

学习理论知识的同时还得多练习。虽然Intel P4 Suite不免费，不提供给个人用户使用，但Github上还有开源的 `p4c`编译器。还有一些开源的教程，利用 `bmv2`软件交换机和`mininet`作为数据面，而且Control Plane也都写好了，咱们只需要专注数据面编程的学习就可以。

推荐2个开源教程。一个是P4官方教程`p4lang/tutorials`，一个是瑞士苏黎世联邦理工学院的`nsg-ethz/p4-learning`。

此外还有一个`jafingerhut/p4-guide`，面向专业的开发者，对初学者不太友好；Xilinx的 [P4-NetFPGA教程](https://github.com/NetFPGA/P4-NetFPGA-public/wiki/Getting-Started "Xilinx P4-NetFPGA教程")不免费，只能看到Slides，看不到code。如果你是某个大学的学生或者老师，倒是可以注册申请一个免费的License访问Github project，获取部分的内容。

## p4lang/tutorials

[p4lang/tutorials](https://github.com/p4lang/tutorials "p4lang/tutorials")提供了`Vagrant`脚本来安装虚拟机和全套的实验环境，但是因为虚拟机无法直接科学shang网，如果直接用官方脚本来安装的话，不会成功的。

SDNLAB用户 @zenbox 制作了一个VM，可以拿来直接用。在[这里](https://github.com/zenhox/p4-quick "p4lang/tutorials实验环境VM，ubuntu 16.04")下载。

这个VM是3年前制作的，基于ubuntu 16.04。如果想用最新的基于ubuntu 20.04的学习环境，可以用我修改后的`Vagrant`和`bash`脚本，替换`tutorials/vm-ubuntu-20.04/`里的同名文件之后，再执行`vagrant up`即可。[传送门：](https://github.com/chenghit/p4learning-setup/tree/main/p4lang-tutorials "p4lang/tutorials实验环境不翻墙安装脚本，基于ubuntu 20.04")

https://github.com/chenghit/p4learning-setup/tree/main/p4lang-tutorials

改动有这么几处：

#### 1. Vagrantfile增加private_network网卡配置，方便SSH操作

```diff
--- /Users/chengc2/p4-learning-env-setup/p4lang-tutorials/Vagrantfile	Sun Jan 30 12:42:59 2022
+++ /Users/chengc2/p4-learning-env-setup/p4lang-tutorials/Vagrantfile.backup.rb	Sun Jan 30 13:35:35 2022
@@ -14,7 +14,6 @@
     dev.vm.provider "virtualbox" do |v|
       v.name = "P4 Tutorial Development" + Time.now.strftime(" %Y-%m-%d")
     end
-    dev.vm.network "private_network", ip: "192.68.56.21"
     dev.vm.provision "file", source: "py3localpath.py", destination: "/home/vagrant/py3localpath.py"
     dev.vm.provision "shell", inline: "chmod 755 /home/vagrant/py3localpath.py"
     dev.vm.provision "file", source: "patches/disable-Wno-error-and-other-small-changes.diff", destination: "/home/vagrant/patches/disable-Wno-error-and-other-small-changes.diff"
@@ -32,7 +31,6 @@
     release.vm.provider "virtualbox" do |v|
       v.name = "P4 Tutorial Release" + Time.now.strftime(" %Y-%m-%d")
     end
-    release.vm.network "private_network", ip: "192.68.56.22"
     release.vm.provision "shell", path: "root-release-bootstrap.sh"
     release.vm.provision "shell", path: "root-common-bootstrap.sh"
     release.vm.provision "shell", privileged: false, path: "user-common-bootstrap.sh"
@@ -54,5 +52,4 @@
     vb.customize ["modifyvm", :id, "--vram", "32"]
   end
 
-end
-+end
\ No newline at end of file
```

**注意：** 从macOS VirtualBox 6.1.28版本开始，仅允许为`host-only`网卡配置`192.68.56.0/21`范围内的IP地址。如果要配置其他范围的IP地址，则要新建一个`/etc/vbox/networks.conf`文件，把允许的subnet写进去，并且不能有空行，否则会触发另外一个`VBoxNetAdpCtl`相关的bug。下面是一个例子：

```java
* 10.0.0.0/8 192.168.0.0/16
* 2001::/64
```

#### 2. 将安装源改为阿里云

为`root-dev-bootstrap.sh`和`root-release-bootstrap.sh`两个文件增加以下内容，把安装源改为阿里云：

```bash
sudo mv /etc/apt/sources.list /etc/apt/sources.list.backup
sudo tee -a /etc/apt/sources.list > /dev/null <<EOT
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
EOT
sudo apt-get update
```

#### 3. 更改git clone的方式

将所有的`git clone https://`改为`git clone git://`，要不然安装进程会卡死在clone的步骤。说到这里，建议大家不要尝试启动`dev`虚机。因为有很多第三方的开发套件是采用`git clone https://`方式安装的，如果前面没有能处理墙的路由器，安装进程会卡死在那里。

#### 4. 为apt-get增加参数，规避Undetermined Error

因为Debian/Ubuntu有个bug，可能会导致`apt-get`多个文件的时候出现`Undetermined Error`。为了规避这个bug，给`root-dev-bootstrap.sh`和`root-release-bootstrap.sh`两个文件的`apt-get`增加`-o Acquire::http::Pipeline-Depth="0"`参数。


## nsg-ethz/p4-learning

[nsg-ethz/p4-learning](https://github.com/nsg-ethz/p4-learning "nsg-ethz/p4-learning")是正了八经儿持续14周的大学课程，Slides教材更新到2020年，内容丰富。它的exercise和demo和`p4lang/tutorials`有一些区别，互为补充。下面是到目前为止两个教程的Exercises，大家可以比较一下。

| p4lang/tutorials exercises | nsg-ethz/p4-learning exercises  |
| :------------------------: | :-----------------------------: |
|           basic            |            Reflector            |
|        basic_tunnel        |            Repeater             |
|            calc            |       L2_Basic_forwarding       |
|            ecn             |           L2_Flooding           |
|          firewall          |           L2_Learning           |
|        link_monitor        |              MPLS               |
|        load_balance        |              RSVP               |
|            mri             |              ECMP               |
|         multicast          |        Flowlet_Switching        |
|         p4runtime          |      Heavy_Hitter_Detector      |
|            qos             |        Count-Min-Sketch         |
|       source_routing       |         Simple_Routing          |
|                            |          Traceroutable          |
|                            | Congestion_Aware_Load_Balancing |
|                            |      Packet-Loss-Detection      |
|                            |          Fast-Reroute           |

这两个教程都不仅可以用来学习，也适合拿来做开发的参考。羡慕他们的Lab，8台Wedge100BF-32X

![图片来源: github.com/nsg-ethz/p4-learning](https://tva1.sinaimg.cn/large/008i3skNgy1gz2ncwy0x2j318l0u0n43.jpg)

这套教程的实验环境所采用的组件和`p4lang/tutorials`基本相同，主要的区别在于`p4lang/tutorials`的实验拓扑和控制面是相对固定的，而`nsg-ethz/p4-learning`提供了一个开发套件`P4-Utils`，其控制面和拓扑更加灵活，还可以自定义拓扑，自己设计一个exercise。

在学习的时候要注意对照项目WiKi。比如我一直搞不明白为什么在定义`portId`类型的时候要占位9个bits？

```c
typedef bit<9> egressSpec_t;
```

后来看了项目WiKi才知道，BMv2 Simple Switch Architecture Model预定义的standard metadata fields就规定了端口号的类型是`bit<9>`。因为要从standard metadata取值，或者赋值给standard metadata，所以在为ingress port和egress port声明变量类型的时候，也必须用`bit<9>`。等真正做项目的时候，端口类型的bit宽度则取决于具体的`target`（设备/芯片）。不同的`target`有不同的Architecture Model，metadata/SDK/`extern` API也不一样，落到P4代码自然也会存在一些差异。

![图片来源: github.com/nsg-ethz/p4-learning](https://tva1.sinaimg.cn/large/008i3skNgy1gz3yd2a7ouj30rb0d1my6.jpg)

`P4-Utils`已经不再提供官方预配的VirtualBox VM镜像，只提供QEMU VM镜像。下载地址和详情请见[安装指南](https://nsg-ethz.github.io/p4-utils/installation.html#use-our-preconfigured-vm "P4-Utils安装使用指南")。

知乎用户 @YAOJ 做了一个`OVA`，[这里下载](https://drive.google.com/u/0/uc?id=1tubqk0PGIbX759tIzJGXqex08igFfzpD&export=download "nsg-ethz/p4-learning实验环境VM，2019年版本")。用VirtualBox导入OVA运行VM之后，记得在Power Manager里面关闭`Display power management`和`Light Locker`，要不然VM容易进入黑屏状态，只能重启VM才能恢复。

`P4-Utils`现已迁移到新版本，基于Python 3，API和Python库也有一些变化。@YAOJ 制作的VM还都是2019年9月的文件，基于Python 2和老版的Python库，exersice也少了一些。所以最好还是找一个可以访问真正的Internet的环境，用脚本安装；要么就从官方安装指南下载最新版本QEMU格式的image，然后安装`qemu-img`，把它转换成`vmdk`、`vdi`或者`vpc`格式。macOS的操作步骤如下：

```python
brew install qemu      # 安装qemu-img
cd ~/Downloads/        # 进入解压缩.tar.gz文件之后，存放qcow2文件的目录
qemu-img convert -f qcow2 -O vdi p4-utils-vm.qcow2 p4-utils-vm.vdi    # 把qcow2镜像转换成VirtualBox VDI格式
```

转换之后，用VirtualBox新建一个64位Ubuntu虚拟机，导入`p4-utils-vm.vdi`作为硬盘文件，用户名/密码：`p4/p4`。

如果虚拟机的`Grub`图形界面无法工作，则需要设置安装源，然后安装`ubuntu-desktop`或者其他的GUI程序，再安装VirtualBox Guest Additions。这个VM的OS版本是Ubuntu 18.04，Codename为`Bionic`。在安装GUI程序的时候，别忘了给`apt-get`命令添加`-o Acquire::http::Pipeline-Depth="0"`参数。

当我们使用`mininet`做练习时，`xterm`需要在图形界面运行，所以GUI程序至关重要。

## 中文《P4学习笔记》

我的校友，C记前同事 @YAOJ （我在强行往自己脸上贴金）在知乎写了一个系列的《[P4学习笔记](https://www.zhihu.com/column/c_1336207793033015296 "《P4学习笔记》 by 知乎@YAOJ")》。这份笔记基于`nsg-ethz/p4-learning`，写得很系统，其中第6篇笔记介绍了使用`P4-Utils`自行设计exercise的方法，是非常优秀的中文入门教材。

## 目前为止的一点心得

宇宙的尽头是考公，P4的尽头是packet format、table、register、counter、meter、pipeline。以往学网络，学的都是BGP等Control Plane的协议；现在学P4，是在Date Plane上编程，要深入理解Pipeline。虽然不是所有的芯片都支持P4，但学习P4对理解ASIC、FPGA、DPU等芯片，以及反过来理解Control Plane协议都会有些帮助。
