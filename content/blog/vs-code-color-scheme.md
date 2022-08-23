---
title: "配置 VS Code sidebar 的主题色"
date: 2022-05-08T22:09:00+08:00
draft: false
tags: ["VS Code","Telegram Desktop"]
categories: ["工具"]
summary: " "
---

最近看黑色主题有点不够舒服，想换成白色。编辑器的 color scheme 设置很好找，但切来切去都无法把 sidebar 改成白色主题。查后解决了，具体方法是在 settings.json 里手动加上这么一个配置项：

```    
    "workbench.colorCustomizations": {
        "activityBar.background": "#f4f4f4",
        "activityBar.foreground": "#000000"
    }
```

----

借此，我又想到了以前解决过的一个问题：

Telegram desktop 最开始是没有 sidebar 的，你可以在某个频道寻找自己喜欢的皮肤来安装。但是后来多出来个 sidebar 区域。当时的所有皮肤是没有对这个 sidebar 进行适配的，导致原生主题的 sidebar 跟已有皮肤的主界面格格不入。

经过搜索，发现有人跟我遇到同样的问题：[链接🔗](https://github.com/telegramdesktop/tdesktop/issues/7509)，问题在他的热心帮助下得以解决。

总结：皮肤包本身是一个压缩文件，把你拿到的皮肤包后缀改成 zip 然后在解压出来的文件里增加一些关于 sidebar 的配置项即可，读者可对比这个文件对自己的皮肤进行修改:

[配置文件](/files/japanserenity.tdesktop-theme)

效果图如下：
![config](/post-images/telegram-sidebar.jpg)

