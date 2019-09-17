# ES6入门---async函数

## 含义

ES6引入了async函数，使得异步操作变得更加的简单。

**async函数是什么？**

> 一句话，它就是Generator函数的语法糖

前面有一个Generator函数读取两个文件的例子，其中Generator函数代码如下：

```js
let gen = function* () {
  let r1 = yield readFile('../data/a.txt');
  let r2 = yield readFile('../data/b.txt');
  console.log(r1.toString());             
  console.log(r2.toString());             
}
```

接下来，是用async函数写成gen函数的代码：

```js
let gen = async function () {
  let r1 = await readFile('../data/a.txt');
  let r2 = await readFile('../data/b.txt');
  console.log(r1.toString());             
  console.log(r2.toString());             
}
```

上面的Generator函数的代码和async函数代码一比较就会发现其中的几点不同：

- async函数将Generator函数的*替换成了async
- async函数将Generator函数的yield替换成了await

就上面两点仅此而已，是不是很像，所以说async函数是对Generator函数的改进，体现在以下四点：

**1. 内置执行器**

Generator函数的执行是必须靠执行器的，所以才有了co模块，而async函数自带执行器。也就是说，async函数的执行，与普通函数一样。

```js
//两者的执行方式比较

//Generator函数
//基于co模块的执行方式
co(gen());

//手动执行Generator函数的方式
let g = gen();
g.next();
//....

//async函数
gen();
```

从上面的两者的比较也能看出来，调用async函数，然后就能自动执行，输出最后的结果。这不同于Generator函数，Generator函数需要调用next方法，或者调用co模块才能真正执行，得到最后的结果。

**2.更好的语义**

async/await比起*/yield，语义更加清楚。async表示函数里面有异步操作，await表示紧跟在后面的表达式需要等待结果。

**3.更广的适用性**

- co模块约定，yield命令后面只能是Thunk函数或Promise对象
- async函数的await命令后面，可以是Promise对象或原始类型的值（数值、字符串、布尔值），但是原始类型的值会自动转换成立即resolved的Promise对象。

**4.返回值是Promise**

async函数的返回值是Promise对象，这比Generator函数的返回值是遍历器对象方便多了。async函数直接可以用then方法指定下一步的操作。

进一步说，async函数完全可以看做多个异步操作，包装成Promise对象。而await命令就是内部then命令的语法糖。

## 基本用法

async函数返回的是一个Promise对象，可以使用then方法添加回调函数，当函数执行的时候，一旦遇到await就会返回，等到异步操作完成再接着执行函数体内后面的语句。

**1.基本用法的例子：**

```js
async function getSum(num) {
  let andSum = await getAnd(num);
  let multiplySum = await getMultiplySum(andSum);
  return multiplySum;
}

function getAnd(num) {
  return new Promise((resolve, rejecte) => {
    resolve(num + 2);
  });
}

function getMultiplySum(num) {
  return new Promise((resolve, rejecte) => {
    resolve(num * 2);
  });
}

getSum(3).then(res => {
  console.log(res);         //10
})
```

上面的代码是一个运算的代码，规定先进行加法运算在进行乘法运算，getSum函数前面有async关键字，表明该函数内部有异步操作，await表明后面的函数需要等待有结果之后，后面的代码才能继续执行。调用getSum函数，会立即返回一个Promise对象。用then接受返回的值。

**2.延迟输出的例子**

指定多少毫秒后输出一个值。

```js
async function asyncPrint(content, ms) {
  await timeout(ms);
  console.log(content);
}

function timeout(ms) {
  return new Promise((resolve, rejecte) => {
    setTimeout(resolve, ms);
  })
}

asyncPrint('hello async', 3000);
```

上面的代码，指定了在3000毫秒之后输出想要输出的内容。

## 语法

async函数的语法规则总体上比较简单，难点是错误处理机制。

### 返回Promise对象

async函数返回一个Promise对象。

1.async函数内部return语句返回的值，会成为then方法回调函数的参数。

```js
async function fun() {
  return 'hello';
}

fun().then(res => {
  console.log(res);         //hello
})
```

上面代码fun函数的return语句的返回值，会被then方法的回调函数接收到。

2.async函数内部抛出错误，会导致返回的Promise对象变为reject状态。抛出的错误对象会被catch方法回调函数接收到。

```js
async function fun() {
  throw new Error('出错了');
}

fun().then(res => {
  console.log(res);
}).catch(err => {
  console.log(err);       //Error: 出错了
})
```

### Promise对象状态的变化

async函数返回的Promise对象，必须等到内部所有await命令后面的Promise对象执行完，才会发生状态改变，除非遇到return语句或者抛出错误。也就是说，只有async函数内部的异步操作执行完成，才会执行then方法指定的回调函数。

在上面的基本用法的运算代码中有所体现。只有先运行了加法的函数，再运行乘法的函数，只有这两个操作全部完成才会执行then方法里面的结果输出。

### await命令

正常情况下，await命令后面是一个Promise对象，返回该对象的结果，如果不是Promise对象，就直接返回对应的值。

```js
async function fun() {
  return await 'hello';
}

fun().then(res => {
  console.log(res);         //hello
})
```

