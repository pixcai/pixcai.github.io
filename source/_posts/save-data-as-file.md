---
title: 保存数据并下载文件
date: 2019-05-07 09:52:00
tags: [javascript, html]
---
方法实现：
```js
function downloadAsFile(data, filename) {
  const href = window.URL.createObjectURL(new Blob([data]));
  const downloadElement = document.createElement('a');

  downloadElement.href = href;
  downloadElement.download = filename;
  downloadElement.click();
  window.URL.revokeObjectURL(href);
}
```