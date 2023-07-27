---
title: "升级到 Vue 3"
date: 2022-08-16T22:15:41+08:00
lastmod: 2022-08-16T23:52:41+08:00
draft: false
category: ["Helloworld"]
tags: ["Vue","Vue3","framework","legacy"]
featured : true
summary: " "
---


原文地址：[https://jaufey.notion.site/Vue-3-48f24aab000244dfb4583c7948221a24](https://jaufey.notion.site/Vue-3-48f24aab000244dfb4583c7948221a24)

---

上周末花了一些时间把手头的一个小项目从 Vue 2.6 升级到 Vue 3.2，把 webpack 从 2 点多升到了 5。本篇文章记录一下过程。

## 迁移原因

1. **心理原因**：这个小项目的模板是从一个旧的 monorepo 的项目脱胎出来的，webpack 的配置有些杂乱有些绕，属于能跑就行。改不太动，索性升级，不然这块内容对我来说永远是技术债，是不想去触摸的一团模糊。
2. **直接原因**：想给应用加一个 [Notification](https://developer.mozilla.org/en-US/docs/Web/API/notification) 的功能，这个功能需要 https 环境。 但是 webpack-dev-server 的 https 配不成功，开启 https 后一访问服务就崩溃。
3. **其他原因**：积累经验

---

## 迁移目标

1. 原先的项目能跑
2. 原先的 webpack 配置，觉得有用的，在迁移后要保留下来
3. 在迁移后，能够通过阅读 webpack 文档
    1. 自由地测试各提供方的 Loader 和 Plugin
    2. 自由地测试 webpack 的新功能
    3. 将来可从工具链角度优化项目性能，使其不再是一团模糊
    4. 进阶：以 webpack 为抓手，扩展了解不同构建工具：gulp, babel, parcel, snowpack，esbuild，vite，swc。（这些名次对我来说很眼熟，但只在初学 HTML 时接触过 parcel，所以不好意思有可能把不同类型的东西并列在了一起）

---

## 迁移了哪些内容

在迁移之前，首先是过了一下这个里面提到 [https://v3-migration.vuejs.org/](https://v3-migration.vuejs.org/) 的问题。

因为首次完整通读了的文档版本就是 Vue 3 的，所以不会心怯，也不需要再读一遍了。

（这是基于初识 Vue 3 所练手的一个小东西：[Github](https://github.com/N-index/games)）

**迁移目标的第一点**决定了我几乎只记录 breaking change。

1. 项目上的改动：
    - Vue 的 protoType 上面不能绑东西了，如果仍需在组件内使用公共方法，则需要在 Vue instance 的 globalProperties 上绑。（迁移文档中有）
    - $set 和 $delete 不存在了。Vue 3 的响应式基于 proxy，可放心地给 Object 增删属性。（迁移文档中有）
    - 原生支持 teleport 内置组件（基本文档中有）
        - 所以之前封装的模态框组件不再需要手动挂载到全局上，转而使用 teleport 包装。
        - 文档中没有提到的变动：在在子组件中使用 $parent 时，Vue 2 不会获取到自定义组件的实例，会跳过。但**在 Vue3 中 transition/teleport 等内置组件都会作为组件实例被 $parent 获取**到。导致我的一些很拉的代码无法 work 了：
            - 在原先的实现中，模态框组件中的  slot 内容想要关闭模态框时，会调用 $parent.closeModal 函数来实现目标。
            - 但升级以后 $parent 获取不到正确的模态框组件实例，所以就会出问题。
            - 怎么解决的呢？外层模态框 provide closeModal 方法，内部 slot 内容组件 inject closeModal 方法，但这是不负责任的飞线做法。正确的做法参考（[https://stackoverflow.com/questions/50942544/emit-event-from-content-in-slot-to-parent](https://stackoverflow.com/questions/50942544/emit-event-from-content-in-slot-to-parent)，改天还需要熟悉一下 slot 的作用域）
    - 生命周期钩子函数名字变动：（迁移文档中有）
        - 组件生命周期：beforeDestroy，destroyed 等改为 mount 所对应的字眼
        - 自定义指令生命周期：inserted, update 等变动
    - 模板中的 filter 被移除。以后都建议使用 computed 来实现相同的效果。（迁移文档中有）
    - new Vue 改为 createApp（迁移文档中有）
    - 事件及事件触发的对象改变。不需要 .native 去获取原生事件了（迁移文档中有）。在 Vue 2 开发时踩过这个坑
    - 组件双向绑定变了。Vue 3 弃用 value+input 的组合，默认为 modelValue+update:modelValue，还可多值双向绑定了。（迁移文档中有）
    - 在 router-view 标签内如果添加了(loading)内容，则路由所对应的组件内容将不会被渲染。（迁移文档中无）
    - 注释放到模板内容第一行的话，$el 的值就是错误的（迁移文档中无）
2. 依赖上的改动
    1. vuex, vue router 升级，但项目没有用到什么新东西
    2. 一些普通的运行时依赖：
        - 如果还能用，没有报错，就没有升级
        - 对 Vue2 和 Vue3 区分了不同版本的依赖
           - 如 vue-toastification 则进行了升级
           - 如 vue-codemirror 报错，其有 Vue3 版本
             - 配置方式发生改变，API 接口改变。兼容 Vue 2版本的 vue-codemirror 的配置粒度较细，兼容 Vue 3 版本的 vue-codemirror 提供了一个 basic-setup 启动配置，基本满足需求。像原来的 keymap 如今包含在 basic-setup 中了
             - 引入依赖的方式发生改变。现在需要单独安装需要用到的 codemirror extensions。比如 language package 或 theme，有利于摇树、节省体积。
             - Vue 3 版本的 vue-codemirror 还待完善，比如 theme 目前只支持一个 one-dark。由此可见虽然 Vue 3 的主生态算是比较完善了，但各依赖对 Vue 3 的适配还在“进行时”，尤其是像 codemirror 这种比较重的东西。
3. 工具链上的变动
    1. Vue 3 对 webpack 的配置进行了比较好的封装，几乎做到了开箱即用。
    2. 如果有自定义 webpack 配置的需求，则在 vue.config.js 中追加配置即可。
        1. 简单的基础配置
        2. 链式调用 webpack chain。在达到迁移目标的第3点的过程中需要多了解。

---

## 迁移目标完成情况

1. （完成）原先的项目能跑
2. （完成）原先的 webpack 配置，觉得有用的，在迁移后要保留下来
    1. Vue Cli 流程走下来该有的几乎都有了
    2. 还缺一个 postcss 未看到效果，待测试安装
3. （持续进行）在迁移后，能够通过阅读 webpack 文档调试工具链

**下一步计划（一个月以内完成）**：

1. 配置 eslint、postcss 
2. 学习使用 composition api 语法
3. 学习使用 [VueUse](https://vueuse.org/)

---

以上所描述的迁移内容大都只是按部就班的东西。

下一篇文章，讲下迁移过程中遇到的一个重点问题：[static hoisting 及思考](/blog/static-hoisting/)
