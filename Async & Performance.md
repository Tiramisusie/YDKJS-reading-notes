# Async & Performance

## Generator

generator 本质上也是函数，但通过 `yield` 关键字可以实现暂停，从而保存当前的执行状态，然后通过 `next()` 方法恢复执行。

generator 的定义方式有下面几种：

```js
// 第一种
function*foo(){}

// 第二种
function *foo(){}

// 第三种
function* foo(){}
```

有以下代码：

```js
var x = 1;

function *gen(){
  x++;
  y = yield x;
  x++
}

var foo = gen();

z = foo.next();		// x:2; z.value:2

foo.next(10);		// x:3; y:10 
```

上面的例子中，有几个要点：

- 函数真正开始执行是在[line 9]第一次调用`next()`；
- `next()`调用的次数比`yield`的个数多`1`;
- 遇到`yield`的时候会对外返回一个对象，这个对象的`value`属性的值是`yield x`中的`x`;
- 在 [line 13] 的`next()`中传入了一个参数`10`，这个值会作为这个`next()`所对应的`yield`的返回值，即`y`的值为`10`，注意与`yield`对外返回的对象区分。

generator 可以作为迭代器，不断的生成值。例如：

```js
function *something() {
	var nextVal;

	while (true) {
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}

		yield nextVal;
	}
}
```

如果想要终止一个 generator ，可以使用 `foo.return(..)`，这会终止一个 generator ，并返回 `return(..)` 里面的参数。

很多时候都会把 generator 和 promise 结合起来使用。简单来说就是在 generator 里面 `yield` 一个 `Promise`，例如：

```js
function foo(x,y) {
  // 返回一个 Promise
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

var it = main();

var p = it.next().value;

// wait for the `p` promise to resolve
p.then(
	function(text){
		it.next( text );
	},
	function(err){
		it.throw( err );
	}
);
```



## Benchmarking & Tuning

在我们测试 js 代码性能的时候，存在很多错误的认知。

### Benchmarking

例如有下面的代码：

```js
var start = (new Date()).getTime(); // or `Date.now()`
// do some operation
var end = (new Date()).getTime();
console.log( "Duration:", (end - start) );
```

这样的测试方式其实是不太正确的。

- 某些浏览器（例如老版本IE）不支持个位数的毫秒精度。对应一个最小时间精度为 15ms 的浏览器，0ms 的执行时间有可能是小于 1ms ，也有可能是小于 15ms 。
- 对于不同的测试环境，对 js 的优化程度可能不同。
- 在获取 `start` 或者 `end` 时间戳的时候有可能会有延迟。

### Repetition

重复执行同一段代码很多次，然后取平均执行时间，这种方法也是不准确的。