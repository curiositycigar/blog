---
title: 简单模拟实现es6 Promise
date: 2018-06-11 16:01:08
categories: 前端
tags: Promise
---
# 简单实现一个ES6 Promise对象

-----

> 一个`promise`对象接收的是一个`callback`
> 这个`callback`接收两个参数`(resolve,reject)`
> 当我们在`callback`内执行`resolve`或`reject`的时候，就会调用`Promise`内定义的 `resolve`和`reject`函数
> 然后，`resolve`和`reject`函数会改变Promise的状态
> 所以它应该是像下面这样的

```
function MyPromise(callback) {
  // 保存this值
  var self = this
  // 记录状态null为pending，true为resolved，false为reject
  var state = null
  // 记录resolve的参数
  var param = null

  // 执行传入的callback并改变promise对象状态
  callback(resolve, reject)
  // resolve方法
  function resolve(data) {
    // 改变状态
    state = true
    param = data
  }

  // reject方法
  function reject(err) {
    state = false
    param = err
  }
}
```

> 但没有`then`方法的`Promise`对象是不完整的(完全没有用)
> 所以我们需要一个`then`方法，要记住`then`方法返回的也是一个`promise`对象
> `then`方法接收两个可选的参数`(onFulfilled, onRejected)`(我们可以先忽略可选两个字)
> `then`方法传进来的参数必须是函数，如果不是就要忽略(PromiseA+规范)(我们可以也先忽略这句话)

```
  this.then = function (onFulfilled, onRejected) {
    // 返回一个新的promise对象
    return new self.constructor(function (resolve, reject) {
      // then
    })
  }
```

> 接下来就是`then`方法的具体实现了
> `then`方法中`onFulfilled, onRejected`的返回值会作为新promise的执行结果

> `onFulfilled, onRejected`这两个函数要在`promise`的状态变为pending或resolved的时候才能分别执行
> 所以如果`promise`方法状态为`resolved`或`rejected`的话，我们就可以直接在then方法中执行`resolve(onFulfilled(param))`和`reject(onRejected(param))`

```
  this.then = function (onFulfilled, onRejected) {
    // 返回一个新的promise对象
    return new self.constructor(function (resolve, reject) {
       if (state === true) {
        // param是promise对象完成后的结果
        resolve(onFulfilled(param))
      } else if (state === false) {
        reject(onRejected(param))
      } else {
        // 没有执行完毕,怎么办
      }
    })
  }
```

> 但如果`promise`的状态为`pending`呢
> 由于原始`promise`的状态我们是无法动态获取的，因此我们就需要在他执行状态改变的时候同时执行`onFulfilled`和`onRejected`方法
> 我们可以把这个方法放在原始`promise`对象的`resolve`和`reject`方法中执行
> 因此我们要在`promise`的对象定义中添加四个参数，分别记录`onFulfilled`和`onRejected`，以及`then`方法返回的新`promise`对象的`resolve`和`reject`
> 然后如果执行`then`方法的时候`promise`对象的状态为`pending`的话，就将上述四个参数记录起来

```
// then方法返回的promise对象的resolve和reject
var nextResolve = null
var nextReject = null
// 记录then方法的参数，onFulfilled和onRejected
var asynconFulfilled = null
var asynconRejected = null

//then方法
this.then = function (onFulfilled, onRejected) {
    // 返回一个新的promise对象
    return new self.constructor(function (resolve, reject) {
      if (state === true) {
        // param是promise对象完成后的结果
        resolve(onFulfilled(param))
      } else if (state === false) {
        reject(onRejected(param))
      } else {
        nextResolve = resolve
        nextReject = reject
        asynconFulfilled = onFulfilled
        asynconRejected = onRejected
      }
    })
  }
```

> 接下来就是原始`promise`中的`resolve`和`reject`的重新实现

```
 // resolve方法
  function resolve(data) {
    // 改变状态
    state = true
    param = data
    nextResolve(asynconFulfilled(param))
  }

  // reject方法
  function reject(err) {
    state = false
    param = err
    nextReject(asynconRejected(param))
  }
```

> 很简单不是吗
> 我们继续
> 上述实现我们一直没有考虑一个很重要的情况，如果`then`方法返回的还是一个`promise`对象，那么如果我们后边还有`then`方法的话就要等待前一个`then`方法中的`promise`对象的状态从`pending`变为完成
> 这要怎么做呢
> 什么时候可以认为then方法返回的`promise`对象执行完毕了呢，这里我们就要用到`then`方法(@_@，边写边用...),
> 以`resolve`方法为例


```
var self = this
// resolve方法
function resolve(data) {
  // 记录onFulfilled的执行结果
  let parmise
  // 改变状态
  state = true
  param = data
  // 执行记录的onFulfilled
  parmise = asynconFulfilled(param)
  if(parmise === undefined){
    // 如果parmise为undefined,就不能解析parmise.constructor
  } else if (parmise.constructor === self.constructor) {
    // 等待传递进来的promise对象执行完毕，然后根据传递进来的promise对象的状态执行resolve或reject
    // 注意，这个param是形参，在then方法的promise中执行
    promise.then(function (param) {
      resolve(param)
    }, function (param) {
      reject(param)
    })
  } else {
    // 这个是前边的then返回的不是promise对象的情况
    resolve(promise)
  }
}
```

