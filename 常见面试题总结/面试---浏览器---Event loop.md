### 进程和线程的区别

进程描述了CPU在运行指令及加载和保存上下文所需的时间，放在应用上来说就代表了一个程序。

线程是进程中更小单位，描述了执行一段指令所需的时间。

把这些概念拿到浏览器来说，当你打开一个Tab页时，其实就是创建了一个进程，一个进程可以有多个线程，比如渲染线程、JS引擎线程、HTTP请求线程等。当你发起一个请求时，其实就是创建了一个线程，当请求结束后，该线程可能就会被销毁。

### JS单线程带来的好处

我们都知道JS是单线程执行的。

上面有提到JS引擎线程和渲染线程，大家应该都知道，在js运行的时候可能会阻止UI渲染，这说明两个线程是**互斥**的。这其中的原因是因为js执行的时候UI线程还在工作，可能会导致不安全的UI渲染。这其实也是一个单线程的好处，得益于js是单线程运行的，可以达到节省内存，节约上下文切换时间。

### 浏览器中的Event Loop

可以把所有的任务分为两种：

- 同步任务
- 异步任务

同步任务指的是，在主线程上排队执行的任务，只有前一个任务完成之后才会执行后一个任务，并且主线程上的存储结构是栈结构，遵循先进后出的原则。

异步任务指的是，不进入主线程而进入“任务队列”的任务，只有“任务队列”通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

**任务队列**

上面有提到异步任务会进入任务队列进行排队，那么他们会在任务队列中好好排队吗，答案是“no”，因为任务队列分为两种：

- 宏任务
- 微任务

有些文章就把微任务说成是任务中的VIP，而宏任务就是那些没有特权的异步任务，所以到底哪些任务属于微任务，哪些属于宏任务呢？

宏任务：script、setTimeout、setInterval、setImmediate、I/O/UI rendering

微任务：Promise、process.nextTick、MutationObserver（h5新特性）

下面来看以下代码的执行顺序：

```js
console.log('script start')

async function async1() {
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2 end')
}
async1()

setTimeout(function() {
  console.log('setTimeout')
}, 0)

new Promise(resolve => {
  console.log('Promise')
  resolve()
})
  .then(function() {
    console.log('promise1')
  })
  .then(function() {
    console.log('promise2')
  })

console.log('script end')
//script start--->async2 end--->Promise--->script end--->async1 end--->promise1--->promise2
```

所以Event Loop执行顺序可以总结成如下：

- 执行同步代码
- 遇到宏任务放到宏队列中，遇到微任务放到微队列中
- 当执行完同步代码后，主线程为空，查询是否有异步代码需要执行
- 执行所有微任务
- 当执行完所有微任务后，如果有必要会渲染界面
- 执行一次宏任务中的一个任务，执行完毕
- 执行所有微任务
- 当执行完所有微任务后，如果有必要会渲染界面
- 依次循环

我自己总结了这样的一个结论：就是在所有代码执行之前，浏览器会先执行一个宏任务，那就是script，所以有异步代码才会先执行微任务，从循环顺序也可以看出是先执行一个宏任务才执行所有的微任务的。

这样也就可以解释一个误区了，那就是认为微任务比宏任务快，其实并不是，只是浏览器先执行了一个宏任务script，所以接下来有异步代码就会先执行微任务。
