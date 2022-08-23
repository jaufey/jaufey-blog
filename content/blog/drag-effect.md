---
title: "碎话：DAD 拖拽效果"
date: 2022-08-09T21:22:00+08:00
lastmod: 2022-08-09T21:22:00+08:00
draft: false
tags: ["交互", "drag", "dnd", "drop", "pointer","拖拽","虚拟列表", "表格", "排序"]
categories: ["编程"]
featured: true
summary: " "
---

前一篇文章提到“有空写一篇列排序的文章”，今天就写一下。尽量做到挖一个坑，填一个坑。原生 markdown 样式或许有些问题，原文在这里：  https://jaufey.notion.site/Drag-effect-e9dc0eec82a548e78de7594d1d2f6b0b

## 功能实现

实现拖拽功能有两种方法。

1. 原生的 drag api
2. 用 pointer event 去[模拟拖拽](https://zh.javascript.info/mouse-drag-and-drop)的效果。d3 的 drag 实现也是如此

这两种方法我都在不同场景用过很多次了，简单比较一下优点和缺点。

| 比较项目 | 原生 Drag API | Pointer event 模拟 |
| --- | --- | --- |
| 功能的完整性 | **优点**：如果容器是可滚动的，<br>则鼠标移动到容器内部边缘时，<br>容器会自动滚动，这是浏览器行为。另外，原生拖拽事件不需要处理与其他事件发生冲突时的情景。 | **致命问题**：<br>鼠标拖动到容器的内部边缘时，<br>容器不会自动滚动，导致拖动物无法到达可视区域外的目标落点。<br>如果实现同样的效果则会显著增加代码的复杂度，削弱代码稳定性。<br>Notion 的 plain table 拖动表头时也有这个问题，但是左侧目录栏的 page list 中上下拖动时，列表项是有对应的滚动效果的。 |
| UI：拖动物的影像 | **缺点**：<br> 1. 原生的 effect 较丑 <br>2. 如果这个 image 是 Dom，浏览器会为其添加半透明效果，观感不佳。<br>3. dragImage 基于鼠标落下位置而声明的偏移不太准确。<br> **优点**：<br> 1. 实现简洁 | 可在拖动的生命周期内手动生成并移动拖动物的影像 DOM。一体两面的优缺点：灵活性较大，实现较麻烦。<br>**优点**：<br> 1. 自己绘制的 DOM 好看很多<br> 2. 基于鼠标的偏移量可准确控制<br> 3. 对拖动物的控制程度较高（比如在声明周期内更新此影像 DOM 的 scale） |
| UI：拖动时的指针效果 | 与 css 所支持的 cursor  类型不同，种类较少且不好看。连单纯的 move 也没有 | 可以自己在移动的生命周期内，可改变 cursor 的样式 |

---

## 列排序的交互
<video src="/post-videos/drag-column.mp4" autoplay muted style="display:block;width:95%;max-width:600px;margin-left:auto;margin-right:auto"></video>


**实现思路**：
1. 拖动开始时，绘制一列假的DOM，跟随鼠标
2. 拖动开始时，隐藏正在拖动的原始DOM

但是最终我**没有使用**这种效果，原因占比从高到低为：

1. 上文表格所列的致命问题：鼠标拖动到容器的内部边缘时，容器不会自动滚动，会导致拖动物无法到达可视区域外的目标落点。
2. 当数据量稍大时，绘制虚假 DOM 的成本变大，且掉帧严重。当然如果继续探索下去，虚拟滚动也是可以了解一下的，但是虚拟滚动除了需要手动调度外，仍需花费精力去解决那个致命问题。

---

其实对于简单场景（不包含太多数据，不会触碰到致命问题）来说，我个人是越来越倾向于 ”为了美观度而硬编码一个虚假 DOM 到代码“ 里了。曾经不想在 JS 中写大片的 HTML，但后来想想，如果把虚假 DOM 的 HTML 字符串全部放到一行的话，看起来也挺干脆的，没有副作用。

下面这种场景就完全避开了缺点，彰显了优点：

<video src="/post-videos/drag-dom.mp4" controls  autoplay muted style="display:block;width:95%;max-width:600px;margin-left:auto;margin-right:auto"></video>
