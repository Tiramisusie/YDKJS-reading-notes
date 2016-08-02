# Chapter 5: Scope Closure

## Nitty Gritty

有以下代码段：

```javascript
var a = 1;

function foo(){
  var a = 2;
  
  function bar(){
    console.log(a);
  }
  
  return bar;
}

var baz = foo();
baz();   // 2
```

以上就是一个简单的闭包例子。

当我们执行`foo()`的时候，把`bar()`返回了，并且赋给了变量`baz`，这样的结果就是`baz`引用了`function bar(){..}`。但执行`baz()`的时候，就是调用了`function bar(){..}`。由于`function bar(){..}`是在函数`function foo(){..}`的作用域中，所以会从函数`foo()`的作用域中开始查找（解析）变量`a`。所以最后输出`2`。

另外一个问题是为什么在`foo()`执行完之后还能获取到函数`foo()`中的变量`a`？为什么`foo()`的作用域没有被 JavaScript 引擎的垃圾回收机制回收？原因是它的作用域还在被使用，被函数`bar()`使用。由于函数`baz()`保存了对函数`bar()`的引用，而函数`bar()`保存了对`foo()`的作用域的引用，所以`foo()`的作用域不会被回收。

**`bar()`对`foo()`的作用域的引用就是我们说的闭包**

## Loops + Closure

有以下代码：

```javascript
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, 0 );
}
```

我们都知道上面的代码会输出5个6，但是为什么？

首先，5个`setTimeout()`的回调函数都是严格的在循环结束之后执行。循环结束后，`i`的值是6；还应该注意到，虽然每次循环都会定义一个`setTimeout()`，但是这5个`setTimeout()`都共享了同一个作用域，而这个作用域中只有一个`i`。所以最终输出的就是5个6。

如果我们需要输出`1,2...5`，可以这样：

```javascript
 for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, 0 );
	})( i );
}
```

上面的代码中，在每个循环中都创建了一个独立的作用域，每个作用域中都有自己的`i`。循环结束后，就会有5个不同的作用域，每个作用域中的`i`的值分别为`1,2,3,4,5`。

### Block Scoping Revisited

上面的代码中，我们利用 IIFE 在每次循环中都创建一个作用域。换句话说，我们需要在每次循环中都有一个块级作用域。在前面讨论过，利用`let`来声明变量时会在所在的块中创建一个块级作用域。于是，我们可以这样修改我们的代码：

```javascript
for (var i=1; i<=5; i++) {
	let j = i; 
	setTimeout( function timer(){
		console.log( j );
	}, 0 );
}
```

或者这样：

```javascript
for (let i=1; i<=5; i++) {	
  //每次循环中，i都会被重新声明，并且赋值为上次循环结束时i的值
	setTimeout( function timer(){
		console.log( i );
	}, 0 );
}
```

