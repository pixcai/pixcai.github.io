---
title: 算法笔记：乘2加1
date: 2018-09-25 10:54:08
tags: [javascript, algorithm]
---
## 问题描述
两个数**A**和**B**，其中**A<B**。只能对**A**进行两种操作：**A=A+1** 或 **A=A*2**，求**A**到**B**的最少操作次数。

## 解决思路
先判断符号位是否相等，如果不等，就一直**A=A+1**。然后判断二进制长度是否相等，如果不等，表明数值相差较大，可以尝试**A=A*2**。如果长度相等，表明数值非常接近，直接**B-A**。

## 代码实现
```js
function minStep(A, B) {
  let step = 0
  let bA = A.toString(2)
  const bB = B.toString(2)

  while (bA[0] !== bB[0]) {
    step += 1
    A = A + 1
    bA = A.toString(2)
  }
  while (bA.length !== bB.length) {
    step += 1
    A = A << 1
    bA = A.toString(2)
  }

  return B - A + step
}
```
