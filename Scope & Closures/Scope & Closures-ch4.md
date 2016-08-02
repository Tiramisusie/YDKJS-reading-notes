# Chapter 4: Hoisting

## Chicken Or The Egg?

我们都知道 JavaScript 有「变量提升」的特性。但是这个特性的内在原因是什么呢？用以下程序来做说明。

```javascript
console.log(a);  //undefined
var a = 2;
```

JavaScript 引擎会把`var a = 2;`分成两个语句：`var a;`（声明），`a = 2;`（赋值）。所有的声明语句，包括变量和函数声明，都会在编译阶段被处理；而赋值语句却会等到执行阶段才会处理。于是，上面的程序在处理过程中实际上会变成以下形式：

```javascript
var a;
console.log(a);
a = 2;
```

以上代码处理的过程看起来就像是把变量声明从原来的地方「移动」到了代码的顶部，所以叫做「变量提升」。要注意，变量和函数的声明都会「提升」。

## Functions First

变量和函数的声明都会被提升，但是函数的提升会优先。就像下面：

```javascript
foo(); // 1

var foo;

function foo() {
	console.log( 1 );
}

foo = function() {
	console.log( 2 );
};
```

上面的例子中的`var foo;`事实上是多余的，因为函数声明`function foo(){..}`会提升到代码的顶部。

另外，同名的函数声明会发生覆盖的现象：

```javascript
foo(); // 3

function foo() {
	console.log( 1 );
}

var foo = function() {
	console.log( 2 );
};

function foo() {
	console.log( 3 );
}
```

