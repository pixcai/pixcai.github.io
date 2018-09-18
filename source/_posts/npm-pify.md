---
title: npm模块学习：pify
date: 2018-09-14 15:11:20
tags: [javascript, node, npm]
---
## 介绍
**pify**的功能和**util.promisify**一样，用来将一个函数包装为支持`Promise`的形式。

项目地址：https://github.com/sindresorhus/pify

## 源码分析
将原本不支持`Promise`调用的函数转化成`Promise`风格，最简单的方法可以这么写：
```js
function promisify(f) {
  return function () {
    return new Promise((resolve, reject) => {
      f.apply(this, [].slice.call(arguments).concat((err, val) => {
        return err ? reject(err) : resolve(val)
      }))
    })
  }
}
```
因为对于支持错误优先的回调函数，总是这种写法，例如：
```js
fs.readFile('package.json', 'utf-8', (err, data) => {
  if (err) {
    // 处理错误
    return
  }
  // 处理数据
})
```
回调函数总是作为最后一个参数，所以上面我们才可以：
```js
[].slice.call(arguments).concat((err, val) => {
  return err ? reject(err) : resolve(val)
})
```
把`Promise`的处理逻辑追加到参数列表的末尾。

**pify**库的核心代码和上面基本相同，但提供了更多的处理。例如选项：`exclude: [/.+(Sync|Stream)$/]`排除了以`Sync`和`Stream`结尾的函数名，避免了我们错误的对函数使用promisify。

除了对函数名处理外，**pify**还提供对整个模块内的函数进行promisify：
```js
for (const key in input) { // eslint-disable-line guard-for-in
  const property = input[key];
  ret[key] = typeof property === 'function' && filter(key) ? processFn(property, options) : property;
}
```
其中`filter`函数过滤掉符合特定模式的函数名，使用者可以在选项中配置。

另外，Node自带了`util.promisify`函数，我们不用手动去写，但是`util.promisify`不能对整个模块内的函数`promisify`。
