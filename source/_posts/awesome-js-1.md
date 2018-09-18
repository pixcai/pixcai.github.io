---
title: JavaScript奇技淫巧(一)
date: 2018-09-18 14:40:27
tags: [javascript, node]
---
### 判断是否是数字
```js
function isNumber(num) {
  if (typeof num === 'number') {
    return num - num === 0;
  }
  if (typeof num === 'string' && num.trim() !== '') {
    return Number.isFinite ? Number.isFinite(+num) : isFinite(+num);
  }
  return false;
}
```
判断`num - num === 0`的原因：
1. `typeof NaN === 'number'`等于`true`、`typeof Infinity === 'number'`等于`true`。
2. `NaN - NaN`等于`NaN`、`Infinity - Infinity`等于`NaN`。
