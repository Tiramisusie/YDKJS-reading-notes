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

