# Chapter 3: Objects

## Type

JS 中有六种数据类型：

- `string`
- `number`
- `boolean`
- `null`
- `undefined`
- `object`

要注意的是，虽然`typeof null`的值是`'object'`，但`null`是属于自己原始类型，而不是`'object'`。这是 JS 语言本身的一个 bug 。

## Contents

在对象中，属性名**总是**字符串。如果使用来非字符串的值来作为属性名，则会先转换成字符串，即使是数字。这种情况在数组中很常见（数组的索引）。

```js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

### Array

数组也是对象，所以可以像对象一样使用`.`或者`[ ]`来添加属性，但这样添加属性后不会改变数组的`length`的值。

在向数组中添加属性的时候，如果属性名是数字的字符串，如`'123'、'2'`，这个属性名会变成数组的索引，最终改变数组的长度：

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length;	// 4

myArray[3];		// "baz"
```

### Property Descriptors

在 ES5 中，所有的属性都可以用 **property descriptor**来描述。每个属性除了有自己的值，还会有三个特征： `writable`, `enumerable`,和 `configurable`。可以用`Object.defineProperty(..)`来添加或修改一个属性。

#### Writable

`writable`决定一个属性的值能否被修改。

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // not writable!
	configurable: true,
	enumerable: true
} );

myObject.a = 3;

myObject.a; // 2
```

如果在严格模式下，上面的代码在尝试修改`a`的值的时候会报错`TypeError`

#### Configurable

`configurableb`表示能否修改该对象的 descriptor 。一旦`configurableb`被设置为`false`，就不能通过任何方式去修改它的 descriptor。如果尝试修改，就会报错。同时，使用`delete`来移除一个属性的时候也会失效。

#### Enumerable

`enumerable`表示一个对象的属性能否被枚举，例如在`for..in`循环中。

### `[[Get]]`

当需要获取一个对象的某个属性值的时候，就会在这个对象中执行`[[Get]]`操作。如果该对象没有这个属性，就会在原型链中继续查找。如果到最后都没有找到，就会返回`undefined`

### `[[Put]]`

相应的，会有一个`[[Put]]`操作，表明意思就是设置该对象的某个属性。但在设置之前还要检测这个属性是否存在。如果存在，这个属性是否可写。还会有其他的判断。

### Existence

要检测一个对象中是否有某个属性，可以用`in`操作符或者`hasOwnProperty(..)`方法。

`obj.hasOwnProperty('a')`方法只会检测`obj`本身是否有`a`这个属性，不会在原型链中查找。但`in`操作符会在`obj`中没有`a`属性的时候继续在原型链中查找。

要注意，`in`操作符是对**属性名**的查找，而不是**属性值**的查找。所以`4 in [2,4,6]`这个操作会返回`false`。



