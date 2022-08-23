---
title: "Markdown 小记"
date: 2022-05-04T03:45:00+08:00
draft: false
tags: ["Markdown"]
categories: ["工具"]
summary: " "
---

本篇不断更新使用 markdown 中的问题及总结

1. Q: 如何在一个 blockquote 中换行？  
   A: 在前一行的末尾添加两个以上的空格即可。同样的可适用于列表项等其他需要换行的地方。


2. Q: hugo 的 post 里放如何放置图片？
   A: 放到 /static 目录下就可以了，如果有其他类型的静态资源需要区分，那么放到 /static/images 里也可以。

3. Q: hugo build 时会将 static 目录放到站点的根路径下，但是 VS Code 本地编辑 markdown 的时候，放置图片时的文件目录自动补全的根路径是当前项目的根目录。于是出现一个问题：
   如果按照 hugo 的图片 url 来引用， 那么 markdown preview enhanced 预览就无法在本地索引到正确图片。
   如果按照 markdown preview enhanced 的路径来预览图片，那么在 build hugo 或者 push hugo to github/cloud 的时候，就需要提前修改为 hugo 可用的 url。如果 markdown preview enhanced 或者 vscode 有个配置项来修改本地自动补全的路径就好了。
   
   A: 经过查询，自动补全有这么一个配置项可以加。`"markdown.extension.completion.root": "static"`。这样一来，图片目录可以快速、正确地找到了。但是 vscode 本身的 image preview 显示不了图片了，而且 markdown preview enhanced 预览模式里仍旧找不到图片。  
   但针对这个问题再继续解决下去就有点琐碎了，像是在写茴字，于是到此为止。  
   需要预览的话可以本地启动 hugo server，不要在 markdown preview enhanced 里预览了，一开始想在 markdown preview enhanced 预览的原因是 markdown 足够轻量，不太关心样式，不需要分屏。