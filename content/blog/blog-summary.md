---
title: "使用 Hugo"
date: 2022-05-04T00:44:41+08:00
lastmod: 2022-05-04T16:42:00+08:00
draft: false
tags: ["Blog"]
---

本文列一下本博客开发使用中的问题和总结。
<!--more-->
## 过程

因为在学 Docker，而且 Hugo 文档中的 install 部分里提到了 hugo image, 所以我就自告奋勇走了 docker 这条路。
环境倒是搭好了，可以初始化、创建 post、live serve。但是每执行一个操作，我就需要在 docker-compose.yml 里改一下 command。docker-compose up 应该是可以执行单个 service 的，所以我只需要把几个命令列在 docker-compose.yml 里就可以了，但是还是感觉用起来不称手。于是在本机上下载了 release 版本。

## 问题解决
写好第一篇文章之后，部署到了 [vercel](https://vercel.com/) 上和 [netlify](https://app.netlify.com/) 上，两个部署起来都特别特别简单（后续追加部署在了 [render](https://render.com/) 上面）。  
但是都遇到了一些问题，解决方案如下。

- 统一本地与托管网站的 Hugo 版本，奇怪的是我最后是从 issue 区找到配置文件的格式的。
   1. vercel 增加配置文件 `vercel.json`
    
           {
                "build": {
                    "env": {
                        "HUGO_VERSION": "0.98.0"
                    }
                }
            }

   2. netlify 增加配置文件 `netlify.toml`

           [context.production.environment]
            HUGO_VERSION = "0.98.0"

- 同时也有一些自己的疏忽，比如忘记将要发布的 post 的 draft 置为 false