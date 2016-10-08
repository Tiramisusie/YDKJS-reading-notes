# Chapter 2: `this` All Makes Sense Now!

## Call-site

「call-site」就是函数被调用的位置，它决定了`this`的指向。

## Nothing But Rules

`this`的绑定遵循以下4条规则中的一条。但要记住，无论是哪一种情况，最重要的还是要找出「call-site」的位置。

### Default Binding

第一条规则是最常见的独立的函数调用：

```javascript
function foo(){
  console.log(this.a);
}

var a = 2;

foo()	// 2
```

在这里，`foo()`是在一个没有任何修饰的函数应用中被调用，所以默认情况下，`this`指向全局对象。如果是严格模式，`this`会指向`undefined`。

### Implicit Binding

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

obj.foo(); // 2
```

这一次，`foo()`的调用位置有一个上下文对象。

所以第二条规则就是：When there is a context object for a function reference, the *implicit binding* rule says that it's *that* object which should be used for the function call's `this` binding.

Only the top/last level of an object property reference chain matters to the call-site. For instance:

```js
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### Implicitly Lost

注意以下几个例子：

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // function reference/alias!

var a = "oops, global"; // `a` also property on global object

bar(); // "oops, global"
```

虽然`bar`表面上是`obj.foo`的一个引用，但事实上只是`foo`的另外一个引用罢了。

```js
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` is just another reference to `foo`

	fn(); // <-- call-site!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

doFoo( obj.foo ); // "oops, global"
setTimeout( obj.foo, 0 ); // "oops, global"
```

### Explicit Binding

第三条规则是`call(..)`和`apply(..)`函数。

### `new` Binding

第四条规则是`new`绑定。

当使用`new`关键词来调用一个具有「构造函数」作用的函数时，这个函数会返回一个全新的对象，并且把`this`绑定到这个新对象。

```js
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

## Everything In Order

在判断`this`的指向时，如果函数的「call-site」同时符合以上多个规则，需要按一定的优先级来应用这些规则：

1. Is the function called with `new` (**new binding**)? If so, `this` is the newly constructed object.

    `var bar = new foo()`

2. Is the function called with `call` or `apply` (**explicit binding**), even hidden inside a `bind` *hard binding*? If so, `this` is the explicitly specified object.

    `var bar = foo.call( obj2 )`

3. Is the function called with a context (**implicit binding**), otherwise known as an owning or containing object? If so, `this` is *that* context object.

    `var bar = obj1.foo()`

4. Otherwise, default the `this` (**default binding**). If in `strict mode`, pick `undefined`, otherwise pick the `global` object.

    `var bar = foo()`

## Binding Exceptions

### Ignored `this`

如果把`null`、`undefined`作为`call(..)`、`apply(..)`、`bind(..)`的第一个参数，这些值会被忽略并最终应用 default binding。

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```



