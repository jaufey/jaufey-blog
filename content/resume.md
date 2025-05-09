---
title: "简历"
date: 2024-09-28T23:00:00+08:00
lastmod: 2024-09-28T23:00:00+08:00
menu: "main"
weight: 50
toc: false
--- 

## 基础信息
<table>
<tbody>
<tr>
<td>姓名：Jaufey   </td>
</tr>
<tr>
    <td>邮箱：<a href="mailto:n-index@outlook.com" target="_blank">n-index@outlook.com</a>  </td>
</tr>

</tbody>
</table>


## 技能
- 掌握 HTML、CSS、熟悉响应式布局 、掌握 JavaScript。
- 掌握 Vue.js 相关生态，掌握 Vue Composition API。
- 使用 Webpack 及 Vite 构建项目。
- 熟悉常见的数据结构与算法，代码编写习惯良好。
- 熟悉 Git 操作，熟悉浏览器调试方法，掌握常见前端性能优化方法。
- 能阅读英文文档，可以说非常喜欢看文档。

--- 

## 项目经历

### 1. DataFocus 官网及文档站点
- <a href="https://www.datafocus.ai" target="_blank">官网</a>
    - 从零开始，90+ 个页面还原 UI 设计，支持响应式布局，支持国际化。  
    - 编写 Node 打包脚本，实现中英文所有页面静态直出，编译时间控制在5分钟内。
    - 实现登录注册、埋点分析、图片懒加载等常见需求，负责日常维护。
- 产品文档站点<sup><a href="https://wiki.datafocus.ai/" target="_blank" title="DataFocus Cloud 数据分析产品文档">1</a></sup><sup>,</sup> <sup><a href="https://www.datafocus.ai/documentation/dataspring/" target="_blank" title="DataSpring ETL 工具产品文档">2</a></sup>
    - 将文档内容从 MediaWiki 迁移至 VitePress，并负责日常维护。
- <a href="https://www.datafocus.ai/infos/" target="_blank">博客站点</a>
    - 将博客内容从 WordPress 迁移至 Strapi CMS + Nuxt.js SSR
    - 负责后端逻辑、前端页面的编写，开发 Strapi 插件赋能运营人员。

### 2. DataFocus Cloud 数据分析工具 Web 端
本人负责此项目前端的部分功能。  技术栈：Vue.js 2, D3.js  
- 实现系统内的部分功能模块，如用户管理、部门岗位、权限管理、数据应用等常规功能。
- 维护系统内存量图形，优化图形的动画效果，调研并使用 D3.js 实现一些新增图形的需求。
- 封装系统内常用组件，推进项目内文档落地。
- 跟踪修复 Bug，支持产品迭代。

### 3. DataSpring ETL 数据处理工具 Web 端
本人负责此项目前端的所有功能。  技术栈：Vue.js 3, Naive UI, Vite, VueUse, Unocss  
- 实现登录、用户管理、任务流管理、算子管理、定时任务、日志管理等功能。
- 任务流编辑功能：
    - 实现拖拽控制节点连线、自动布局、图操作等画布相关功能。
    - 实现 40 个、多种类型的数据处理器的逻辑实现；封装相同类型的处理器实现。
    - 使用 Yup + VeeValidate 封装表单验证组件，支持国际化。
- 性能优化：bundle-analyzer 分析打包大小并优化：国际化分包、异步组件加载等。

-----
谢谢你的阅读。