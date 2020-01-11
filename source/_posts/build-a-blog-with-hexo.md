---
title: 木犀后端分享--hexo博客搭建
comments: true
date: 2018-03-04 09:06:23
updated: 2018-03-04 09:06:23
tags: [hexo]
categories: Hexo
permalink:
---
用hexo搭建一个博客的简要步骤.
# 原料
- github账号
- git
- nodeJS和npm

# 基本步骤
1.通过[下载nvm](https://www.jianshu.com/p/8671e439a811)来下载和管理node和npm(推荐)
2.在github上新建一个名为“<你的github用户名>.github.io”的仓库
3.在系统某处依次运行下列命令:
``` bash
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```
这时，你如果访问http://127.0.0.1:4000 就可以发现你的博客已经在本地跑起来了。
4.修改博客根目录下**_config.yml**配置文件,改动两个地方:url,deploy
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: Hexo
subtitle:
description:
author: John Doe
language:
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com   #比如，改成我的配置 url: http://andrewpqc.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
  
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date
  
# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type:

#上面的deploy的配置改成我的就是下面的这样:
#deploy: 
#  type: git
#  repo: https://github.com/Andrewpqc/Andrewpqc.github.io.git
#  branch: master
```
5.下载git部署插件
``` bash
$ npm install hexo-deployer-git --save
```
6.部署
``` bash
$ hexo d -g #生成并部署
```
此时打开浏览器，输入https://<你的github用户名>.github.io就会发现你的博客已经远端部署成功了。

# hexo常用命令
``` bash
$ hexo n "我的博客" == hexo new "我的博客" #新建文章
$ hexo p == hexo publish
$ hexo g == hexo generate               #生成
$ hexo s == hexo server                 #启动服务预览
$ hexo d == hexo deploy                 #部署
$ hexo server                           #Hexo 会监视文件变动并自动更新，无须重启服务器。
$ hexo server -s                        #静态模式
$ hexo server -p 5000                   #更改端口
$ hexo server -i 192.168.1.1            #自定义IP
$ hexo clean                            #清除缓存，网页正常情况下可以忽略此条命令
```
更多具体命令请看->[这儿](https://segmentfault.com/a/1190000002632530)

# hexo主题
hexo有很多的[主题](https://hexo.io/themes/)可以使用，使用起来也非常简单。**hexo init**之后，博客文件夹中默认安装了landscape主题。如果我们还想要使用其他的主题可以直接到github上把该主题的仓库克隆到landscape所在的目录。然后更改博客根目录下的_config.yml中的theme配置选项即可实现主题的切换。

hexo主题的页面布局、配色、功能等方面一般都给了用户较大的自定义权限，用户可以通过修改主题文件夹下的_config.yml配置文件和下载插件的形式来实现自定义。[1](https://reuixiy.github.io/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html)
接下来，就开始探索你的博客吧!

# 经验之谈
1.备份很重要。
2.hexo消失了怎么办。

# 参考
[使用Hexo搭建博客](https://baoyuzhang.github.io/2017/05/12/【Hexo搭建独立博客全纪录】（三）使用Hexo搭建博客/)
[使用hexo+github搭建免费个人博客详细教程](http://blog.haoji.me/build-blog-website-by-hexo-github.html?from=xa)
[hexo主题下载及配置](http://tengj.github.io/2016/02/26/hexo2/)
[NexT主题个性化配置](https://pengbinlee.github.io/NexT-主题的安装及个性化设置/)