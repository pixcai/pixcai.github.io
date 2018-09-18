---
title: npm模块学习：p-map
date: 2018-09-13 15:35:51
tags: [javascript, node, npm]
---
## 介绍
**p-map**的功能是用来对一组`Promise`对象进行操作，当其中任何一个`Promise`的状态变为`fulfilled`时，执行我们传递的回调函数，只要有一个`Promise`状态变为`rejected`，所有未执行的`Promise`都不再执行。

项目地址：https://github.com/sindresorhus/p-map

## 源码分析
**p-map**的代码并不复杂，以下是`p-map`的核心代码：
```js
const next = () => {
  // 如果当前Promise状态为rejected，就不再执行下一个
  if (isRejected) {
    return;
  }

  const nextItem = iterator.next();
  const i = currentIndex;
  currentIndex++;

  if (nextItem.done) {
    isIterableDone = true;
    // 如果全部执行成功，则resolve
    if (resolvingCount === 0) {
      resolve(ret);
    }

    return;
  }

  resolvingCount++;

  Promise.resolve(nextItem.value)
    // 对结果进行处理
    .then(element => mapper(element, i))
    .then(
      value => {
        ret[i] = value;
        resolvingCount--;
        next();
      },
      error => {
        isRejected = true;
        reject(error);
      }
    );
};
```
以上逻辑比较清晰，其中最大的亮点就是使用迭代器判断是否结束。大部分人遇到**p-map**这种需求，都会想到用`Promise.all`和`Array.prototype.map`之类的方法去实现效果。例如：
```js
function pMap(promises, mapper) {
  return Promise.all(promises.map((item, i) => {
    return Promise.resolve(item).then(data => mapper(data, i))
  }))
}
```
这段代码和**p-map**库的实现差别在于：即使有一个`Promise`变为`rejected`，后面的`Promise`仍然会执行，所以实际上并不能满足需求。

而**p-map**中迭代器的使用不仅大大简化了实现难度，还提供了额外的控制机会，`concurrency`选项能发挥作用，也是使用迭代器带来的好处。
