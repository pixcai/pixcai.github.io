---
title: npm模块学习：p-map
date: 2018-09-13 15:35:51
tags: [javascript, node, npm]
---
## 描述

之前看到[敖天羽的博客](https://codesky.me/archives/javascript-promise-foreach.wind)遇到这种需求，他因此写了一个和`p-map`功能相似的库，代码只有几行：
```js
module.exports = (promises, callback, errorHandler) => {
  return Promise.all(promises.map(promise => {
    return promise
      .then(result => {
        callback(result)
      })
      .catch(err => {
        errorHandler(err)
      })
  }))
}
```
这几行代码有几点不足：

1、`Promise.all`总是会**resolved**，对其`catch`总是无效的，例如`foreach(promises, callback, errHandler).catch(catchHandler)`这里的`catchHandler`永远不会被执行，如果无意中在`catchHandler`中处理某些逻辑，很难发现这些逻辑为什么没有执行。

2、不能在`then`回调中接受结果，例如`foreach(promises, callback, errHandler).then(result => console.log(result))`永远输出`[undefined, ...]`。

对于特定需求而言，上面一段代码完全能达到要求，以上两点纯属多余，但是作为一个库来说，最重要的是提供一个通用解决方案，并且最好不违背已经存在的一些常识，例如上面`catch`和`then`的问题，按照`Promise.all`的用法去用，难免就会陷入一些坑。

## 实现

以下是`p-map`的核心代码：
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
`p-map`的巧妙在于：提供了处理单个`Promise`成功回调的时机，但是整体又兼容`Promise.all`。我们可以像`Promise.all`函数一样去用它，如：`pMap(promises, mapper).then(callback).catch(errorHandler)`，也可以用`await`一次获取全部结果`const result = await pMap(promises, mapper)`。

此外，`p-map`还提供了一个选项：`concurrency`，配置一次性可以跑几个`Promise`(当然不是真正的并发，只是同时把这几个加入任务队列而已)，对于大量`Promise`的时候还是很有用的。

## 改进
`p-map`存在的一个小问题是：如果数组中总共15个`Promise`，一开始设置跑5个，但是失败了2个，后面就会变成3个3个的跑了。最好在失败后能重新恢复5个5个的跑。