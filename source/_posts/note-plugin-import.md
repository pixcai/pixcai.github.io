---
title: 按需加载原理
date: 2018-09-26 16:37:12
tags: [javascript, babel]
---
使用**antd**之类的框架经常需要以下配置用于减少打包后的体积：
```js
{
  "plugins": [
    ["import", { "libraryName": "antd", "libraryDirectory": "es", "style": "css" }] // `style: true` 会加载 less 文件
  ]
}
```
这个插件的功能是利用**babel**解析后得到抽象语法树，收集导入库的语句，在编译时替换为导入指定文件。

以**antd**为例，以下代码做的就是根据`libraryName`，也就是**antd**，来判断，如果导入语句符合`import {Button} from 'antd'`的形式，即从**antd**导入了组件，那么就将这条语句加入到待替换列表：
```js
ImportDeclaration(path) {
  const { node } = path;

  // path maybe removed by prev instances.
  if (!node) return;

  const { value } = node.source;
  const libraryName = this.libraryName;
  const types = this.types;
  if (value === libraryName) {
    node.specifiers.forEach(spec => {
      if (types.isImportSpecifier(spec)) {
        this.specified[spec.local.name] = spec.imported.name;
      } else {
        this.libraryObjs[spec.local.name] = true;
      }
    });
    this.pathsToRemove.push(path);
  }
}
```
得到了要替换的语句，就可以根据插件中配置的选项进行替换。例如：
```js
import {Button} from 'antd'
```
根据插件配置解析后得到的文件路径为`antd/es/button`，其中`es`是插件中的`libraryDirectory`选项，指定组件所在的源文件目录。因此，上面的语句就变成了下面的形式：
```js
import Button from 'antd/es/button'
```
这样就不必导入整个库了。
```js
importMethod(methodName, file) {
  if (!this.selectedMethods[methodName]) {
    const libraryDirectory = this.libraryDirectory;
    const style = this.style;
    const transformedMethodName = this.camel2UnderlineComponentName  // eslint-disable-line
      ? camel2Underline(methodName)
      : this.camel2DashComponentName
        ? camel2Dash(methodName)
        : methodName;
    const path = winPath(
      this.customName ? this.customName(transformedMethodName) : join(this.libraryName, libraryDirectory, transformedMethodName, this.fileName) // eslint-disable-line
    );
    this.selectedMethods[methodName] = addDefault(file.path, path, { nameHint: methodName });
    if (style === true) {
      addSideEffect(file.path, `${path}/style`);
    } else if (style === 'css') {
      addSideEffect(file.path, `${path}/style/css`);
    } else if (typeof style === 'function') {
      const stylePath = style(path, file);
      if (stylePath) {
        addSideEffect(file.path, stylePath);
      }
    }
  }
  return Object.assign({}, this.selectedMethods[methodName]);
}
```
总结下来就一句话：按需加载不过是将`import {Button} from 'antd'`的形式转换为`import Button from 'antd/es/button'`的形式的过程。
