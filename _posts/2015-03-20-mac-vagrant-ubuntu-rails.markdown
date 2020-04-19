---
layout: post
title:  "Mac-Vagrant-Ubuntu-Rails开发环境V0.1"
date:   2015-03-20 02:17:42 +0800
---
#####Notes:

本文会随着我持续使用Vagrant而保持更新

（并没有空更新了@2020）

如果发现有错误、疑议或者描述不清楚的地方还请多多指点和提出宝贵的意见，非常感谢

由于众所周知的原因，请在根据本教程按部就班的操作之前先自行配置好可用的梯子网络，能花钱的解决的，就不要浪费时间:)

随后会更新linux版的vagrant配置以及通过vagrant本地虚拟出nginx服务器环境



#####Log:

20150131 V0.1 实现基于Mac OS 10.10，通过vagrant+ubuntu14.04构建出一个rails demo的运行环境

20150320 V0.1 把本文从github pages上的旧blog搬到这里，没有内容update



#####A.介绍

什么是vagrant，为什么要用它？



vagrant是一个基于虚拟机并管理它的工具，跟Docker相比有什么不同可以看看[这篇文章](http://www.cnblogs.com/vikings-blog/p/3973265.html)很专业很详细的对比，以我个人的一点目前的理解，就是直接使用传统的虚拟机来构建环境有很多不方便的地方，而vagrant的出现正是来弥补直接使用虚拟机构建环境的缺点的。 很多时候我们开发一些东西，需要模拟出不同的运行环境，这个时候有以下几种选择：



* **本地配置环境** 这个是早期最没效率的办法，缺点很多，不方便管理；执行环境单一；容易把系统本身的环境变量配出问题等等



* **虚拟机** 这个方法的缺点在于需要占用比较大的硬盘空间，并且管理依然不太方便，虽然执行环境可以配置出多套来，但是每种环境就意味着要多添加出一个虚拟机，虽然这是个硬盘越来越便宜的时代，但是依然还是下策。



* **vagrant** (based on 虚拟机) 通过vagrant，可以只抽取虚拟机中必要的组件执行从而快速构建出不同的虚拟机，甚至可以把环境写入配置文件来发给其它人进行镜像式的环境复制以及部署。默认是不启用GUI图形界面的，直接通过终端ssh登入虚拟机，配置好环境并设定好虚拟机与宿主机的目录映射与网络映射以后，就可以只利用该虚拟机跑环境，利用宿主机的编辑器、版本管理、浏览器等进行方便的编辑与访问了。



#####B.基本运行环境

截至目前笔者只在Mac OS上进行了安装验证，基于linux宿主机的验证预计在未来几个月内进行实际配置，会及时更新到这里



宿主机系统: Mac OS X Yosemite 10.10.2

虚拟机：VirtualBox 4.3.20

Vagrant 1.7.2

首先需要安装VirtualBox，[下载页面](https://www.virtualbox.org/wiki/Downloads)



接下来是安装Vagrant，[下载页面](http://www.vagrantup.com/downloads)



安装过程与其它Mac软件安装方法无异，不需要进行额外的配置 全部安装好后，打开终端，输入



    $ vagrant -v



看到版本号输出说明已经成功安装



#####C.虚拟机配置

vagrant配置环境不同于传统的虚拟机需要下载并载入庞大的镜像进行安装 它是通过载入”Box” 这样的概念来迅速构建，Box相当于提取了运行应用所必要的运行库等的精简版的镜像，并且可以自行定制Box 可以在[这里](https://atlas.hashicorp.com/boxes/search) 查看可用的Box，注意区分有些是官方提供基础Box，而有些是网络上个人提供的Box，当然也可以自行注册账户并提交自己构建的Box，这个暂不列入讨论范围。 我们需要构建的环境是ubuntu14.04，所以可以看到列表中排在[第一位](https://atlas.hashicorp.com/ubuntu/boxes/trusty64)的就是14.04



接下来找一个工作目录，假设说我们需要一个项目目录名为railsdev1 那么首先创建它并进入



    $ mkdir railsdev1



    $ cd railsdev1



然后基于刚才查看的Box来构建，输入



    $ vagrant init ubuntu/trusty64



第一次使用这个Box需要下载，根据网速的不同可能需要等待几分钟到几十分钟不等，先去喝杯咖啡: ) 成功后，会提示说Vagrantfile已经生成到了这个目录，可以根据注释来进行定制和修改 此时已经可以登录虚拟机环境了，但是我们可以先对Vagrantfile进行配置再进入 用你喜欢的编辑器(Vim/sublime text)打开目录内生成的这个Vagrantfile文件，这里给出我的一份参考，你可以根据我的以及阅读文件中的注释来进行自定义配置 注：为了篇幅整洁删除了源文件中的所有注释，只留下了配置行



    Vagrantfile

    Vagrant.configure(2) do |config|

    config.vm.box = "ubuntu/trusty64"

    config.vm.network "forwarded_port", guest: 3000, host: 3000

    config.vm.synced_folder "/Users/SNK/Code/railsdev1", "/home/vagrant"

    config.vm.provider "virtualbox" do |vb|

    vb.memory = "2048"

    end

    end



以上配置分别是:设定ubuntu 14.04的box为基础环境

设置虚拟机的端口3000映射到宿主机的3000端口

设定映射目录，前后两个双引号代表的分别是我的宿主机目录与虚拟机目录 你需要根据情况来修改前面双引号内的地址

设定虚拟机内存为2GB

配置好并保存后，就可以执行vagrant up来启动了



    $ vagrant up



启动好后，虚拟机就已经跑在后台了，我们需要输入



    $ vagrant ssh



来登录它



注：如果vagrant up之后，又修改了Vagrantfile文件，需要输入



    $ vagrant reload



来重新载入配置，因为执行了vagrant up后，不管你有没有ssh登录进去，虚拟机一直跑在后台，是不会自动重新载入配置文件的，需要手动reload才会基于修改后的配置文件重新启动。



#####D. 虚拟机的Rails执行环境配置

登录进去之后，可以看到命令行是



    vagrant@vagrant-ubuntu-trusty-64:$



vagrant默认是提供用户名和密码都为vagrant的账户来登录



首先更新源



    $ sudo apt-get update



然后安装一些基本的dependencies



    $ sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties -y



然后安装Ruby 可以选择通过rbenv或者rvm或者source的方式， 这里我选用rvm方式安装： 还是先安装基本的dependencies



    $ sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev -y



获取rvm



    $ curl -L https://get.rvm.io | bash -s stable



注意这里可能会出现下图的情况，RVM需要校验一个公钥的签名但是并没有找到 默认给提供了两种解决方法，任意选择一种执行即可

![2015032001](https://dn-huntersnk.qbox.me/blog/2015/2015032001.png)



    $ command curl -sSL https://rvm.io/mpapis.asc | gpg --import -



成功后重新执行



    $ curl -L https://get.rvm.io | bash -s stable



成功后，source它并配置生效环境变量



    $ source ~/.rvm/scripts/rvm

    $ echo "source ~/.rvm/scripts/rvm" &gt;&gt; ~/.bashrc



接下来通过rvm安装Ruby 这里笔者选用的是2.1.5版本的ruby



    $ rvm install 2.1.5



![2015032002](https://dn-huntersnk.qbox.me/blog/2015/2015032002.png)





没有二进制安装包的话可能会需要久一点的时间来自动下载源码和编译 装好后设置成默认



    $ rvm use 2.1.5 --default



    $ echo "gem: --no-ri --no-rdoc" &gt; ~/.gemrc



如果是服务器端或者是最小化运行端，可以配置上面这句来不安装ruby的文档，节省时间和空间。



![2015032003](https://dn-huntersnk.qbox.me/blog/2015/2015032003.png)



接下来安装rails 4.2.0.rc1 注：如果是已有项目，把源码目录移动到挂载的目录内，直接进入执行bundle install就会自动安装所有gem，不需要这个步骤单独安装rails



    $ gem install rails -v 4.2.0.rc1



会自动安装一些必要的gem



接下生成一个基于该版本的demo项目 根据网络等因素可能要多等一会儿



    $ rails _4.2.0.rc1_ new demo



执行完成后，进入该demo目录



![2015032004](https://dn-huntersnk.qbox.me/blog/2015/2015032004.png)



    $ cd demo



尝试启动rails服务器会发现Could not find a js runtime错误 安装nodejs即可解决



![2015032005](https://dn-huntersnk.qbox.me/blog/2015/2015032005.png)



    $ sudo apt-get install nodejs -y



#####E.运行Rails demo

安装完成后尝试启动rails服务器



    $ rails s -b 0.0.0.0



之前我们在Vagrantfile中配置的把虚拟机中的3000端口映射到了本地的3000端口 此时在本地浏览器打开localhost:3000看看



it works!



![2015032006](https://dn-huntersnk.qbox.me/blog/2015/2015032006.png)



启动rails服务时我绑定了启动地址 -b 0.0.0.0，因为目前发现在我本地的环境里，不绑定的话有的项目可以成功转发，而有的项目却不行 保险起见先默认都这样执行以确保本地能访问到，我还需要继续研究这个vagrant网络映射到本地的配置关系，会持续更新到这里。