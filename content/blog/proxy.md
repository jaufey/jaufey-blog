---
title: "当 Proxy 照进现实"
date: 2022-07-16T02:12:22+08:00
lastmod: 2023-07-03T19:23:00+08:00
draft: false
tags: ["ES6","proxy","object method defination","this", "reactive", "arrow function"]
categories: ["编程"]
summary: " "
---


## 前提

在 ES5 及以前，我们为对象定义一个方法的时候，需要这样：
```javascript
const person = {
    bark: function(){}
}
```

在 ES6 之后，JS 允许写这样的代码：  
（消除了冒号和 function 关键字，并且，这种简写方法可以使用 super 关键字来修饰）
```javascript
const person = {
    bark(){}
}
```

---

## 实践

目前在做一个 workflow 的编辑面板，其中每个 node 都是可拖拽的。  
于是需要在更新坐标数据的时候，同时更新这个 node 的 UI 位置。

```javascript
export default class Node{
    constructor(el){
        this.el = el;
        this.curPos = {x:0,y:0};
    }

    moveTo(x,y){
        this.curPos.x = x;
        this.curPos.y = y;
        this.el.style.left = x + 'px';
        this.el.style.top = y + 'px';
    }
}
```

于是，一个想法自然而然就会产生：坐标数据的更新和 UI 的更新几乎总是同步的，为什么不根据数据的变更去自动更新 UI 位置呢，所以第一次尝试使用 proxy。（虽然目前的开发框架整体是响应式的，但绘制画布这一块还是手动去维护逻辑的。手动维护在我看来还是比较有必要的）



```javascript
export default class Node{
    constructor(el){
        this.el = el;
        this.curPos = new Proxy(
            // target
            {x:0,y:0},
            // handler
            {
                get(target, prop){ 
                   return target[prop];
                },
                set(target, prop, val){
                    const which = prop === 'x' ? 'left' : prop === 'y' ? 'top' : '';
                    if(which){
                        target[prop] = val;
                        this.el.style[which] = val + 'px';
                        return true;
                    }else{
                        // assert - incorrect prop, throw error or failed with silence
                        return false;
                    }
                }
            });
    }

    moveTo(x,y){
        this.curPos.x = x;
        this.curPos.y = y;
    }
}
```

看起来多此一举，代码并没有减少多少，甚至看上去更乱了。
但谁知道呢，有两个原因鼓励我继续探索：  
1. 想用一下 proxy，至少也要浅尝辄止
2. 这里的例子只是最小复现，未来很可能会有 moveXTo, moveYTo, moveXBy, moveYBy 等方法都可以触发此 proxy handler

---

## 问题及总结

可是上面的代码是有问题的：
proxy 的 setter 里的 this 指向了 这个 proxy 的 handler，
我几乎没有用过 bind 去改变 this 的指向，现在该不会要 bind 吧，多么古旧的做法。
一定是什么出了问题，莫非这是 proxy 的特异（exotic）行为？  

经过两个 console.log 的排查，确认了这个 setter **不是一个箭头函数，而是一个普通的匿名函数**。
改成箭头函数之后，this 就能拿到正确结果了。


直觉欺骗了我。



-----

后续：
几个月前借着重构 UI 的时间，把这坨屎铲了。——2023年7月3日。
