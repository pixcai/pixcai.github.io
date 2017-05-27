---
title: requestAnimationFrame与setTimeout使用区别
date: 2017-05-27 10:45:19
---
虽然很多文章甚至官方宣称最好用`requestAnimationFrame`代替`setTimeout`来实现动画，但是在某些性能要求很高的情形下，还得酌情使用`requestAnimationFrame`。

`requestAnimationFrame`是尽可能快的更新，以使动画效果流畅，因此特别占用GPU和CPU。如果一个页面有多处动画效果并且持续动画，GPU和CPU会急剧上升。而`setTimeout`函数可以控制刷新的帧数，降低GPU负担，在适合的场景中使用效果反而比`requestAnimationFrame`好。

但*需要注意的是*，用`setTimeout`来更新动画，在浏览器繁忙的时候，`setTimeout`的调度会受到阻塞，本来设定帧率为60FPS，可能实际效果只有10FPS，极其不稳定。而且在浏览器最小化的状态下，`setTimeout`依然在执行，占用CPU，而`requestAnimationFrame`则不会。
