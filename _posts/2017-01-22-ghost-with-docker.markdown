---
layout: post
title:  "Deploying Ghost With Docker-Compose"
date:   2017-01-22 02:17:42 +0800
---
如题，通过 docker-compose 部署 nginx 和 ghost,（同时启用 SSL）, 主要内容如下



1. 基于 ubuntu 14.04 安装 Docker 和 docker-compose

2. 配置（Dockerfile,docker-compose,Nginx 以及 Ghost）

3. 操作及部署



##bla bla bla 的引子



了解到 Docker 这门技术其实已经快两年了，早在2015年鸟哥就推荐过给我，当时我还找来一本[Docker从入门到实践]读了一遍，读完之后并没有用到任何实际的项目中。我本地的开发环境用了 [Vagrant](https://huntersnk.com/20150320/)，好处是在家里和公司电脑的不同 OS 上同步开发环境比较 smooth，但缺点是，实际上线部署过的一些小项目，都还是我很土鳖的在手动配置环境，因为 Vagrant 并不适用于生产环境，我一直没有去着手解决这个问题。<br>

前几天有天早上起床，收到监控邮件提醒说我的 blog 502了，尝试访问一下发现 nginx 没啥问题，只是运行的 Ghost 进程不知什么原因停了，最快的恢复方案是登陆服务器 restart ghost service ，具体原因可以后续再排查，但当时我在床上，手边只有手机，不想下床取电脑来操作，就突发奇想：要是通过 hubot 聊天机器人控制我的服务器就方便了，不用下床拿电脑，只需在手机的聊天工具里给 bot 发一条 restart ghost service 就搞定了。于是就起床吃个早餐喝杯咖啡开始折腾（懒惰果然是提高生产力的最大动机），准备折腾时发现 ghost 有了新版本可以更新，隐约回想起当初了解到到一些 Docker 数据和程序分离的理念，更新会方便很多，并且如果新版本出现问题，可以迅速无痛迁移回旧版程序。于是反正也要折腾，我就决定干脆就改用 Docker 来部署。期间也踩了一点点坑，甚至遇到了一个 Ghost 官方 image 的 bug ，把部署过程记录下来，供有同样部署需求的朋友们参考，另外，我后续计划会再更新一篇把本地 Rails 开发环境也更换为 Docker 的文章，好处以后再说，作为业余开发者，简而言之就是觉得用它很爽。



## 准备工作



1. 一个 64bit 的 ubuntu 服务器(16.10/16.04/14.04)

2. 登陆服务器进行基本配置（自行 Goolgle 现成脚本或参考我之前写的[架设 Ghost VPS](https://huntersnk.com/20161211/) 的第一部分 VPS 基本配置）

3. 准备自己的域名

4. 准备 SSL 证书（可选）

5. 把域名配置指向服务器的 ip



## 安装 Docker 和 docker-compose



这里 Docker 主要是指 docker-engine,是 Docker 运行的基本环境，怎么安装其实官网写的很详细，英文顺畅的朋友可以通过这个[链接](https://docs.docker.com/engine/installation/linux/ubuntu/) 去详细了解每一步为什么这样做，在这里我会列举全部的指令如何做，但是只做简单说明，不详细介绍为什么这样做。

前面的准备工作做好，尤其是对服务器进行过基本配置后，开始安装 Docker 环境



`$ sudo apt-get update && sudo apt-get upgrade`<br>

先进行基本更新

<pre><code>$ sudo apt-get install curl \

linux-image-extra-$(uname -r) \

linux-image-extra-virtual

</pre></code>

安装一些 docker 需要及官方推荐的包

<pre><code>$ sudo apt-get install apt-transport-https

                       ca-certificates</pre></code>

设置 apt 可以通过 https 来安装



`$ curl -fsSL https://yum.dockerproject.org/gpg | sudo apt-key add -`<br>

增加 docker 官方的 GPG 密钥



`$ apt-key fingerprint 58118E89F3A912897C070ADBF76221572C52609D`<br>

验证特征ID 是否为这一串官方给出的id



注意，接下来这点==官网没有写==，但大部分情况下都需要自行安装

如果是 ubuntu 14.04 or 16.xx 的服务器<br>

`$ sudo apt-get install software-properties-common -y`<br>



随后才能把官方地址添加到源里

<pre><code>$ sudo add-apt-repository \

       "deb https://apt.dockerproject.org/repo/ \

       ubuntu-$(lsb_release -cs) \

       main"

</pre></code>



`$ sudo apt-get update`<br>

再更新一下 apt package



`$ sudo apt-get -y install docker-engine`<br>

注意，这里默认是安装最新版的 Docker，如果是部署自己开发的应用，要注意指定跟自己本地开发环境相对应的 Docker 的版本号。



安装完成后，运行一个 hello-world 试试看<br>

`$ sudo docker run hello-world`<br>



你会看到类似下面的信息<br>

![](https://dn-huntersnk.qbox.me/blog/2017/2017012201.png)



由于本地没有 hello-world 的 image<br>

Docker 会自动 pull 一个下来并执行。<br>

至此 Docker 安装完毕<br>



接下来安装 docker-compose<br>

它是一个供管理多个 docker container/容器的工具<br>



<pre><code>$ curl -L "https://github.com/docker/compose/releases/download/1.10.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

</pre></code>

先拉它下来



`$ chmod +x /usr/local/bin/docker-compose`<br>

然后给它执行权限



安装完成，可以通过<br>

`$ docker-compose --version`<br>

查看它的版本号

![](https://dn-huntersnk.qbox.me/blog/2017/2017012202.png)

## 配置



#### 一.讲解

接下来我们开始配置，为了方便懒得准备的朋友，我上传了一份配置文件目录在 [Github](https://github.com/huntersnk/ghost-nginx-ssl-with-docker-compose) 上，90%的工作都已经做好了，只需要你进去修改域名为你自己的域名即可，如果需要启用 SSL(https)，再放两个证书密钥文件就可以，下一小节快速实操部分会列出直接修改我放在 Github 部署文件的方法，如果心急可以跳过这一小节讲解部分直接看实例。



![](https://dn-huntersnk.qbox.me/blog/2017/2017012203.png)



我们要准备的文件目录就是上图中的层次关系，其中圆角矩形为文件夹，下横线为文件。

也就是说，

Deploy-folder 目录下，有四个文件和两个目录，其中 ghost-content 目录留空，ssl 目录用于放置你准备好的证书文件

我们来挨个看一下



#### docker-compose.yml

它是主要的配置文件，我们所用到的 ghost 和 nginx 实例都由它来配置执行<br>

具体内容如下



`docker-compose.yml`

<br>

<pre><code>

nginx:

   build: ./

   container_name: nginx1

   ports:

     - "443:443"

     - "80:80"

   links:

     - ghost

   restart: always

ghost:

   image: ghost

   container\_name: ghost1

   environment:

     - "NODE\_ENV=production"

   volumes:

     - ./ghost-content:/var/lib/ghost

   restart: always

</pre></code>



可以看到配置文件主题是分为两个大块构成，分别是 ==nginx== 和 ==ghost==<br>

各自的第一行就不同，nginx 下是 ==build ./== 而 ghost 下是 ==image: ghost== ，这里表示，ghost 是直接指定 image 为 ghost,也就是基于 ghost 镜像来生成容器，而 nginx 没有直接指定镜像，而是通过 build 生成容器，build 的源是 ./ 当前目录，它会在当前目录寻找 ==Dockerfile== 文件来进行构建，从而生成容器。为什么 nginx 不直接使用 image 马上会讲到，我们先继续看这个配置文件。<br>

==container_name== 字段，顾名思义定义的是容器名，你可以自行更改。

==nginx== 下的 ==ports== 字段，定义的是把 nginx 容器的443和80端口分别映射到服务器的443和80端口<br>

注意，如果你不需要启用 SSL，就删掉 - "443:443" 部分，只留下80端口。<br>

你也许会疑惑为什么 ghost 不需要映射端口，是因为我们是使用了 nginx 来做服务端，它负责响应我们的80、443请求，而 ghost 容器不需要直接链接到服务器端口，只需要在它自己在服务器内部暴露端口给 nginx，由我们配置好的 nginx 进行转发即可访问。<br>

nginx 下 ==links== 字段，定义的是给 nginx 容器指定好需要链接（转发）的 ghost 容器，因为 docker 的核心是隔离，如果不制定端口和链接的容器，容器与容器直接是不能进行通讯的，所以我们这里要告诉 nginx，你去链接 ghost 容器。

==restart: always== 字段是定义容器的运行方式，之后只要我们执行了服务，以后无论是修改配置文件还是服务器硬性 restart，这两个容器都会自动启动，不需要再干预。<br>

==environment== 字段是给 ghost 定义运行环境，ghost 默认是以 development 模式启动，而这里是用来部署，所以指定为 production 环境。<br>

==volumes:== 字段是用来把容器内的目录或文件映射到服务器本身上的。应用的程序体部分在运行时是不需要有变化的，所以放在容器里运行，结束容器后就会自动初始化，而程序部署后，由用户操作所产生的数据、数据库、配置等需要做持久化处理，所以需要映射并保存到宿主机器上来，不能运行到容器里，否则重启一下服务器，你写的文章都消失不见，你注册好的账号也都无影踪了。这里我们是把 ghost 容器内的 /var/lib/ghost 目录映射到 ./ghost-content 目录下，就是刚才图中所画的空文件夹。



####  Dockerfile

Dockerfile 就是刚才配置的 docker-compose.yml 文件中 nginx 字段下的 build ./ 命令所寻找并执行的文件，刚才说为什么 nginx 不直接调用 image ，而是要单独指定一个文件来 build 呢，是因为我们需要对 nginx 进行一些配置和修改，以及把为启用 SSL 而准备好的证书文件拷贝到 nginx 容器内。因为操作比较多，不方便单在刚才那个文件里定义，我们就把这些操作抽离出来，单独做一个 Dockerfile 文件用来 build nginx 容器，具体内容如下<br>

`Dockerfile`

<pre><code>FROM nginx

RUN rm -rf /etc/nginx/conf.d/default.conf

ADD ./nginx.conf /etc/nginx/conf.d/nginx.conf

RUN mkdir /etc/nginx/ssl

COPY ssl/ /etc/nginx/ssl/

</pre></code>



==FROM nginx== 是指基于 nginx 镜像来 build

接下来的四个操作指令都很好理解了，分别是



1. 删除默认的配置文件；

2. 拷贝当前目录下的 nginx.conf 文件到容器内的对应目录下；

3. 以及在容器内新建一个ssl 目录；

4. 把当前目录下的 ssl 目录内的证书拷贝进去。

> 注意，跟上个配置文件中的443部分类似，如果你不需要启用 SSL,就删除最后两行 RUN 及 COPY 的地方。



#### nginx.conf

这个是 nginx 的配置文件，我们在上一个文件里删除默认的，并拷贝进去一份自己的文件就是它，==注意这个文件,不要照搬!== 需要自行修改的地方比较多，需要把所有的:<br>

==www.your-own-URL.com==<br>

==your-own-URL.com==<br>

修改替换为你自己的域名



以及把所有的<br>

==/etc/nginx/ssl/yourfile1.crt;==<br>

==/etc/nginx/ssl/yourfile2.key;==<br>

修改为你自己的证书文件名<br>



> 如果你不需要 SSL，就删除所有包含 443 的 server 大块

也就是==第二个==和==第四个== server 块。



`nginx.conf`



<pre><code>

server {

    server\_name www.your-own-URL.com;

    listen 80;

    listen [::]:80 ipv6only=on;

    return 301 https://your-own-URL.com$request\_uri;

}

server {

    server\_name www.your-own-URL.com;

    listen 443 ssl;

    listen [::]:443 ssl ipv6only=on;

    ssl on;

    ssl\_certificate /etc/nginx/ssl/yourfile1.crt;

    ssl\_certificate\_key /etc/nginx/ssl/yourfile2.key;

    ssl\_protocols TLSv1 TLSv1.1 TLSv1.2;

    ssl\_prefer\_server\_ciphers on;

    return 301 https://your-own-URL.com$request\_uri;

}

server {

    server\_name your-own-URL.com;

    listen 80;

    listen [::]:80;

    return 301 https://your-own-URL.com$request\_uri;

}

server {

    server\_name your-own-URL.com;

    listen 443 ssl spdy;

    listen [::]:443 ssl;

    ssl on;

    ssl\_certificate /etc/nginx/ssl/yourfile1.crt;

    ssl\_certificate\_key /etc/nginx/ssl/yourfile2.key;

    ssl\_ciphers CHACHA20:EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH:ECDHE-RSA-AES128-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA128:DHE-RSA-AES128-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA128:ECDHE-RSA-AES128-SHA384:ECDHE-RSA-AES128-SHA128:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES128-SHA128:DHE-RSA-AES128-SHA128:DHE-RSA-AES128-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA384:AES128-GCM-SHA128:AES128-SHA128:AES128-SHA128:AES128-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4;

    ssl\_protocols TLSv1 TLSv1.1 TLSv1.2;

    ssl\_prefer\_server\_ciphers on;

    add\_header Strict-Transport-Security 'max-age=31536000; includeSubDomains';

    add\_header X-Cache $upstream\_cache\_status;

    ssl\_session\_timeout 5m;

   root /usr/share/nginx/html;

   index index.html index.htm;

   client\_max_body\_size 10G;

   location / {

       proxy\_pass http://ghost:2368;

       proxy\_set\_header X-Forwarded-For $proxy\_add\_x\_forwarded\_for;

       proxy\_set\_header Host $http\_host;

       proxy\_set\_header X-Forwarded-Proto $scheme;

       proxy\_buffering off;

   }

}</pre></code>



#### config.js

它是 ghost 的配置文件，其实正常来讲它不需要单独准备一份，因为前面 docker-compose.yml 文件里把 ghost 的目录指定挂载到服务器本地的目录后，执行 ghost 容器时会自动生成出来一份。但是，自动生成出来的配置文件有个 bug，具体现象为只能以 development 模式运行，而无法以 production 模式运行，需要在 production 内增加一段 path 定义，参见 [Ghost-BUG](https://github.com/docker-library/ghost/issues/2#issuecomment-81368071) ，所以单独准备出来，已经增加好了这段 path 定义，而你拿来用所唯一需要修改的地方就是 ==production 下的 url ==字段。



`config.js`



<pre><code>

var path = require('path'),

    config;

config = {

    production: {

        url: 'https://your-own-URL.com',

        mail: {},

        database: {

            client: 'sqlite3',

            connection: {

                filename: path.join(process.env.GHOST\_CONTENT, '/data/ghost.db')

            },

            debug: false

        },

        server: {

            host: '0.0.0.0',

            port: '2368'

        },

        paths: {

            contentPath: path.join(process.env.GHOST\_CONTENT, '/')

        }

    },

    development: {

        url: 'http://localhost:2368',

        database: {

            client: 'sqlite3',

            connection: {

                filename: path.join(process.env.GHOST\_CONTENT, '/data/ghost-dev.db')

            },

            debug: false

        },

        server: {

            host: '0.0.0.0',

            port: '2368'

        },

        paths: {

            contentPath: path.join(process.env.GHOST\_CONTENT, '/')

        }

    },

    testing: {

        url: 'http://0.0.0.0:2369',

        database: {

            client: 'sqlite3',

            connection: {

                filename: path.join(process.env.GHOST\_CONTENT, '/data/ghost-test.db')

            },

            pool: {

                afterCreate: function (conn, done) {

                    conn.run('PRAGMA synchronous=OFF;' +

                    'PRAGMA journal\_mode=MEMORY;' +

                    'PRAGMA locking\_mode=EXCLUSIVE;' +

                    'BEGIN EXCLUSIVE; COMMIT;', done);

                }

            },

            useNullAsDefault: true

        },

        server: {

            host: '0.0.0.0',

            port: '2369'

        },

        logging: false

    }

};

module.exports = config;</pre></code>



你只需要修改 production 下的 url: 'https://your-own-URL.com',<br>

把它改为你自己的域名即可。



> 如果不需要 SSL，就把 https 改成 http



#### 二.快速实操配置

如果不想自己手动挨个创建上面的文件，可以在服务器端拉取我放在 github 上的配置目录，注意如果你已经按照上一条讲解部分里配置完毕，就不用再次操作这一小节的内容了。



    $ git clone https://github.com/huntersnk/ghost-nginx-ssl-with-docker-compose



如果还没安装 git 请先



    $ sudo apt-get install git -y



再拉取，随后把证书及 key 文件拷贝到 ssl 目录下<br>

然后修改 ==nginx.conf== 文件<br>

所有 server_name 及 return 301 后面的 URL 地址改为你自己的域名地址按照对应的格式，注意是否带 s 以及是否带 www。然后把<br>

ssl\_certificate /etc/nginx/ssl/yourfile1.crt;

ssl\_certificate\_key /etc/nginx/ssl/yourfile2.key;

后面的 ==yourfile1== 和 ==yourfile2== 都改成你刚拷贝到 ssl 目录内的两个文件名



最后修改 ==config.js== 文件里 production 的 url 地址<br>

把它改成你自己的域名地址。





## 操作及部署

以上配置文件准备完毕后，即可快速部署了<br>

在配置文件目录下（也就是 Dockerfile docker-compose.yml 所在的目录）

执行



    $ docker-compose up -d

等候片刻， docker 会去自动拉取所需要的 nginx 镜像和 ghost 镜像

等全部执行完毕之后



    $ docker ps

看是否有两个容器在运行<br>

这时 ghost 应该已经在 ghost-content 目录初始化出来一些目录及文件了

然后把准备好的配置文件拷贝进去，覆盖它默认生成的



    $ cp config.js ghost-content/



随后执行



    $ docker restart ghost1

来重启 ghost 镜像，载入我们刚覆盖的配置文件



这时在浏览器里访问你的域名，就可以看到 ghost blog 了<br>

访问 你的域名/admin 即可进行后续的 ghost blog 本身管理设置。

<br>

<br>

<br>





