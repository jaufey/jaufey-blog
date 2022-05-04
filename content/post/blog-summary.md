---
title: "使用 Hugo"
date: 2022-05-04T00:44:41+08:00
draft: false
tags: ["Blog"]
---

本文列一下本博客开发使用中的问题和总结。

因为在学 Docker，而且 Hugo 文档中的 install 部分里提到了 hugo image, 所以我就自告奋勇走了 docker 这条路。
环境倒是搭好了，可以初始化，也可以创建 post，也可以 server 了，但是每执行一个操作，我就需要在 docker-compose.yml 里改一下 command。docker-compose up 应该是可以执行单个 service 的，所以我只需要把几个命令列在 docker-compose.yml 里就可以了，但是还是感觉用起来不称手。于是在本机上下载了 relase 版本。

写好第一篇文章之后，部署到了 vercel 上和 netlify 上，两个部署起来都特别特别简单。
但是都有一些问题：
1. vercel 的问题在于其文档中标注了它的 hugo 版本是 v0.58.2，而我本地开发用的是 v0.98.0。导致出现一个问题，vercel 没能正常处理列表项中的换行——本应属于同一列表的列表项被分割成了多个只含一个列表项的列表。这个绝对不能忍。目前还不知道有没有办法升级 vercel build 时的 Hugo 版本。我倒是在皮肤包里看到了 netlify 的配置文件，里面声明了 Hugo 版本是 0.74.3，所以我要去看一下 vercel 有没有配置文件可以写。
2. netlify 则有一个问题，版本配置对了，但是好像 build 不成功。新加页面展示不出来