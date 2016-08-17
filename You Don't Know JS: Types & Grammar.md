# You Don't Know JS: Types & Grammar

# Chapter 1: Types

javascript有七中内建的类型：

`null`, `undefinded`, `boolean`, `number`, `string`, `object`, `symbol`(es6中新增)

除了`object`，其他的类型都叫做‘原始类型’。

可以用‘typeof’操作符检查某个值的类型。返回值是代表该值的类型的字符串，例如

```
typeof '12'   === 'string'
```

要注意的是 `typeof null === 'object'` 

如果有以下代码
```js
typeof a; // 'undefinded'
```

则`a`会是以下两种情况中的一种：

- `a`还没定义；
- `a`已经定义，但没有赋值，或这赋值为`undefinded`

在引用一个不确定是否已定义的变量时，`typeof`的这种行为会很有用。例如以下代码：

```js
if(typeof foo !== 'undefinded'){
	// do smething with foo...
}
```

如果像下面这样写，当`bar`不存在的时候会报错：

```js
if (bar) {
	// do something with bar...
}
```

# Chapter 2: Values

## Number

当使用数字字面量来调用`Number`的内建方法时，如`42.toFixed(2)`，要注意`.`操作符会优先被解析成数字中的小数点，而不是方法的调用符号。

```js
// invalid syntax:
42.toFixed( 3 );	// SyntaxError

// these are all valid:
(42).toFixed( 3 );	// "42.000"
0.42.toFixed( 3 );	// "0.420"
42..toFixed( 3 );	// "42.000"
```

### Small Decimal Values

二进制浮点数（javasctipt中的`number`）的一个副作用是‘不够准确’，例如：

```js
0.1 + 0.2 === 0.3 // false
```

出现这种情况的原因是，`0.1`在二进制浮点数的表示形式中并不完全等于`0.1`，`0.2`也有类似的情况！所以它们相加的结果并不是`0.3`，而是非常接近`0.30000000000000004`的一个数。

### The Not Number, Number

javascript有一个非常特别的数字叫`NaN`，字面的意思是`not a number`，但其实更叫准确的说法应该是‘非法数字’。因为`typeof NaN === 'number'`！`NaN`不等于任何一个数字，或者更准确的说，`NaN`是javascript中唯一一个不等于自身的值，所以会有：`NaN !== NaN`。

检测`NaN`的一个比较简单的方式是用`isNaN(..)`。但这个方法并不靠谱，因为它只会检测传进去的值是否是数字。例如

```js
isNaN(2/'fo'); // true
isNaN('fo')	   // true
```

很明显以上检测结果与`NaN`的正确定义不符。在es6中新增的`Number.isNaN(..)`方法可以正确检测`NaN`。同时，可以利用以下polyfill。

```js
if (!Number.isNaN) {
	Number.isNaN = function(value) {
		return typeof value === 'number' && window.isNaN(value)
	}
}

// another way 
if (!Number.isNaN) {
	Number.isNaN = function(value) {
		return value !== value
	}
}
```

### Value vs. Reference

javascript中有值传递和引用传递。对于`a = b`，如果是值传递，当`b`的值改变时，不会影响`a`的值；如果是引用传递，则会同时影响`a`的值。值的类型决定了是哪一种传递。

- `null`,`string`,`undefinded`,`number`,`boolean`,`symbol`是*值*传递；
- `object`包括`array`和`function`是*引用*传递

在赋值和参数传递的时候要尤为注意是哪一种传递，对于值的改变是否是你所期待的。