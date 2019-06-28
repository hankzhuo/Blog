# 第一篇：什么是 Redux

第一次看 redux 文档时候，说实话我是看的比较抽象的，比如 `Action、 dipatch、Reducer、Store` 等概念，对于刚入 React 全家桶，这些概念会让人觉得有点绕。

但它太棒了，它是 React 全家桶里很重要的组成部分，如果你项目中使用 React 技术栈（当然 React 与 Redux 不是绑定关系），那么 Redux 思想是必须要了解下，当你使用过一段时间时候，真的会被其思想感染，只能说这些库的作者是真的强，给他们点赞。

偶然间，在国外技术论坛看到一些文章，把 Redux 解释的非常通俗易懂，把 Redux 与现实中例子结合起来，构成了非常清晰的、完整的一套逻辑。

## Redux 是什么
为啥要用到 Redux？因为在一个 app 项目中，如果涉及到 `state` 复杂交互，在 React 组件内部对于 `state` 的管理是不方便，不便于维护，使用统一管理 `state` 的库是比较合适的。当然，如果你的项目比较简单，完全可以不需要 redux。

那么 Redux 是什么？官方文档解释：
> Redux is a predictable state container for JavaScript apps.

如果你了解过 React，那么肯定知道 `state` 概念，那么 Redux 可以理解为 **state 的外部状态管理容器**，是不是还很有点意思了？接下来我会给你一步步解释每个概念。把 Redux 与 “去银行取钱的过程”结合起来，更好地理解它们之间的关系。

![](http://a3.qpic.cn/psb?/V144SrtM47BnfG/tPO5oB4T5Xm4Qbs7bFwxMyHG1owPzwBsG*J85JNIHj8!/b/dFYAAAAAAAAA&ek=1&kp=1&pt=0&bo=KwGoAAAAAAARF6I!&tl=3&vuin=477615869&tm=1533045600&sce=60-4-3&rf=viewer_4)

## 解释 API 

首先，你的钱都储存在银行金库里，银行保存着你所有的钱，相对应的，把银行金库比作 Redux 里的 `Store`，而 `Store` 储存着你 app 中的所有 `State`。

先记住这个对应关系：
> 金库 <==> `Store`

> 你的钱  <==> `State`

现在，你要去银行取钱，这个想法或者目的，相对应的，Redux 里的`Action` 。

> 取钱想法 <==> `Action`

去取钱，还是存钱？这个对应的 Redux 里是 `Action` 里的 `Type`：

> 取钱还是存钱 <==> `Type`

`action` 形式简单表示如下：
`
{
  type: "WITHDRAW_MONEY", 
  amount: "￥1,000"
}
`
`
把卡给收银员或者 ATM 取钱，然后把钱给你，相对应的，就是 Redux 里的，发起一个 `action` ，传递给 `Reducer` 返回一个新的 `State`。

> 收银员 / ATM  <==>  `Reducer`

## 总结：

本篇内容主要讲 Redux 的理论，把 Redux 比喻成去银行取钱，能更好理解 Redux。

Redux 与实际的对应关系：

1. 金库或银行 <==> `Store`
2. 你的钱  <==> `State`
3. 取钱的想法或目的 <==>`Action`
4. 取钱还是存钱 <==> `Type`
5. 收银员 / ATM  <==>  `Reducer`

流程：
去银行取钱，把卡给收银员(或 ATM 机)，取多少钱，然后收银员把钱取出来给你的过程。

在 app 中，更新 `state` 的过程：`dipatch` 一个 `action` 给 `reducer`（收银员），`reducer`（收营员） 返回一个新的 `state`（你银行还剩下的钱） 。

![](http://m.qpic.cn/psb?/V144SrtM47BnfG/omd3zgmZVOQlwd5eTFk.3Xy20stEiLEaSs4Ir1FYzVw!/b/dFcAAAAAAAAA&bo=9QLtAgAAAAADFyo!&rf=viewer_4)

## 下一篇

从 React 中内部更新 state 到 Redux 获取 state。，快开启 Redux 的知识之旅吧。

[下一篇]()