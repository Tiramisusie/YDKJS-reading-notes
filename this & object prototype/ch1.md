# Chapter 1: `this` Or That?

## Confusions

关于`this`的一个常见误解是函数内部的`this`指向函数本身。但事实上`this`不指向函数本身。在函数内部引用这个函数本身最好的办法是用这个函数的函数名。

还能用`arguments.callee`来引用函数本身，但这个方法已经被废弃了。

### Its Scope

需要明确一点，`this`不指向函数的词法作用域。

## What's `this`?

`this`的绑定是运行时的绑定，或者说，`this`的绑定与函数定义的位置没有任何关系，而是由函数调用的位置决定。