> 前面我们忽略了两点 (**`then`方法接收两个可选的参数`(onFulfilled, onRejected)`**) 和 (**`then`方法传进来的参数必须是函数，如果不是就要忽略**)

```
var self = this
// resolve方法
function resolve(data) {
  // 记录onFulfilled的执行结果
  var parmise
  // 改变状态
  state = true
  param = data
  // 执行记录的onFulfilled
  // begin--------------
  if (typeof onFulfilled === 'function') {
    promise = onFulfilled(param)
    if (promise === undefined) {
      // 待补充
    } else if (promise.constructor === self.constructor) {
      // 注意，这个param是形参，在then方法的promise中执行
      promise.then(function (param) {
        resolve(param)
      }, function (param) {
        reject(param)
      })
    } else {
      reject(promise)
    }
  } else {
    // 如果onFulfilled不是function，忽略，直接resolve或reject
    resolve(param)
  }
  // ---------------end
}

```

> 上面begin到end之间的代码还要在`then`方法调用，所以我们可以把这段代码抽象为一个函数
> `resolve `和 `reject`的原理相同，只要注意**如果不是function** 的话需要执行reject

---
> `onFulfilled`和 `onRejected`只有在[执行环境]堆栈仅包含**平台代码**时才可被调用
> 所以将上述***begin-end***之间的代码放到`seTimeout`中执行(浏览器环境)

```
function resolve(data) {
  // 记录onFulfilled的执行结果
  var parmise
  // 改变状态
  state = true
  param = data
  // 执行记录的onFulfilled
  window.setTimeout(function () {
    // begin--------------
    // 上述代码
    // ---------------end
  }, 0)
}
```

> 下面是完整代码

注：因为懒，所以很多地方不符合规范

```
// 简单实现ES6 Promise
function MyPromise(callback) {
  // 保存this值
  var self = this
  // 记录状态null为pending，true为resolved，false为reject
  var state = null
  // 记录resolve的参数
  var param = null
  // then方法返回的promise对象的resolve和reject
  var nextResolve = null
  var nextReject = null
  // 记录then方法的参数，onFulfilled和onRejected
  var asynconFulfilled = null
  var asynconRejected = null

  // 执行并改变promise对象状态
  callback(resolve, reject)
  // then方法
  this.then = function (onFulfilled, onRejected) {
    // 返回一个新的promise对象
    return new self.constructor(function (resolve, reject) {
      // 判断异步代码是否执行完毕(是否resolve或reject)
      // 若执行完毕就在then方法中立即执行,否则将四个参数记录下来,等待state就绪后再执行doAsyn*函数
      if (state === true) {
        doAsynconFulfilled(onFulfilled, resolve, reject)
      } else if (state === false) {
        doAsynconRejected(onRejected, resolve, reject)
      } else {
        nextResolve = resolve
        nextReject = reject
        asynconFulfilled = onFulfilled
        asynconRejected = onRejected
      }
    })
  }
  // resolve方法
  function resolve(data) {
    // 改变状态
    state = true
    param = data
	if(nextResolve){
		doAsynconFulfilled(asynconFulfilled, nextResolve, nextReject)
	}
  }

  // reject方法
  function reject(err) {
    state = false
    param = err
	if(nextReject){
		doAsynconRejected(asynconRejected, nextResolve, nextReject)
	}
  }

  // 核心方法(我觉得是)

  function doAsynconFulfilled(onFulfilled, resolve, reject) {
	  window.setTimeout(function () {
		// 判断onFulfilled是否为function，不是则忽略
		if (typeof onFulfilled === 'function') {
		  // 执行onFulfilled方法获取返回值promise()
		  let promise = onFulfilled(param)
		  // 如果promise为undefined 执行 if
		  // 如果promise为MyPromise对象 执行 else if
		  // 如果promise为非MyPromise对象 执行 else
		  if (promise === undefined) {
			resolve(param)
			// 待补充
		  } else if (promise.constructor === self.constructor) {
			// 等待传递进来的promise对象执行完毕，然后根据传递进来的promise对象的状态执行resolve或reject
			promise.then(function (param) {
			  resolve(param)
			}, function (param) {
			  reject(param)
			})
		  } else {
			// 执行then方法返回的对象的resolve
			resolve(promise)
		  }
		} else {
		  // 传递参数
		  resolve(param)
		}
	  }, 0)
    
  }

  function doAsynconRejected(onRejected, resolve, reject) {
	window.setTimeout(function () {
		if (typeof onRejected === 'function') {
		  let promise = onRejected(param)
		  if (promise === undefined) {
			reject(param)
			// 待补充
		  } else if (promise.constructor === self.constructor) {
			promise.then(function (param) {
			  resolve(param)
			}, function (param) {
			  reject(param)
			})
		  } else {
			reject(promise)
		  }
		} else {
		  // 传递错误信息
		  reject(param)
		}
	}, 0)
  }
}
// 测试使用
var b = function (message) {
	return new MyPromise(function (resolve, reject) {
		document.body.onclick = function () {
			resolve('click:' + message)
		}
	})
}
var a = new MyPromise(function (resolve, reject) {
  resolve(123)
}).then(function (message) {
	return b(message)
}).then().then(function (message) {
	console.log('final:' + message)
},function (err) {
	console.log('final:' + err)
})
console.log('window')
```
> 完毕！