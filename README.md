# 怎样开始学习P4

## P4 Language Specification

这是最正宗的理论教材了。从概念到架构到语法，一步一步地介绍P4语言。

https://p4.org/specs/

学习理论知识的同时还得多练习。虽然Barefoot的P4编译器不免费，不提供给个人用户使用，但Github上还有开源的 `p4c`编译器可以使用。还有一些开源的教程，利用 `bmv2`软件交换机和`mininet`作为数据面，而且Control Plane也都写好了，咱们只需要专注数据面编程的学习就可以。

有2个最受欢迎的开源教程，一个是P4官方教程`p4lang/tutorials`，一个是瑞士苏黎世联邦理工学院的`nsg-ethz/p4-learning`。此外还有一个`jafingerhut/p4-guide`，面向专业的开发者，对初学者不太友好。

## p4lang/tutorials

https://github.com/p4lang/tutorials

这个教程提供了`Vagrant`脚本来安装虚拟机和全套的实验环境，但是因为虚拟机无法直接科学shang网，所以直接用官方脚本来安装的话，不会成功的。

SDNLAB用户 @zenbox 制作了一个VM，可以拿来直接用：

https://github.com/zenhox/p4-quick

这个VM是3年前制作的，基于ubuntu 16.04。如果想用最新的基于ubuntu 20.04的学习环境，可以用我修改后的`Vagrant`和`bash`脚本，替换`tutorials/vm-ubuntu-20.04/`里的同名文件之后，再执行`vagrant up`即可。传送门：

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

**注意：**从macOS VirtualBox 6.1.28版本开始，仅允许为`host-only`网卡配置`192.68.56.0/21`范围内的IP地址。如果要配置其他范围的IP地址，则要新建一个`/etc/vbox/networks.conf`文件，把允许的subnet写进去，并且不能有空行，否则会触发另外一个`VBoxNetAdpCtl`相关的bug。下面是一个例子：

```java
* 10.0.0.0/8 192.168.0.0/16
* 2001::/64
```

#### 2. 将安装源改为阿里云

为`root-dev-bootstrap.sh`和`root-release-bootstrap.sh`两个文件增加以下内容，把安装源改为阿里云：

```java
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

这个教程比p4lang的教程更好，它是正了八经儿持续14周的大学课程。exercise和demo非常丰富，Slides教材也更新，内容更多。不仅可以用来学习，也适合拿来做开发的参考。

羡慕他们的Lab，8台Wedge100BF-32X，刚出来就用上了！

![图片来源: github.com/nsg-ethz/p4-learning](https://tva1.sinaimg.cn/large/008i3skNgy1gyvxffr46pj318l0u0n43.jpg)

这套教程的实验环境所采用的组件和`p4lang/tutorials`基本相同，区别在于`p4lang/tutorials`的实验拓扑是固定的，而`nsg-ethz/p4-learning`提供了一套开发套件`P4-Utils`，通过修改JSON文件为每个实验生成不同的拓扑，操作步骤也比`p4lang/tutorials`简单一些。

`P4-Utils`已经不再提供官方预配VirtualBox VM，只提供QEMU VM。

https://nsg-ethz.github.io/p4-utils/installation.html#use-our-preconfigured-vm

手工安装脚本没有修改的空间，不过可以使用 @YAOJ 做的一个`OVA`，虽然也需要科学的方法，但比虚机科学要简单多了。5.7GB。

https://drive.google.com/u/0/uc?id=1tubqk0PGIbX759tIzJGXqex08igFfzpD&export=download

`P4-Utils`现已迁移到新版本，基于Python 3，API也有一些变化。@YAOJ 制作的VM还都是2019年9月的文件，基于Python 2，exersice也少了一点点。暂时我还没敢`git pull`，打算以后找个可以访问Internet的环境再试着做一个新的VM。

## 中文《P4学习笔记》

我的校友，C记前同事 @YAOJ （其实我和他完全没有时空交集，强行往脸上贴金了）在知乎写了一个系列的《P4学习笔记》。这份笔记基于`nsg-ethz/p4-learning`,写得很系统，是非常优秀的中文入门教材。

https://www.zhihu.com/column/c_1336207793033015296

## 目前为止的一点心得

宇宙的尽头是考公，P4的尽头是packet format，table, queue, pipeline。以往学网络，学的都是BGP等Control Plane的协议；现在学P4，是在Date Plane上编程，要深入理解Pipeline。所以，虽然不是所有的芯片都支持P4，但学习P4对理解ASIC、FPGA、DPU等芯片，以及反过来理解Control Plane协议都会有些帮助。













