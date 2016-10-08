# You Don't Know JS: ES6 & Beyond

## Iterators

 *iterator* 就是一种结构化的模式，它允许你从一个数据源获取数据，且每一次只能获取一个数据。这种默认其实早就存在于 js 中，ES6 只是把它抽象成标准化的接口。

### Interfaces

`Iterator`有以下的接口：

```js
Iterator 
	next() {method}: retrieves next IteratorResult
	// 下面两个方法是可选的
	return() {method}: stops iterator and returns IteratorResult
	throw() {method}: signals error and returns IteratorResult
```

还有一个 `IteratorResult` 接口：

```js
IteratorResult
	value {property}: current iteration value or final return value
		(optional if `undefined`)
	done {property}: boolean, indicates completion status
```

一次 iterator 操作的返回结果必须是以上的形式。

### `next()` Iteration

对于数组，可以用以下方式来迭代返回它的值：

```js
var arr = [1,2,3];

var it = arr[Symbol.iterator]();

it.next();		// { value: 1, done: false }
it.next();		// { value: 2, done: false }
it.next();		// { value: 3, done: false }

it.next();		// { value: undefined, done: true }
```

在上面的例子中，当返回的 `value` 为 `3` 的时候，`done` 的值不是 `true` ，而是要再调用一次 `next()` 才会返回 `done:true` 。具体原因在下面的内容中说明。

对于任何已经遍历完成的 iterators ，再次调用 `next()` 方法只会返回 `{ value: undefined, done: true }`。

### Optional: `return(..)` and `throw(..)`

`return()` 和 `throw()` 都是可选的方法。

`return()` 方法表示 iterator 的终止，即不再返回数据。调用 `return()` 之后会返回一个 `IteratorResult` 对象，`return(..)` 的参数会作为 `IteratorResult` 中的 `value` 的值。

`throw(..)` 用于抛出 iterator 中的错误或异常。

**注意** 通常在 `return()` 和 `throw()` 调用后都不应该再产生其他值。

### Iterator Loop

对于是 iterable 的 iterator ，例如数组，可以使用 `for..of` 循环来进行遍历。而对于自定义的 iterator ，可以添加 `Symbol.iterator` 方法使其变成一个 iterable 。（一个 iterable 就是一个可以生成 iterator 的对象）

```js
var it = {
	// make the `it` iterator an iterable
	[Symbol.iterator]() { return this; },

	next() { .. },
	..
};

it[Symbol.iterator]() === it;		// true

for (var v of it) {
  console.log(v);
}
```

上面的 `for..of` 循环可以看成是以下的等价 `for` 循环：

```js
for (var v, res; (res = it.next()) && !res.done; ) {
	v = res.value;
	console.log( v );
}
```

可以看出，每一次循环，都会调用一次 `next()` 方法，并判断一次 `done` 的值。

当 `done` 的值为 `true` 时，将不会执行循环体中的代码。加入有这样一个对象 `{value: 1, done: true}` ，那么 `1` 将会被忽略或者说丢失。所以 iterator 返回最后一个值的时候，`done` 的值依然是 `false` ，要再调用一次 `next()` ，`done` 的值才会是 `true`。

### Custom Iterators

利用 iterator 构造一个 Fibonacci 序列：

```js
var Fib = {
	[Symbol.iterator]() {
		var n1 = 1, n2 = 1;

		return {
			// make the iterator an iterable
			[Symbol.iterator]() { return this; },

			next() {
				var current = n2;
				n2 = n1;
				n1 = n1 + current;
				return { value: current, done: false };
			},

			return(v) {
				console.log(
					"Fibonacci sequence abandoned."
				);
				return { value: v, done: true };
			}
		};
	}
};

for (var v of Fib) {
	console.log( v );

	if (v > 50) break;
}
// 1 1 2 3 5 8 13 21 34 55
// Fibonacci sequence abandoned.
```

调用 `Fib[Symbol.iterator]()` 方法之后会返回一个包含 `next()` 和 `return()` 方法的 iterator 对象。

### Iterator Consumption

除了可以用 `for..of` 循环来迭代一个 iterator，还能用 `…` 操作符。

类似于数组的结构赋值：

```js
var [x,...y] = [1,2,3,4,5];
x	// 1
y	// [2,3,4,5]
```

对于上面的 Fibonacci 数列，可以用以下形式调用：

```js
var it Fib[Symbol.iterator]();

var [a,b,c,d,e] = it;

console.log(a,b,c,d,e);
// "Fibonacci sequence abandoned."
// 1 1 2 3 5
```

**注意** 上面的调用方式会终止 iterator，从个调用对应的 `return()` 方法，所以会先打印出 `"Fibonacci sequence abandoned."` 。

## Generator

generator 的详细内容参考第二章

### `yield` 的优先级

`yield` 的优先级很低，只比 `…` 和 `,` 的优先级高。

### `yield *`

`yield *` 是一种叫做  *yield delegation* 的机制，简单的说就是可以在一个 generator 中点调用另外一个 iterator。

### Iterator Control

利用 `yield *` 调用 generator 本身，可以实现类似于递归的效果：

```js
function *foo(x) {
	if (x < 3) {
		x = yield *foo( x + 1 );
	}
	return x * 2;
}

var it = foo( 1 );
it.next();				// { value: 24, done: true }
```

上面的程序没有 `yield ..` 这样的表达式，所以 generator 本身并没有暂停。利用 `yield *` 调用自身使得迭代能以递归的形式一直进行下去，所以只需要调用一次 `next()` 就能跑完整个 generator。

通常，要跑完一个 generator ，`next()` 的调用次数要比 `yield` 的个数多 `1` 。这是因为 `yield` 表达式会暂停 generator ，并等待一个值，就像在问一个问题“What value should I use here? I'll wait to hear back”。下一次 `next(..)` 的调用就会把这个值以参数的形式传进来，并恢复 generator 。

```js
function *foo() {
	var x = yield 1;
	var y = yield 2;
	var z = yield 3;
	console.log( x, y, z );
}

var it = foo();

// start up the generator
it.next();				// { value: 1, done: false }

// answer first question
it.next( "foo" );		// { value: 2, done: false }

// answer second question
it.next( "bar" );		// { value: 3, done: false }

// answer third question
it.next( "baz" );		// "foo" "bar" "baz"
						// { value: undefined, done: true }
```

第一次调用 `next()` 是用于启动 generator，之后的每一次 `next()` 都对应一个 `yield` 。

### Early Completion

`return(..)` 和 `throw(..)` 方法可以立即结束 generator。之后再调用 `next(..)` 方法只会返回 `{ value: undefined, done: true } `。

### Error Handling

可以用 `try...catch` 来捕获错误。`try...catch`可以写在 generator 里面或外面：

```js
function *foo() {
	try {
		yield 1;
	}
	catch (err) {
		console.log( err );
	}

	yield 2;

	throw "Hello!";
}

var it = foo();

it.next();				// { value: 1, done: false }

try {
	it.throw( "Hi!" );	// Hi!
						// { value: 2, done: false }
	it.next();

	console.log( "never gets here" );
}
catch (err) {
	console.log( err );	// Hello!
}
```



 