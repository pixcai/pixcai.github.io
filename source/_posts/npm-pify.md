---
title: npm模块学习：pify
date: 2018-09-14 15:11:20
tags: [javascript, node, npm]
---
将原本不支持`Promise`调用的函数转化成`Promise`风格，最简单的方法可以这么写：
```js
function promisify(f) {
  const util = require('util')
  if (util.promisify) {
    return util.promisify(f)
  } else {
    return function () {
      return new Promise((resolve, reject) => {
        f.apply(this, [].slice.call(arguments).concat((err, val) => {
          err ? reject(err) : resolve(val)
        }))
      })
    }
  }
}
```
如果要支持更多功能，例如将模块内所有的函数promisify、接收多个返回值，就要用到**pify**提供的功能。