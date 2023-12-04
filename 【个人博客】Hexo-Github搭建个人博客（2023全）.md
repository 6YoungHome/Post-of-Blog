---
title: 【个人博客】Hexo+Github搭建个人博客（2023全）
typora-root-url: 【个人博客】Hexo-Github搭建个人博客（2023全）
cover: /img/blog_img/11.png
description: 利用Hexo模板，在github pages上构建个人博客
abbrlink: de56ed25
date: 2023-07-05 10:44:35
tags:
  - Hexo
  - 个人博客
  - Github pages
categories:	
  - 个人博客
---









# 相关环境准备

## 安装Node.js

{% link NodeJS,NodeJS,https://nodejs.org/en/  %}

可以参考由 Node.js 提供的 [指导](https://nodejs.org/en/download/package-manager/)。

## 安装git

{% link Git,Git,https://git-scm.com/ %}

大家查询相关资料下载即可

### 安装 Hexo

```bash
npm config set registry https://registry.npm.taobao.org # 将npm源替换为阿里的镜像，安装更快
npm install hexo-cli -g # 安装Hexo
hexo -v # 返回Hexo版本，确定安装成功
```

# 博客构建

## 本地测试

在本地创建一个目录，然后运行下述命令

```bash
hexo init # 初始化博客
hexo generate # 申城网站信息
hexo server # 开始网站服务
```

然后在浏览器中输入 localhost:4000 , 就能看到，我们的博客就初步搭建成功啦

![hello world](A7DdZq.png)



## Github部署

首先注册一个[github](www.github.com)账号，简单学习一下Github的使用方法，不会的可以查一查，这方面的资料非常多。

相信这一步应该不用教学吧ψ(._. )>

在你的github中新建仓库“username.github.io”，其中，username就是你注册时使用的用户名。

修改你博客根目录下的_config.yml文件中的deploy配置项：

```
deploy:
  type: git
  repository: git@github.com:username/username.github.io.git  # 你的仓库地址
  branch: gh-pages
```

他会在你的仓库的gh-pages分支部署你的页面，然后再master分支保存你网站的源代码，不会相互覆盖。

通过如下代码进行部署：

```
hexo clean # 清除缓存
hexo generate # 生成网站
hexo deploy # 部署至远程网络

# 简写
hexo cl
hexo g
hexo d

# 简简写
hexo cl
hexo g -d  # hexo d -g
```

然后就可以在浏览器上输入username.github.io访问你的博客了。

# 博客优化

## 网站信息

```
title: 网站名称
subtitle: '子标题'
description: '网站描述'
keywords:
author: 你的名字（昵称）
language: zh-CN
timezone: ''
```

![image-20230707162938139](/image-20230707162938139.png)



## 网站导航

修改根目录下的_config.landscape.yml文件

```
menu:
  首页: / || fas fas fa-home
  分类: /categories/ || fas fa-folder-open
  时间轴: /archives/ || fas fa-archive
  标签: /tags/ || fas fa-tags
  友情链接: /link/ || fas fa-link
  关于我: /about/ || fas fa-heart
```



## 关键页面生成

进入 Hexo 博客的根目录，执行以下命令生成页面：

```
hexo new page pagename
```

此处的pagename为categories、archives、tags、link、about等

打卡source/pagename/index.md，修改其中内容：

```
---
title: 中文pagename
date: 2023-06-28 12:53:45
type: "pagename"
---
```



如此就初步构建好了你的博客啦，之后可以进一步使用butterfly主题优化你的博客。





