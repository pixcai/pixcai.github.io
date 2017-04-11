---
title: Node.js基础入门与操作
date: 2017-04-08 04:07:37
---
### 什么是`Node`项目
一个`Node`项目根目录下，一般都存在一个`package.json`文件，它定义了项目的依赖模块和编译、运行、测试相关的命令以及其它的配置信息，比如项目版本、仓库地址、许可证等等。`npm install`命令根据这个文件，自动下载所需的依赖模块，也就是配置项目的运行和开发环境。其中`package.json`文件可以手动创建，例如`touch package.json`，或者通过`npm init`命令交互式创建。一个最简单的`package.json`文件示例如下，其中只定义了两项配置：项目名称和项目版本。
```json
{
  name: "app", 
  version: "1.0.0"
}
```
对于一个完整的项目而言，`package.json`包含的信息很多。以下是一个小型项目的`package.json`：
```json
{
  "name": "geojson-map",
  "version": "1.0.0",
  "description": "Draw GeoJSON map data to Canvas",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "start": "python -m SimpleHTTPServer"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/pixcai/geojson-map.git"
  },
  "keywords": [
    "Canvas",
    "GeoJSON"
  ],
  "author": "pixcai",
  "license": "GPL-3.0",
  "bugs": {
    "url": "https://github.com/pixcai/geojson-map/issues"
  },
  "homepage": "https://github.com/pixcai/geojson-map#readme",
  "devDependencies": {
    "webpack": "^1.14.0"
  }
}
```
按照这个例子，下面详细解释每个字段所代表的含义。