上面代码中await后面不是一个Promise对象，是一个字符串，直接将字符串的值返回。

另外一种情况是，await后面是一个thenable对象（即定义then方法的对象），那么await会将其等同于Promise对象。

```js
let obj = {
  name: 'lee',
  then(resolve, rejecte) {
    resolve(this.name);
  }
};

(async () => {
  let res = await obj;
  console.log(res);         //lee
})();
```

上面的代码中await后面跟的是一个对象，这个对象不是Promise对象，但是因为定义了then方法，await会将其视为Promise对象处理。

之前JavaScript一直没有休眠的语法，但是借助await命令可以让程序停顿指定的时间，下面是一个简化的睡眠的函数：

```js
async function asyncFun() {
  for(let i = 1; i <= 5; i++) {
    console.log(i);
    await sleep(2000);
  }
}

function sleep(interval) {
  return new Promise((resolve, rejecte) => {
    setTimeout(resolve, interval);
  })
}

asyncFun();
```

上面的代码每隔2秒钟会输出一下。

async函数如果有多个await命令，其中任何一个await语句后面的Promise的对象变为reject状态，那么整个async函数都会中断执行。

```js
async function asyncFun() {
  await Promise.reject('出错了');
  await Promise.resolve('hello')
}
```

上面的代码中，第二个await语句不会执行，因为第一个await语句状态变成了reject。

有时，我们希望无论前一个异步是否成功与否，也不要中断后面的异步操作。这时可以将前一个异步操作放在try/catch代码块中，这样无论这个异步是否成功，都不会影响后面的执行。

```js
async function asyncFun() {
  try {
    await Promise.reject('出错了');    
  } catch(e) {
    console.log(e);             //出错了
  }
  return await Promise.resolve('hello')
}

asyncFun().then(res => {
  console.log(res);             //hello
});
```

另一种解决办法是在await后面的Promise对象再跟一个catch方法，处理前面可能出现的错误。

```js
async function asyncFun() {
  await Promise.reject('出错了').catch(err => {
    console.log(err)            //出错了
  })
  return await Promise.resolve('hello')
}

asyncFun().then(res => {
  console.log(res)              //hello
})
```

### 错误处理

如果await后面的异步操作出错，那么等同于async函数返回的Promise对象被reject。

```js
async function asyncFun() {
  await new Promise((resolve, rejecte) => {
    throw new Error('出错了');
  });
}

asyncFun().then(res => {
  console.log(res);             
}).catch(err => {
  console.log(err);           //出错了
})
```
上面代码中await后面的Promise对象抛出了一个错误，这样会导致async函数最终的状态为reject，抛出的错误也会被async的catch方法的回调函数执行。

防止出错的方法，是将其放在try/catch代码块中，如果有多个await命令，可以统一放。

### 注意使用点

**1. 前面已经说过了，await命令后面的Promise对象，运行结果可能是reject，如果只有一个await命令，可以选择在Promise后面用catch方法来避免，如果有多个await命令，最好把await命令放在try/catch代码块中。两种方式，可以视情况而定**

```js
//第一种方式
async function asyncFun() {
  await sleep().catch(err => {
    console.log(err);
  })
}

//第二种方式
async function asyncFun() {
  try {
    await sleep();
  } catch(err) {
    console.log(err);
  }
}
```

**2. 多个await命令后面的异步操作，如果不存在继发关系，最好让他们同时触发，这样可以节约时间**

```js
async function asyncFun() {
  let [fun1, fun2] = await Promise.all([asyncFun1(), asyncFun2()])
}
```

**3. await命令只能用在async函数中，如果用在普通函数中就会报错**

**4. async函数可以保留运行栈**

```js
let a = () => {
  b().then(() => c())
}
```

上面的代码中，函数a内部运行了一个异步任务b函数，当b函数运行的时候，函数a不会中断，而是继续执行。等到b函数运行结束，可能a早就已经结束了，b所在的上下文环境已经消失了，这时如果b或c出错，错误堆栈将不包括a。

```js
let a = async () => {
  await b()
  c()
}
```

再看一下，async函数，b函数运行的时候，a函数是暂停执行的，上下文环境都保存这，一旦b函数或c函数报错，错误堆栈将包括a函数。

## async函数实现的原理

async函数实现的原理，就是将Generator函数和自动执行器，包装在一个函数里面。

```js
async function fun() {}

//等同于

function fun() {
  return spawn(function* () {
    //...
  })
}
```

所有async函数都可以写成上面的第二种形式，其中spawn函数就是自动执行器。

下面给出spawn函数的实现，其实就是前面有写到过的自动执行器的翻版。

```js
function spawn(genF) {
  return new Promise((resolve, rejecte) => {
    let gen = genF();
    
    function step(nextF) {
      let result;
      try{
        result = gen.next(nextF);
      } catch(e) {
        rejecte(e)
      }

      if(result.done) {
        return resolve(result.value);
      } 

      Promise.resolve(result.value).then(res => {
        step((res) => {
          return gen.next(res);
        })
      }).catch(err => {
        step((err) => {
          return gen.throw(err);
        })
      })
    }

    step(() => {
      return gen.next(undefined);
    });
  })
}
```