---
layout: post
title:  "基于 Rails 和七牛的图片上传小工具"
date:   2015-01-08 02:17:42 +0800
---
我 blog 上的图片一直都是存储在[七牛云存储](http://www.qiniu.com/)上，几年前用 Jekyll on Github 时是手工在七牛网站里上传，然后一张张获取 URL；后来用 Wordpress 时有现成插件，可以在博客后台上传图片时，直接也传给七牛一份，然后用七牛返回的图片地址替换服务器的图片地址即可；再后来，我切换到现在一直使用的 Ghost，回到了手工在七牛网站里上传图片的日子，所以我一直想做个上传用的小工具，方便我给 blog 插入图片时迅速获取图片 URL，搜索后发现有其它开发者做的 Ghost with Qiniu 修改版，优点是可以通过 Ghost 的后台上传图片，但缺点是因为修改了 Ghost 源码，就无法再直接更新 Ghost 新版本，加上我对 javascript 不熟，就放弃了修改 Ghost 的思路，转而用 Rails 来尝试开发，于是就有了这个小工具。

目前实现了



1. 上传至指定的七牛 Bucket 以及目录 URL

2. 一键复制 URL 地址或者 Markdown 格式的图片插入地址



基础功能的实现跟网上流传的 "Pinterest" with Qiniu and Rails in 15 minutes 类似，目前只是做了一些针对上传 URL 定义，拷贝及图片 Preview 的修改，后续计划完善图片上传功能，以及跟自身博客搭配使用等更方便的优化，比如多图片同时上传（这是七牛官方后台早就实现了的功能），最终目的是要在各个使用维度上都比七牛自己的上传页更方便使用，才有存在的意义。



## 版本

V0.1 @ 20170107



## 需求：



* OS：Linux or macOS

* Ruby version = 2.3.1

* Rails version = 5.0.1



## 使用：



首先从 [Github](https://github.com/huntersnk/Upload-Pic-With-Rails-Qiniu) 上克隆代码到本地<br>

拷贝根目录下的 sercrets.config  到  config/secrets.yml<br>

把你自己的七牛 qiniu_access_key 和 qiniu_secret_key 填入 secrets.yml 文件<br>

> 七牛 key 获取[地址](https://portal.qiniu.com/user/key)



![secrets](https://dn-huntersnk.qbox.me/blog/2017/2017010801.png)



修改 /app/controllers/pictures_controller.rb 文件<br>



    ... ...

    def create



    # 第30行

    outurl = '这里修改为你自己的七牛URL'



    upload_ret = JSON.parse(Base64.urlsafe_decode64(params[:upload_ret]))

    ... ...

    private



    def generate_qiniu_upload_token



    # 第78行

    bucket = "修改这里为你自己的 bucket 名"



    put_policy = Qiniu::Auth::PutPolicy.new(bucket)



    # 第83行，可选修改上传自定义URL，这里默认是 你的域名/test/2017/文件名 方式

    put_policy.save_key = 'test/2017/${fname}'

     

    ... ...



> 更多七牛相关的配置可以参考[七牛开发者中心](http://developer.qiniu.com/)



然后安装依赖包



 local$: bundle install



如果提示 mime-types 版本过高，执行



 local$: bundle update mime-types

 

创建数据库



 local$: rails db:create



执行迁移



 local$: rails db:migrate



启动服务器



 local$: rails s



在浏览器内访问 localhost:3000<br>

即可进行图片上传测试<br>

![upload1](https://dn-huntersnk.qbox.me/blog/2017/2017010802.png)



![upload2](https://dn-huntersnk.qbox.me/blog/2017/2017010803.png)



![upload3](https://dn-huntersnk.qbox.me/blog/2017/2017010804.png)



![upload4](https://dn-huntersnk.qbox.me/blog/2017/2017010805.png)



## 后续计划（todo）：

* 多图片同时上传

* 拖动上传

* 在网页端自定义图片上传地址

* 图片水印接口

* 工具本身美化

<br>

<br>

<br>