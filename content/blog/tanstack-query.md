---
title: "体验 TanStack Query 遇到了问题"
date: 2023-06-22T11:17:00+08:00
lastmod: 2023-06-22T11:17:00+08:00
draft: false
category: ["前端"]
tags: ["fetch","deduplication","server state management"]
featured : true
summary: " "
---

## 前言
首先，[TanStack Query](https://tanstack.com/query/latest) 是什么？它是一个异步状态管理库，最常用于服务端状态的管理。它提供了请求去重、缓存、后台更新过期数据等功能，且开发者不需要考虑缓存数据的垃圾回收等问题。开发者获得一些好的开发体验，用户获得更流畅的使用体验。

我们可以把它看作 useFetch() 的强化版。最常使用的 API 见这段代码：
```javascript
const {isLoading, error, data, refetch} = useQuery({
    queryKey: 'getList',
    queryFn: getList
});
```

## 问题
虽然本人的使用时间较短，使用范围较小，但确实遇到了一些问题，所以本文就谈一下。

### 用户层面
1. 对用户来说，由 TanStack Query 加持的网站打破了其以往浏览网页时的心智模型。至少对于我这种拥有用户兼开发者双重身份的人来说，打开控制台浏览网页已是司空见惯，所以当我进入一个页面而没有看到 Network  发送请求时，难免怀疑是不是网页出了BUG，怀疑我看到的内容是不是最新的，进一步如果内容不是最新的，会继续怀疑 F5 刷新网页后会是最新的吗？即便不开控制台，那么进入页面后没有出现 loading 效果也足以引发怀疑——“这跟我以前的上网体验不一致”，于我而言，进入网页后有个短暂的 loading 效果并不是什么减分项，这早已是铁的现象，更何况随着网速越来越快，loading 的时间也越来越短。而遇到没有 loading 效果的网页时，我第一反应只会是怀疑而不是感到哇塞。TanStack Query 赋予了开发者掌控缓存的能力，同时也剥夺了有着轻量强迫症的用户对自己上网体验的掌控感。

    开发者通过“对不同 URL 区分不同的缓存策略”、搭配着“使用 isFetching 作为指示器提示给用户” 或许可以缓解这个问题，但这也提高了开发成本。

### 开发者层面
1. 使用 TanStack Query 对开发者有一点点门槛，这个门槛就是：需要开发者曾使用过 `useFetch`、`useRequest` 这类 hooks（composables），或者自己尝试过封装过一个简陋的 `useFetch`。综合讲其实就是需要开发者比较熟悉    Composition API。
  
    以我的眼光看，现今还有不少开发者没有达到这个门槛，他们仍在中大型项目中使用命令式的请求方式（即 fetch 或 axios ），不仅制造了大量冗余代码，而且每次都手动维护 `isLoading, isError, error, data` 等状态，降低了开发体验。
    
    命令式的优点是方便由用户行为来触发请求，毕竟永远只需要 "onClick=fetch" 就可以了，但这也就造就了初次体验 TanStack Query 的开发者不明白为什么要在 setup 中使用 useQuery, 也不太懂 useQuery 为什么会需要 enabled、immediate 这类参数。

2. TanStack Query 虽然提供了 [Dependent Queries
](https://tanstack.com/query/latest/docs/vue/guides/dependent-queries) 的文档，但假如要在 useQuery 之后接续其他非 useQuery 的任务时，最佳实践就要由开发者花费时间探索了（比如我就探索出一种奇怪且愚蠢的处理方式：`watchEffect( () => { if(isSuccess){...} })`）。

    一个很常见的需求就是在数据获取成功后展示一条 Toast 提示，一些人建议把相关逻辑放在 onSuccess 这个 callback 中。但 Tanstack Query 文档已将那几个 callback 参数标记为 deprecated, 并将在 V5 版本中移除，作者在[这里](https://github.com/TanStack/query/discussions/5279)表明了为什么，他建议的 new QueryClient 方式我还没有实践。
    
    另外：我看了作者的多篇文章，作为一个没有使用过 React 的人，发现 React 的 useEffect 好像造成了很多混乱，就连我认为的一个平均水平稍高的前端群组也经常有人讨论 useEffect 的最佳实践，我不太想在探讨 TanStack Query 的时候去学习 React 的基础知识。

3. 因为缓存数据是 readonly 的，开发者[不可以修改 cache 数据](https://github.com/TanStack/query/issues/4750)，所以会导致开发习惯的变化：
    
    假设网页需要渲染一个从 API 获取的 list，允许用户通过点击来折叠或展开内容。那么在没有使用 TanStack Query 之前，我可能直接就在获取 list 之后为每个 item 追加一个 collapsed 属性，并动态地修改它。在使用了 TanStack Query 之后，开发者则需要多维护一个 map 来表示 item 的折叠状态，开发体验有种降低的感觉。
4. useQuery 提供的返回值并不完全符合开发需求，比如我自己封装的 useFetch 就暴露了 i18n 之后的 errorMessage 出来，所以开发者可能还要基于 useQuery 二次封装。

5. 维护成本。我不认为 TanStack Query 会完全替代 useFetch，所以你的项目会可能会同时包含大量 useQuery 和大量 useFetch。

## 总结
个人感觉，在不想增加太多心智负担的情况下，TanStack Query 其实更适合偏现代、轻量、经过良好设计的应用。如果你不想折腾，可以暂时不用。毕竟，说不定在还未开始尝试之前，它就被淘汰了😅，毕竟作者似乎也因 RSC 的出现而透露出一股俏皮且随遇而安的恐慌，详见 [You Might Not Need React Query](https://tkdodo.eu/blog/you-might-not-need-react-query)。

当然现在我还是没有定论的，因为我还没有深度使用，本文仍会不断补充。