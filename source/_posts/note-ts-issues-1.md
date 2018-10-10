---
title: TypeScript相关问题(一)
date: 2018-10-10 11:05:29
tags: typescript
---
### 1. 当代码中导入了第三方库的时候，直接编译文件报错。例如以下目录：
```bash
.
├── demo.ts
├── node_modules
├── package.json
├── tsconfig.json
└── yarn.lock
```
其中`demo.ts`的代码如下：
```js
import { of } from 'rxjs'

of(1, 2, 3).subscribe(x => console.log(x))
```
用以下方式运行：
```bash
pixcai@sierra:~/demo|⇒ ./node_modules/.bin/tsc demo.ts
node_modules/rxjs/internal/Observable.d.ts(82,59): error TS2693: 'Promise' only refers to a type,but is being used as a value here.
node_modules/rxjs/internal/types.d.ts(35,84): error TS2304: Cannot find name 'Iterable'.
node_modules/rxjs/internal/types.d.ts(39,6): error TS2304: Cannot find name 'Symbol'.
```
如果**TypeScript**装在全局，即`npx tsc demo.ts`，也是一样。

**问题原因**：`tsconfig.json`没生效。

**解决办法**：`npx tsc -p tsconfig.json`或`npx tsc -p .`或`npx tsc --lib es2015,dom demo.ts`。

### 2. `document.getElementById()`、`document.querySelector()`等函数返回的是`Element`类型，如果要取某些属性，比如：
```js
const target = document.querySelector('.target')
target.style.top = '100px'
```
则会报错：
```bash
error TS2339: Property 'style' does not exist on type 'Element'.
```

**解决办法**：转换为`HTMLElement`类型，`const target = <HTMLElement>document.querySelector('.target')`。
