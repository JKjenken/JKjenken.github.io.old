---
layout:     post
title:      "Hello My Blog"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-12-06 21:00:00
author:     "linjiankai"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 生活
    - 部署
---

> “Just do it. ”


我的 Blog 开通了.

[comment]: <> ( [跳过絮絮叨叨 ](#build) 

2019年的最后一个月,也是21世纪10年代的最后一个月,我终于下定决心要开通博客啦！

作为一个程序员,真的要具备极客思维,要善于捣鼓,善于折腾,这次花了几天时间,终于把博客捣鼓出来了接下来一定要好好的维护它.

很久前我就畅想着在互联网拥有自己的名片,拥有一个私人的空间,能够分享自己的生活、学习的知识、喜欢的歌,着迷的音乐,热衷的剧集.

从今天开始,just do it！


<p id = "build"></p>

## 正文

第一篇博文,我想给大家分享一下我是如何搭建这个博客的.  

在这里要特别感谢[黄玄](http://huangxuan.me)无私的分享,整个博客的源码都来源于他的github仓库,我去除了disqus评论功能(在国内使用受限),改为使用gitalk.

这个博客的搭建是通过 [GitHub Pages](https://pages.github.com/) + [Jekyll](http://jekyllrb.com/)

优点：
* **Markdown** 写作
* 通过Git提交博文
* 搭建特别简单,利用Github Pages提供的域名和免费空间,不需要再去购买服务器


---

直接fork我的项目或者其他的模版库到自己的github 就可以直接运行了,接下来就是搭建本地调试环境

很多博文都推荐使用`gem install jekyll`,但是一些系统没有内置Ruby语言(jekyll依赖于Ruby)的话,就还需要安装Ruby,再执行其他的命令并且还需要注意版本适配的问题⚠️

因此我采用docker来部署jekyll,下面我贴出docker-compose.yml的配置(关于docker的使用有疑问的朋友可以参见网上的其他相关资料)

```
dockerfile
jekyll:
    image: jekyll/jekyll:pages
    command: jekyll serve --watch
    ports:
        - 3999:4000
    volumes:
        - /Users/linjiankai/IdeaProjects/jenken.github.io:/srv/jekyll/

```
挂载你自己对应的数据卷目录位置后,执行`docker-compose up` 即可

接下来就是真正有价值的漫长的编写博文、发布博文的时间,关键是坚持不懈的把自己的博客更新下去,输出自己觉得有价值的东西.

不断反思,不断思考.

互勉.


