---
title: 我的JavaScript命名规范
date: 2017-04-25 23:55:32
---
## 变量
1、普通变量采用小驼峰命名法。
例如：`var isChar = false`、`var ok = true`

2、变量私有但可以暴露给外部访问，则用单个下划线。
例如：`var _status = false`

3、变量私有且只在特定范围内有效，强烈不建议直接访问，则用双下划线。
例如：`var __status = false`

4、与关键字或保留字或内置变量同名的变量用前缀符号`$`表示，并且`$`应该在下划线后。
例如：`var $class = null`、`var __$dirname = __dirname`

对于单下划线的私有变量，可以直接访问，但不能用于修改。双下划线的变量既不能直接访问更不能直接修改。
如果需要对私有变量进行修改，必须提供`getter/setter`函数。
例如：
```js
var __status = false
function setStatus(value) {
	__status = value
}
```

## 函数
### 一、参数
1、如果某个参数是必填的，那么它应该符合小驼峰命名法。
例如：`function clone(obj) {}`

2、如果某个参数是可选的，那么它应该用单个前缀下划线表示。
例如：`function clone(obj, _isDeep) {}`

3、如果某个参数需要使用`new`关键字构造，则第一个字母应该大写。
例如：`function extend(BaseClass, properties) {}`

### 二、函数名
1、普通函数名应该符合小驼峰命名法，如果函数应该用`new`关键字构造，那么它的首字母应该大写。
例如：`function clone(obj) {}`、`function BaseClass() {}`

2、私有函数命名必须以双下划线开始同时以双下划线结束，并且私有函数不存在单下划线的情况。
例如：`function __find__() {}`是正确的，`function _find_() {}`是错误的

## 代码例子
```js
function extend(BaseClass, properties) {
	var $class = function () {
		if (Object.prototype.toString.call(this.__init__) === '[object Function]') {
			this.__init__.apply(this, arguments)
		}
	}

	function __BuiltinClass__() {}

	if (BaseClass) {
		__BuiltinClass__.prototype = BaseClass.prototype
	}
	var __proto__ = $class.prototype = new __BuiltinClass__()

	for (var key in properties) {
		if (properties.hasOwnProperty(key)) {
			__proto__[key] = properties[key]
		}
	}
	__proto__.constructor = $class

	return $class
}

function Shape(_options) {
	this.options = _options
}

Shape.prototype.getType = function () {
	return this.__type
}

var Rect = extend(Shape, {
	__init__: function (x, y, width, height) {
		this.set({
			x: x, y: y, 
			width: width, height: height
		})
	}, 
	set: function (properties) {
		for (var key in properties) {
			if (properties.hasOwnProperty(key)) {
				this[key] = properties[key]
			}
		}
	}, 
	__type: 'rect'
})

var rect = new Rect(0, 0, 300, 150)
console.log(rect.x, rect.y, rect.width, rect.height, rect.getType())
```