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

## Array

### Array-Likes

很多时候，需要把一个‘类数组’（一个用数字来索引的值的集合，例如函数中的`arguments`对象）转换成一个‘真数组’，然后使用数组特有的一些操作。一个常用的方法是借用`slice(..)`方法来实现转换。

```js
function foo() {
	var arr = Array.prototype.slice.call( arguments );
	arr.push( "bam" );
	console.log( arr );
}

foo( "bar", "baz" ); // ["bar","baz","bam"]
```

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

## Chapter 3: Natives

当使用`a = new String('str')`的方式来创建字符串的时候，对`a`使用`typeof`操作符的时候会返回‘object’而不是'string'。以下的内建对象也会有类似的情况出现。

- Number()
- Boolean()
- Array()
- Object()
- Function()
- RegExp()
- Date()
- Error()
- Symbol()

原因是用这种‘构造函数’的方式来新建值的时候，会返回一个对象，这个对象包裹着具体的值。

对于这种`typeof a === 'object'`的情况，可以借用`Object`的`toString(..)`方法来检测值的类型。

```js
Object.prototype.toString.call(a)  // '[object String]'
```

### Boxing Wrappers

js中的原始值都是没有属性或者方法的，为了能有这样`a.length`的用法，js引擎会自动在值的外面包一层对象。

### Natives as Constructors

#### `Array(..)`

不要在任何情况下尝试新建一个稀疏数组，因为它会有很多意想不到的行为。

避免以下行为：

- 显式修改数组的`.length`属性，并且新的长度比数组实际的长度要长；
- 用`delete`操作符删除一个数组的值

#### `RegExp(..)`

如果不需要动态创建正则表达式对象，都不要用`new RegExp(..)`的形式。

```js
// 动态创建
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );
var matches = someText.match( namePattern );
```

## Chapter 4: Coercion

### JSON Stringification

可以使用`JSON.stringify(..)`把一些值转换成兼容json的字符串值。但当它遇到`undefined`, `function`, `symbol`这些值的时候会自动忽略。如果数组中包含这样的值，则对应的值会变成`null`，如果是在对象中遇到，则会忽略。

```js
JSON.stringify( undefined );                    // undefined
JSON.stringify( function(){} );                 // undefined
JSON.stringify( [1,undefined,function(){},4] ); // "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } );      // "{"a":2}"
```

### Falsy Values

以下值可以通过类型转换变成`false`：

- undefined
- null
- false
- +0, -0, and NaN
- ""

不在这个列表上的可以认为都是‘true’

## Explicit Coercion

### Explicitly: Strings <--> Numbers

字符串和数字之间的转换可以使用以下方法来完成。

```js
var a = 42;
var b = String( a );

var c = "3.14";
var d = Number( c );

b; // "42"
d; // 3.14
// 其他方法
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```


#### `Date` To `number`

可以使用一元操作符`+`把日期转换成数字，得到的结果是这个日期对应的unix时间戳，最常用的一个例子是获取当前时间的时间戳。

```js
var timestamp = +new Date();
// 效果相当于 
var timestamp = Date.now();		// 在ES5中引入
// 或者
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();
```

#### The Curious Case of the `~`

`~`操作符的作用简单来说就是`-(x+1)`，但实际上是在二进制位上的操作，而不仅仅是数学层面的操作。例如。

```js
~2; // -3
~-1 // 0
```

与`indexOf(..)`结合后会有以下用法

```js
var a = "Hello World";

~a.indexOf( "lo" );			// -4   <-- truthy!

if (~a.indexOf( "lo" )) {	// true
	// found it!
}

~a.indexOf( "ol" );			// 0    <-- falsy!
!~a.indexOf( "ol" );		// true

if (!~a.indexOf( "ol" )) {	// true
	// not found!
}
```

### Operators `||` and `&&`

在javascript中，`||`和`&&`的操作结果不是返回`true`或者`false`，而是返回两个操作数的结果中的一个。例如：

```js
var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
```

### Loose equality

当两个值使用`==`进行比较时，会先进行类型转换，转换的规则如下：

```js
x == y;
```

> 2. If x is null and y is undefined, return true.
> 3. If x is undefined and y is null, return true.
> 4. If Type(x) is Number and Type(y) is String,
>    return the result of the comparison x == ToNumber(y).
> 5. If Type(x) is String and Type(y) is Number,
>    return the result of the comparison ToNumber(x) == y.
> 6. If Type(x) is Boolean,
>    return the result of the comparison ToNumber(x) == y.
> 7. If Type(y) is Boolean,
>    return the result of the comparison x == ToNumber(y).
> 8. If Type(x) is either String or Number and Type(y) is Object,
>    return the result of the comparison x == ToPrimitive(y).
> 9. If Type(x) is Object and Type(y) is either String or Number,
>    return the result of the comparison ToPrimitive(x) == y.

**注意** 在`==`比较中，`null`只有和`undefinded`比较才会返回`true`，其他情况都会返回`false`。

一些特别的情况：

```js
"0" == null;			// false
"0" == undefined;		// false
"0" == false;			// true -- UH OH!
"0" == NaN;				// false
"0" == 0;				// true
"0" == "";				// false

false == null;			// false
false == undefined;		// false
false == NaN;			// false
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
false == {};			// false

"" == null;				// false
"" == undefined;		// false
"" == NaN;				// false
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
"" == {};				// false

0 == null;				// false
0 == undefined;			// false
0 == NaN;				// false
0 == [];				// true -- UH OH!
0 == {};				// false

[] == ![]    //true

2 == [2];		// true
"" == [null];	// true
```

## Chapter 5: Grammar