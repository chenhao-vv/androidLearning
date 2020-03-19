---
title: RecyclerView的几种刷新方法
tags: 新建,模板,小书匠
renderNumberedHeading: true
grammar_cjkRuby: true
---

对于RecyclerView的Item Animator，有一个常见的坑就是“闪屏问题”。
这个问题的描述是：当Item视图中有图片和文字，当更新文字并调用notifyItemChanged()时，文字改变的同时图片会闪一下。这个问题的原因是当调用notifyItemChanged()时，会调用DefaultItemAnimator的animateChangeImpl()执行change动画，该动画会使得Item的透明度从0变为1，从而造成闪屏。

解决办法很简单，在rv.setAdapter()之前调用((SimpleItemAnimator)rv.getItemAnimator()).setSupportsChangeAnimations(false)禁用change动画。



参考：
[几种刷新方法，代码展示](https://www.cnblogs.com/baiqiantao/p/6956425.html)
[局部刷新闪动原因](https://wetest.qq.com/lab/view/176.html?from=content_zhihuzhuanlan)