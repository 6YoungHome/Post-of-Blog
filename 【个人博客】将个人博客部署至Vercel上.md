---
title: 【个人博客】将个人博客部署至Vercel上
typora-root-url: 【个人博客】将个人博客部署至Vercel上
description: 将你的个人博客从github pages更换为部署至Vercel上
cover: /img/blog_img/13.png
tags:
  - Hexo
  - 个人博客
  - Github pages
  - Vercel
categories:
  - 个人博客
abbrlink: 1f6043a5
date: 2023-07-10 00:36:58
---





[Vercel](https://vercel.com/)是一个极佳的提供网站托管服务的平台，在构建自己的个人博客时可以解决很多我们遇到的问题。本网站就是基于Hexo引擎模板开发，托管在Vercel上的。

{% link Vercel,Vercel,https://vercel.com/  %}

![image-20230710010639582](/image-20230710010639582.png)

# Vercel的优势

1. 生成博客时不需要自主生成并部署，直接将本地内容push到github仓库，vercel会自动部署并更新博客内容。
2. 由于历史原因github pages不能被百度搜索引擎收录，而通过Vercel搭建的博客可以被百度收录。
3. 可以将github仓库源代码设为私密，防止个人博客源代码的泄漏。



# 部署方法

下面我将介绍如何将自己的github pages 部署至Vercel中。

## 准备工作

- 首先，通过自己的github账号注册并登录Vercel。

![image-20230710010830669](/image-20230710010830669.png)

- 关联自己的github账号以后，可以导入自己博客所在的仓库username.github.io。

![image-20230710111605977](/image-20230710111605977.png)

## 博客部署

- 选择自己博客选用的构建框架（我使用的是Hexo，还有很多诸如Hugo等框架可以使用），选择Deploy进行部署。

![image-20230710140005941](/image-20230710140005941.png)

- 部署成功之后就会出现如下页面，展示你的博客封面，并进行一波Congratulations！！！

![image-20230710140214579](/image-20230710140214579.png)

- 在Add Domain 处可以为博客配置自己购买的域名，当然你使用Vercel自动生成的域名也可以。

![image-20230710140420457](/image-20230710140420457.png)

- 在此处添加你的域名，并根据要求进行DNS配置即可（在你购买域名服务的地方一般会提供免费域名解析服务（DNS），为你的域名添加76.76.21.21的ip即可实现）
- 看到如下界面说明你的博客部署成功。

![image-20230710140714148](/image-20230710140714148.png)

## 其他配置

可以选择将github的仓库设置为私人，这样就可以避免自己的源代码泄露。

![img](/2021-12-31_19-16.png)
