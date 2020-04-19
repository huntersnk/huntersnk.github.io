---
layout: post
title:  "架设 Ghost VPS"
date:   2016-12-11 02:17:42 +0800
---
今年一年没写 blog，有点偷懒。



在去年跟鸟哥还有小蜗老师一块儿玩的播客 ThreeCast 中，有一期是讲 blog 的，当时我刚把自己的 blog 从 Wordpress 迁移到了 Ghost，就在节目中说要写一篇如何架设 Ghost 的文章......转眼快两年过去了，这篇文章一直被我坑到现在。我之前搭建 Ghost 的服务器位于 US，由于众所周知的原因，经常无法访问，能访问时延迟也很大。在据传要[新铺建 HK 至 US 的光缆建成](http://news.xinhuanet.com/tech/2016-10/13/c_1119710556.htm)之前，US 的 VPS 确实都不太适用于架设面向国内访问者的站点，所以，我决定把我的 Ghost blog 从 US 迁移到 HK，顺便升级一下 Ghost 源程序。

废话不多说，开始今天的正题，你会从这篇博文中了解到以下这些内容：



 1. 对 VPS 进行基本配置

 2. 安装 Ghost 所需要的环境

 3. Ghost 安装配置



需求：



 * VPS OS: ubuntu 14.04 x64

 * Nodejs version : 6.9.0

 * Ghost version : 0.11.3

 * 本地 OS:  Linux / macOS / Linux in a virtual machine



> 注意，从2016年11月2号 release 的 0.11.3 版开始，Ghost 正式支持 Node v6 LTS 了

所以这里选用 Nodejs 6.9.0，而不是早期的 v4 系列版本。



阅读指引:<br>

server$: 表示服务器终端

local$: 表示本机终端

冒号后才是需要键入的指令



### VPS基本配置





登录服务器后，首先进行更新<br>

<pre><code>server$: sudo apt-get update && sudo apt-get upgrade</code></pre>



安装基本组件

<pre><code>server$: sudo aptitude install build-essential zip git vim libssl-dev -y

</code></pre>



创建一个新用户

<pre><code>server$: sudo adduser demo</code></pre>



加入 sudo

<pre><code>server$: sudo gpasswd -a demo sudo</code></pre>



在本机（Linux or Mac ）生成一个 keygen

<pre><code>local$: ssh-keygen</code></pre>



一般生成的文件在本机用户账户的 .ssh 目录下

<pre><code>local$: cat ~/.ssh/id_rsa.pub</code></pre>



你会看到类似下面这大一串长度的字符串：

> ssh-rsa ABCDABsadjklasdjklasjdlkasjdklasjkfldjdsjfdsfsdfsdf

sdfsdfdsfdsfdsfdsfsdfdsfdsfdsfdsfdsfdsfdsfdsfdsfdsfd

qweqweqwewqeqweqwewq bla@bla bla



复制它，回到服务器端

默认还是 root 登录，我们切换到新创建的 demo 用户

<pre><code>server$: su - demo

server$: mkdir .ssh

server$: chmod 700 .ssh

server$: vim .ssh/authorized_keys</code></pre>



把刚才复制的那一串公钥字符串粘贴到这里

保存退出<br>

[ESC] ----- :wq ----- [ENTER]



紧接着

<pre><code>server$: chmod 600 .ssh/authorized_keys</code></pre>



回到 root 用户

<pre><code>server$: exit</code></pre>



配置 ssh

<pre><code>server$: vim /etc/ssh/sshd_config</code></pre>



找到 Port 22这一行<br>

改成 Port 1103 或其它，修改 SSH 登录的端口号



找到 PermitRootLogin yes 这一行<br>

改成 PermitRootLogin no<br>

关闭 root 直接登录<br>

保存退出<br>

[ESC] ----- :wq ----- [ENTER]



配置基本的防火墙,开启 SSH 端口、http 、https、SMTP

<pre><code>server$: sudo ufw allow 1103/tcp (如果你修改成别的端口，记得替换这里的数字)

server$: sudo ufw allow 80/tcp

server$: sudo ufw allow 443/tcp

server$: sudo ufw allow 25/tcp</code></pre>

完成规则添加后，可以检查一下是否有问题

<pre><code>server$: sudo ufw show added</code></pre>

确认无误后启用它

<pre><code>server$: sudo ufw enable</code></pre>



配置一下时区及时间

<pre><code>server$: sudo dpkg-reconfigure tzdata</code></pre>

在现实的配置页面中选择你的物理服务器所在的地区/时间



安装 NTP

<pre><code>server$: sudo apt-get install ntp -y</code></pre>



创建一个交换分区<br>

>注意：如果你的服务器是 SSD 硬盘，就不建议创建交换分区

一般交换分区设置为服务器物理内存的2倍



<pre><code>server$: sudo fallocate -l 2G /swapfile

server$: sudo chmod 600 /swapfile

server$: sudo mkswap /swapfile

server$: sudo swapon /swapfile</code></pre>

设置交换分区在每次服务器重启、启动时自动挂载

<pre><code>server$: sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'</code></pre>



重启一下 ssh

<pre><code>server$: service ssh restart</code></pre>

这时最好先不要退出这个登录，在本机新开一个终端，测试是否能够正常登录服务器

<pre><code>local$: ssh - p 1103 demo@xx.xx.xx.xx (你的服务器IP地址)</code></pre>

如果能直接登录，说明刚才设置的密钥及端口都生效了

如果不能登录，在还没退出的终端里及时排查错误，修改完后记得再次重启 ssh



使用 demo 登录后，切换到 root

<pre><code>server$: su - root</code></pre>



继续配置服务器，安装 Fail2Ban

<pre><code>server$: sudo apt-get install fail2ban -y</code></pre>



拷贝配置文件

<pre><code>server$: sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local</code></pre>



编辑

<pre><code>server$: sudo vim /etc/fail2ban/jail.local</code></pre>



[ssh] <br>

把 port  =  ssh 改为 port = 1103



[ssh-ddos]<br>

enabled = true

port = 1103



保存退出<br>

[ESC] ----- :wq ----- [ENTER]



重启 Fail2ban

<pre><code>server$: sudo service fail2ban restart</code></pre>



### 安装 Ghost 需求环境

通过 NVM 安装管理 nodejs

<pre><code>server$: curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash

server$: source ~/.profile

server$: nvm ls-remote</code></pre>

安装6.9.0版本并设置为默认版本

<pre><code>server$: nvm install v6.9.0

server$: nvm use v6.9.0

server$: nvm alias default v6.9.0</code></pre>

安装完毕，这时可以输入 nvm 检查可以使用的命令



### 安装 Ghost

<pre><code>server$: sudo mkdir -p /var/www/

server$: cd /var/www/

server$: sudo wget https://ghost.org/zip/ghost-latest.zip

server$: sudo unzip -d ghost ghost-latest.zip

server$: cd ghost

server$: npm install --production</code></pre>



如果有报错，通过查看 error log 来排除



紧接着配置 Ghost

<pre><code>server$: sudo cp config.example.js config.js

server$: sudo vim config.js</code></pre>

把里面的 url: ''

改成你自己的 URL



使用以下命令创建 /etc/init.d/ghost 文件：

<pre><code>server$:  sudo curl https://raw.githubusercontent.com/TryGhost/Ghost-Config/master/init.d/ghost \

  -o /etc/init.d/ghost



server$: sudo useradd -r ghost -U</code></pre>

确保 Ghost 用户可以访问安装目录：



<pre><code>server$: sudo chown -R ghost:ghost /你的 Ghost 安装目录</code></pre>

使用以下命令给这个初始化脚本加上可执行权限：

<pre><code>server$: sudo chmod 755 /etc/init.d/ghost</code></pre>





使用 vim /etc/init.d/ghost 命令打开文件并检查以下内容：

将 GHOST_ROOT 变量的值更换为你的 Ghost 安装路径

检查 DAEMON 变量的值是否和 which node 的输出值相同

完成后，你可以使用以下命令来控制 ghost 的启动/停止/重启/状态



<pre><code>server$: sudo service ghost start

server$: sudo service ghost stop

server$: sudo service ghost restart

server$: sudo service ghost status</code></pre>



为了让 ghost 能在每次系统启动的时候同时启动，需要将刚刚创建的初始化脚本注册为启动项。

<pre><code>server$: sudo update-rc.d ghost defaults

server$: sudo update-rc.d ghost enable</code></pre>





安装 Nginx

<pre><code>server$: sudo apt-get install nginx -y

server$: sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/ghost.conf

server$: sudo vim /etc/nginx/sites-available/ghost.conf</code></pre>

按照你的需求修改配置，注意，如果你需要像我一样使用SSL

请自行根据 SSL 签发商提供的文件进行配置，把证书密钥等放在 Nginx 推荐放到的位置上



配置完 nginx 的 conf 后，记得把该配置文件链接到 nginx 另外一个目录的配置里

否则 Nginx 配置不会生效。

<pre><code>server$: sudo ln -s /etc/nginx/sites-available/ghost.conf /etc/nginx/sites-enabled/ghost.conf</code></pre>



配置完成后，重启服务器

<pre><code>server$: sudo service nginx restart</code></pre>



在浏览器中尝试访问

http(s)://你的域名/ghost

即可对 ghost 应用进行初始化配置及创建



Enjoy :)



参考：<br>

https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04

https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers

https://gist.github.com/janikvonrotz/8542013

https://www.digitalocean.com/community/tutorials/how-to-create-a-blog-with-ghost-and-nginx-on-ubuntu-14-04

https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04

http://docs.ghostchina.com/installation/deploy/

https://support.comodo.com/index.php?/Knowledgebase/Article/View/801/0/nginx-csr-generation-using-openssl

https://support.comodo.com/index.php?/Knowledgebase/Article/View/1091/0/certificate-installation--nginx





