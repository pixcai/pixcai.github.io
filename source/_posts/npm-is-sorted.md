---
title: npm模块学习：is-sorted
date: 2018-09-17 10:17:00
tags: [javascript, node, npm]
---
### 介绍
**is-sorted**用来检查数字数组中的元素是否按升序或降序排序。

项目地址：https://github.com/dcousens/is-sorted

### 源码分析
代码逻辑比较简单，通过遍历数组，判断当前元素与前一个元素的差值大于零或小于零，从而得出元素是否顺序排列：
```js
for (var i = 1, length = array.length; i < length; ++i) {
  if (comparator(array[i - 1], array[i]) > 0) return false
}
```
`comparator`是一个比较函数，默认是按升序排列，也可以手动指定。
```js
function defaultComparator (a, b) {
  return a - b
}
```

### 改进
作者直接将比较函数的结果与零比较，将范围限制的比较窄。如果将比较操作放入比较函数内部，那我们只需要：
```js
function defaultComparator(a, b) {
  return a - b > 0
}
// 然后
if (!comparator(array[i - 1], array[i])) return false
```
这样写法就更灵活了，不仅能判断升序还是降序，还能判断是否按某一固定数值升降，例如：
```js
function customComparator(a, b) {
  return a - b > 2
}
```
