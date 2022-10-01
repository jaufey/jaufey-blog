---
title: "Static hoisting 静态提升"
date: 2022-10-01T13:35:00+08:00
draft: false
tags: ["经验", "随想", "Vue", "Vue2"]
categories: ["编程"]
summary: " "
---


文章地址：[https://jaufey.notion.site/Static-hoisting-85e751cf04694a47a1b8d9f9f04cfb35](https://jaufey.notion.site/Static-hoisting-85e751cf04694a47a1b8d9f9f04cfb35)

----


在 JS 中，有一个术语叫做 [Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)，指在编译期将变量、函数、类的声明**提升**至作用域顶端。

本篇文章所讲的 Static hoisting 与 Hoisting 有一点相似，但相关性不大。Static hoisting 是 Vue3 在模板编译阶段的一种优化手段，恰好有一个**提升**的操作，所以名字中包含 hoisting，Static 则表示 hoisting 的对象是静态的。

----

## 我是如何感知到 “静态提升” 的

[某项目从 Vue2 升级到 Vue3 后](https://www.notion.so/Vue-3-48f24aab000244dfb4583c7948221a24)，出现了一个 “副作用未被销毁 / 副作用被 cache” 的问题。

### 问题描述

1. 假设有个 v-if=”toggle” 控制的 A 组件。
    1. 在 A 组件初始化的时候，**徒手**在组件元素中绘制了一个圆圈(**这算是框架内的未定义行为、副作用**)。
    2. 当 toggle 由 true 切为 false 再切回 true 的时候，即便我没有显式指定任何缓存方式，A 组件却仍保留了上一次初始化时绘制的圆圈，导致每展示 A 组件一次，副作用便累积一次。
2. 此问题**只在构建后出现，而不会在开发间出现**。脚手架配置为 Vue-cli 默认配置。
3. 副作用只是累计了副作用中的 DOM 元素，而事件监听等功能则没有被缓存。

在线示例：[https://demo-0814-test.vercel.app/](https://demo-0814-test.vercel.app/)

代码：[https://github.com/N-index/demo-0814](https://github.com/N-index/demo-0814)

### 问题解决

经过排查与请教，发现是 “[静态提升](https://vuejs.org/guide/extras/rendering-mechanism.html#static-hoisting)” 这个构建期的**优化手段**导致了这个问题。

解决方案为以下任意一种：

1. 在构建阶段，关闭静态提升的功能： [https://github.com/vuejs/core/issues/5256#issuecomment-1173723516](https://github.com/vuejs/core/issues/5256#issuecomment-1173723516)
2. 在副作用的父容器上添加 ref 属性，使得静态提升越过这个静态节点。
    
    具体参考：[ref 和 key 的编译示例](https://vue-next-template-explorer.netlify.app/#eyJzcmMiOiI8ZGl2PlxuICA8ZGl2IGNsYXNzPVwiYVwiPmZvbzwvZGl2PlxuICA8ZGl2IGNsYXNzPVwiYlwiPmZvbzwvZGl2PlxuICA8ZGl2IGNsYXNzPVwiY1wiIGtleT1cInN0YXRpY0tleVwiPmZvbzwvZGl2PlxuICA8ZGl2IGNsYXNzPVwiZFwiIDprZXk9XCInc3BlY2lhbEtleSdcIj5mb288L2Rpdj5cbiAgPGRpdiBjbGFzcz1cImVcIiByZWY9XCJzcGVjaWFsRm9vXCI+Zm9vPC9kaXY+XG4gIDxkaXY+e3sgZHluYW1pYyB9fTwvZGl2PlxuPC9kaXY+Iiwic3NyIjpmYWxzZSwib3B0aW9ucyI6eyJob2lzdFN0YXRpYyI6dHJ1ZX19)
    

### 问题总结

一句话描述 “静态提升” ：在模板编译阶段生成渲染函数的过程中，对于静态内容，Vue 将其对应的 vnode 结果 hoist 到了渲染函数外部，而不是每次都渲染都重新生成 vnode。(示例：[编译期间的静态提升](https://vue-next-template-explorer.netlify.app/#eyJzcmMiOiI8ZGl2PlxuICA8ZGl2PmZvbzwvZGl2PlxuICA8ZGl2PmJhcjwvZGl2PlxuICA8ZGl2Pnt7IGR5bmFtaWMgfX08L2Rpdj5cbjwvZGl2PiIsInNzciI6ZmFsc2UsIm9wdGlvbnMiOnsiaG9pc3RTdGF0aWMiOnRydWV9fQ==))


Vue3 基于这样一个**假设**，才有了这个优化策略：  
&ensp; &ensp; &ensp; &ensp; 静态内容不会被改动，每次都会渲染出一样的 DOM，所以直接缓存起来就可以了。

由此，可推断出上面所说的第 2 种方案之所以可行，是因为：  
&ensp; &ensp; &ensp; &ensp;编译模板时遇到 ref ：哦这个 DOM 大概率不是静态的，所以就不提升了。

另外，问题描述中的第 3 点**可能**是因为这个原因：  
&ensp; &ensp; &ensp; &ensp;[大量静态内容时会使用 cloneNode](https://vuejs.org/guide/extras/rendering-mechanism.html#static-hoisting:~:text=same%20piece%20of%20content%20is%20reused%20elsewhere%20in%20the%20app%2C%20new%20DOM%20nodes%20are%20created%20using%20native)，而且 cloneNode 指挥克隆 DOM 结构而不会克隆事件。只是猜测，因为我无从得知本文场景中第二次渲染静态内容的时候是否使用了 cloneNode。

-----

## 延伸思考

问题在开发阶段不会暴露，而只在编译后才暴露。说明了**基于假设的、默认开启**的优化策略在遇到 未定义行为的时候，有不小的概率会产生未预期的结果。我们常调侃 “在我机器上是正常的呀”，但在这个问题上 Vue3 又给我加了一码：“这东西在开发期间是正常的呀！”，要说这个 Feature 有什么问题，“没写进 breaking change” 或许算一个。