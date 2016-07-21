# Chapter 2: Lexical Scope

## Lex-time

lexical scope，词法作用域，就是在词法分析阶段定义的作用域。换句话说，词法作用域就是基于你在写程序时变量和作用域块（blocks of scope）的位置，所以在 lexer（词法分析器？）处理你的代码的时候就已经被确定了。

作用域在很多时候都是嵌套的。在查找变量时，总是从代码被执行的、最里层的作用域开始，一层一层的往外查找。当找到第一个符合的变量时，就会停止。

**NOTE**：无论一个函数在什么地方、怎么样被调用，它的词法作用域都只能由这个函数声明的位置所决定。

另外，对于`foo.bar.baz`这样的调用，词法作用域的查找处理只会发生在`foo`上面，一旦找到了`foo`之后，接下来就会由`object property-access`规则来负责`bar`和`baz`的解析。

## Cheating Lexical

上面提到，词法作用域是在函数定义时被决定的，那么是否有办法在运行时改变词法作用域呢？

JavaScript里面有两个改变词法作用域的机制，但都不被推荐，而且这样做会导致性能下降。

### `eval`

利用`eval(..)`函数可以在程序运行过程中动态的生成代码，并且引擎会把这些动态生成的代码看做是在程序编写过程中就已经存在的，也就是说引擎不会知道或者关心这些代码是否是动态插入的。这样做的结果就是可以在程序运行阶段修改词法作用域。

看看下面的例子：

```javascript
function foo(str, a) {
	eval( str ); // cheating!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1, 3
```

上面的代码当执行到`eval(..)`的时候，就会把`var b = 3`当做是一直都存在，而不是后来生成的。于是在`foo(..)`的词法作用域中，由`eval(..)`生成的变量`b`就会覆盖掉全局的变量`b`。

如果在`eval(..)`的词法作用域中使用了严格模式的话，它就不会修改它周围的作用域了。就像这样：

```javascript
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

