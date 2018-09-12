---
title: 我的JavaScript命名规范
date: 2017-04-25 23:55:32
tags: javascript
---
## 变量
1、普通变量采用小驼峰命名法。
例如：`var isChar = false`、`var ok = true`

2、变量私有但可以暴露给外部访问，则用单个下划线。
例如：`var _status = false`

3、变量私有且只在特定范围内有效，强烈不建议直接访问，则用双下划线。
例如：`var __status = false`
特殊情况下，如果变量名前后加双下划线有一定意义，那么可以用前后双下划线形式。
例如：
```js
var obj = {}
var __proto__ = obj.__proto__    // 此处变量前后加双下划线有特殊意义
```

4、与关键字或保留字或内置变量同名的变量用前缀符号`$`表示，并且`$`应该在下划线后。
例如：`var $class = null`、`var __$dirname = __dirname`

对于单下划线的私有变量，可以直接访问，但不能用于修改。双下划线的变量既不能直接访问更不能直接修改。
如果需要对私有变量进行修改，必须提供`getter/setter`函数。
例如：
```js
function A() {
	this.__status = false
	// 提供Setter/Getter函数
	this.setStatus = function (value) {
		this.__status = value
	}
	this.getStatus = function () {
		return this.__status
	}
}
```

## 函数
在函数主体为空或者只有一条语句的情况下，函数可以写作一行，其它任何情况下函数都必须写成多行。

### 一、参数
1、如果某个参数是必填的，那么它应该符合小驼峰命名法。
例如：`function clone(obj) {}`
如果参数是引用类型，除非明确知道修改所带来的影响和结果，否则在对引用类型进行更改时，必须进行深复制，以免不当的操作所带来的结果向后传递，使错误源难以定位。

2、如果某个参数是可选的，那么它应该用单个前缀下划线表示。
例如：`function clone(obj, _isDeep) {}`

3、如果某个参数需要使用`new`关键字构造，则第一个字母应该大写。
例如：`function extend(BaseClass, properties) {}`

### 二、函数名
1、普通函数名应该符合小驼峰命名法，如果函数应该用`new`关键字构造，那么它的首字母应该大写。
例如：`function clone(obj) {}`、`function BaseClass() {}`

2、私有函数命名必须以双下划线开始同时以双下划线结束，并且私有函数不存在单下划线的情况。
例如：`function __find__() {}`是正确的，`function _find_() {}`是错误的。

### 三、参数检查规范
对于需要传参的函数调用，除非参数符合规范命名而且唯一，例如大写开头的参数需要使用`new`构造、单下划线开头的参数是可选的。除此之外，其它类型不明确的参数，例如数字、字符串、对象都是小驼峰命名，在使用参数前，必须对参数进行类型检查。这个类型检查只针对需要使用到的参数及其值，那些传参了但未使用的参数可以忽略。如果`B`函数在`A`函数内部被调用，那么当`B`函数用到`A`的参数并且`A`也用到这个参数，则`B`不需再进行类型检查，`A`必须保证自己使用的参数传给`B`函数时依然是正确的。
例如：
```js
function B(name, value) {
	this.name = name
	if (typeof value !== 'number') {
		value = 0
	}
	this.value = value
}

function A(name) {
	if (!(typeof name === 'string' && name !== '')) {
		name = 'A'
	}
	this.name = name
	this.b = new B(name, '123')
}
```

## 语句
1、声明语句
声明语句之后必须留一空行。

2、`while`和`if`语句
如果主体只有一条语句，关键字和内容应该放在同一行，并且省略花括号。如果有多条语句，应该用花括号包裹并且以多行显示。
在单行显示的情况下，应该始终与上条语句之间留一空行，如果在它下面的语句是声明语句，那么它与下面的语句之间也应该留一空行。如果它下面的语句不是声明语句，则空行可留可不留。如果单行的长度超过**80**个字符，那么可以选择换行，但换行后主体必须被花括号包裹。
例如：
```js
// 错误
isOk = false

if (isOk) callback()
var result = 0        // 声明语句之后没有留空行
if (result)           // if换行不能省略花括号
	isOk = true

// 正确
isOk = false

if (isOk) callback()

var result = 0

if (result) {
	isOk = true
}
```
3、`for`语句
任何情况下花括号都不省略，即使主体只有一条语句。

4、`switch`语句
主体内的`case`与`switch`关键字保持一定缩进，如果`case`的主体只有一条语句，可以写成单行形式。`default`条件是必须的，不可省略。

## 代码例子
```js
function extend(_BaseClass, _properties) {
	var $class = function () {
		if (Object.prototype.toString.call(this.__init__) === '[object Function]') {
			this.__init__.apply(this, arguments)
		}
	}

	function BuiltinClass() {}

	if (_BaseClass) {
		BuiltinClass.prototype = _BaseClass.prototype
	}
	var __proto__ = $class.prototype = new BuiltinClass()

	if (Object.prototype.toString.call(_properties) === '[object Object]') {
		for (var key in _properties) {
			if (_properties.hasOwnProperty(key)) {
				__proto__[key] = _properties[key]
			}
		}
	}
	__proto__.constructor = $class

	return $class
}

function Shape(_options) {
	this.options = _options ? _options : {}
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
		if (Object.prototype.toString.call(properties) === '[object Object]') {
			for (var key in properties) {
				if (properties.hasOwnProperty(key)) {
					this[key] = properties[key]
				}
			}
		}
	}, 
	__type: 'rect'
})

var rect = new Rect(0, 0, 300, 150)
console.log(rect.x, rect.y, rect.width, rect.height, rect.getType())
```
