# ES6入门---Generator函数的异步应用的剖析

异步编程对JavaScript语言太重要了，JavaScript语言的执行环境是“单线程”的，如果没有异步编程，根本没有办法用，非卡死不可。

## 传统方法

在ES6诞生以前，异步编程的方法，大概有下面四种。

- 回调函数
- 事件监听
- 发布/订阅
- Promise对象

ES6的Generator函数将JavaScript的异步编程带入了一个全新的阶段。

### 异步的概念

所谓“异步”，简单的说就是一个任务不是连续完成的，可以理解成该任务被人分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二阶段。

比如，有一个任务是读取文件进行处理，任务的第一段是向操作系统发出请求，要求读取文件。然后。程序执行其他的任务，等到操作系统返回文件，再接着执行任务的第二段（处理文件）。这种不连续的执行，就叫做异步。

相应的，连续的执行就叫做同步。由于是连续执行，不能插入其他任务，所以操作系统从硬盘读取文件的这段时间，程序只能干等着。

### 回调函数

JavaScript语言对异步编程的实现，就是回调函数。所谓回调函数，就是把任务的第二段单独写在一个函数里面，等到重新执行这个任务的时候，就会直接调用这个函数。回调函数的英文名callback，直译过来就是“重新回调”。

```js
const fs = require('fs');

fs.readFile('../data/a.txt', (err, data) => {
  if(err) throw err;
  console.log(data.toString());
})
```

上面的代码是简单的读取一个文本内容的操作。readFile的第二个参数就是回调函数，也就是任务的第二段，只有等到操作系统返回了../data/a.txt这个文件以后，第二段也就是回调函数才会执行。

**一个有趣的问题？**

为什么Node约定，回调函数的第一个参数，必须是错误对象err（如果没有错误该参数就是null）？

原因是执行分成两个阶段，第一阶段执行完成之后，任务所在的上下文环境已经结束了。在这以后抛出的错误，原来的上下文环境已经无法捕捉，只能当做参数，传回第二阶段。

### Promise

回调函数本身并没有问题，他的问题出现在多个回调函数的嵌套。多个回调函数会出现强耦合的现象，只要有其中一个需要修改，那么他的上层回调函数和下层回调函数，可能都需要跟着修改。这种情况就称为**回调函数地狱**

Promise对象就是为了解决这个问题出现的，他不是新的语法功能，而是一种新的写法，允许将回调函数的嵌套，改成链式调用。

Promise的最大的问题是代码冗余，原来的任务被Promise包装了一下，不管什么操作，一眼看去都是一堆的then，语义变得很不清楚。

## Generator

### 协程

传统的编程语言，早有异步编程的解决方案（其实是多任务的解决方案）。其中有一种叫做“协程”，意思是多个线程互相协作，完成异步任务。

协程有点像函数，又有点像线程。他的运行流程大致如下：

- 第一步：协程A开始运行
- 第二步：协程A执行到一半，进入暂停，执行权转移到协程B
- 第三段：（一段时间后）协程B交还执行权
- 第四步：协程A恢复执行

上面流程的协程A，就是异步任务，因为他分成两段（或多段）执行。

读取文件的协程写法如下：

```js
function* gen() {
  //其它代码
  let f = yield fs.readFile('../data/a.txt');
  //其他代码
}
```

上面的代码的gen函数是一个协程，他的奥妙就在于yield表达式，他表示执行到此处，执行权将交给其他的协程，也就是说，yield表达式是异步两个阶段的分界线。

上面的代码是不是看起来很熟悉，没错就是我们写的Generator函数，那么协程和Generator函数之间的关系是怎样的呢？下面一点一点学习。

### 协程的Generator函数实现

Generator函数是协程在ES6的实现，最大的特点就是可以交出函数的执行权（即暂停执行）

整个Generator函数就是一个封装的异步任务，或者说是异步任务的容器，异步操作需要暂停的地方，都用yield表达式注明。

调用Generator函数，会返回一个内部指针对象，这是Generator函数不同于普通函数的一个地方，执行函数不会返回返回结果，返回的是指针对象。调用指针对象的next方法，会移动内部指针，指向第一个遇到yield表达式。

换言之，next方法的作用是分阶段执行Generator函数，每次调用next方法，回返回一个对象，表示当前阶段的信息。

### Generator函数的数据交换和错误处理

Generator函数可以暂停执行和恢复执行，这是他能封装异步任务的根本原因。除此之外，他还有两个特性，使他可以作为异步编程的完整解决方案：函数体内外的数据交换和错误处理机制。

其实这两个特性我们在前面学习Generator函数的时候都有接触过，还有过详细的介绍，只不过没有接触过他的官方名称而已。

函数体内外的数据交换：其实就是之前我们接触的next函数能传值的情况，next返回的value值，是Generator函数向外输出数据，next方法接收的参数，是向Generator函数内部输入数据。

错误处理机制：就是Generator函数的throw方法，能捕获函数体外抛出的错误。

## Thunk函数

Thunk函数是自动执行Generator函数的一种方法。

### 参数的求值策略

Thunk函数早在上个世纪60年代就诞生了。

那时，编程语言刚刚起步，计算机学家还在研究，编译器怎么写比较好，一个争论的焦点是**求值策略**，即函数的参数到底应该在何时进行求值。

**一种意见：传值调用**

传值调用：就是在进入函数体之前，就计算好参数的值，再将这个值传入函数，注意最后传入的是已经计算好的值。C语言就是采用的这种策略。

**另一种：传名调用**

传名调用：顾名思义，就是直接将表达式传入函数体，只有在函数体中用到的时候再求值。Haskell语言采用的是这种策略。

**传值调用和传名调用，哪一种比较好？**

回答是个有利弊，其实这个答案也可想而知，既然提出来了，就是有不同的适用情况。

原因是：传值调用比较简单，但是对参数求值的之后再传入函数体，就会出现一种情况，函数内没有用到传进来的参数，这样就会造成浪费。

### Thunk函数的含义

“传名调用”的实现，往往是将参数放到一个临时的函数中，再将这个临时函数传入函数体，这个临时函数就叫做Thunk函数。

```js
let x = 5;

function fun(value) {
  return value * 2;
}

console.log(fun(x + 1));           //12
 
//相当于
let thunk = function() {
  return x + 1;
}

function fun(value) {
  return value() * 2;
}

console.log(fun(thunk));        //12
```

上面的代码中，函数fun的参数x+1被一个函数替换了。这样只要在用到参数的地方，对thunk函数求值即可。

这就是Thunk函数的定义，他是“传名调用”的一种实现策略，用来替换某个表达式。

### JavaScript语言的Thunk函数

JavaScript语言是传值调用，他的Thunk函数含义有所不同。在JavaScript语言中，Thunk函数替换的不是表达式，而是多参数函数，将其替换成一个只接受回调函数作为参数的单参数函数。

上面这段官方给出的JavaScript函数的Thunk函数的含义，一开始我读了好几遍也不是很明白，接下来就一步步的了解。

**1. 一个普通的异步函数**

node中读取文件就是一个异步函数，代码如下：

```js
fs.readFile('../data/a.txt', 'utf-8', (err, data) => {
  if(err) throw err;
  console.log(data.toString());
})
```

上面代码的这个写法就是直接将三个参数都传给了fs.readFile这个方法了，并且其中的最后一个参数是一个callback函数。这种函数就是**多参数函数**。

我觉得符合多参数函数的这个概念的参数要包括以下特点：

- 前面无论有多少个参数，最后一个参数是callback函数
- 有且只能有一个callback函数参数，其余的参数是普通参数

上面对多参数函数的特点的总结是自己总结的，可能由不对的地方，因为搜索了以下没有具体对多函数参数这个概念的特点的介绍，所以根据例子来进行总结，以后如果遇到了更合适的解释会进行修改。

**2. 封装成一个符合JavaScript语言的Thunk函数**

先来看一段代码，代码很简单，一看就懂，但是具体表达的意义可能不是很清楚，下面会介绍。

```js
const fs = require('fs');

//正常的node读取文件的代码
fs.readFile('../data/a.txt', 'utf-8', (err, data) => {
  if(err) throw err;
  console.log(data.toString());
})

//封装成一个符合thunk函数的读取文件的代码
function thunk(fileName, codeType) {
  return function(callback) {
    return fs.readFile(fileName, codeType, callback);
  }
}

let readFileThunk = thunk('../data/a.txt', 'utf-8');
readFileThunk((err, data) => {
  if(err) throw err;
  console.log(data.toString());
})
```

上面是正常的文件读取和封装成Thunk函数的文件读取的代码，着重看封装成Thunk函数的代码：

- 执行let readFileThunk = thunk('../data/a.txt', 'utf-8');返回的结果是一个函数。
- readFileThunk这个函数，只能接受一个参数，而且这个参数是一个callback函数

上面Thunk函数的封装，还能写成readFileThunk()()这样的格式，这种格式会使代码看起来更加更加整洁，一目了然。要分装的函数也可以作为参数传，之前只能写死在回调函数中。

```js
function thunk(fun) {
  return function(...args) {
    return function(callback) {
      return fun.call(this, ...args, callback);
    }
  }
  
}

let readFileThunk = thunk(fs.readFile);
readFileThunk('../data/a.txt', 'utf-8')((err, data) => {
  if(err) throw err;
  console.log(data.toString());
})
```

**3. Thunk函数的特点**

根据上面的代码，通过对传统的异步函数进行封装之后，会得到一个只接受一个参数的函数，而且这个参数是一个callback函数，那这就是一个Thunk函数。就像上面的readFileThunk函数那样。

特点：

- 得到一个只接受一个参数的函数
- 这个参数只能是一个callback函数

根据上面的步骤就完成了简单的Thunk函数的封装，其实在任何函数中，只要参数有回调函数，并且只有一个回调函数，都能改写成Thunk函数的形式，下面是一个简单的Thunk函数的转换器。

```js
//ES5的写法
var Thunk = function(fun) {
  return function() {
    var args = Array.prototype.slice.call(arguments);
    return function(callback) {
      args.push(callback);
      return fun.apply(this, args);
    }
  }
}

//ES6的写法
let Thunk = function(fun) {
  return function(...args) {
    return function(callback) {
      return fun.call(this, ...args, callback);
    }
  }
}
```

### Thunkify模块

上面的Thunk函数的封装，是我们手动来做的，但是以后遇到一个有回调函数参数的函数都手动来封装吗？在这个开源时代当然不会是这样了，有一个第三方的thunkify模块，直接使用就好了。

首先安装：

> npm install thunkify --save

使用方法如下：

```js
const thunkify = require('thunkify');
const fs = require('fs');

let read = thunkify(fs.readFile);
read('../data/a.txt', 'utf-8')((err, data) => {
  if(err) throw err;
  console.log(data.toString());
})
```

其实Thunkify的源码与上面的那个简单的转换器很相似。让我们一步步写一下。

**1. 先写基础的代码**

根据上面的转换器的()()这种格式来写一下基本的代码：

```js
function makeThunkify(fun) {
  return function(...args) {
    return function(callback) {
      return fun.call(this, ...args, callback);
    }
  }
}

function f(a, b, callback) {
  let sum = a + b;
  callback(null, sum);
}

let thunkify = makeThunkify(f);
thunkify(1, 2)((err, res) => {
  console.log(res);             //undefined
});
```

上面的代码是仿照简单的Thunk函数读取文件的例子进行改写的，但是发现输出的结果是undefined。

**1. 保存上下文的问题**

上面的运行结果是undefined的原因是没有保存上下文。

```js
function makeThunkify(fun) {
  return function(...args) {
    let ctx = this;
    return function(callback) {
      return fun.call(ctx, ...args, callback);
    }
  }
}
```

**2. 捕获错误**

我上面的代码没有写对err的操作，所以不能捕获错误，需要对执行的函数进行try/catch，并且当出错的时候，传递出错误信息。

```js
function makeThunkify(fun) {
  return function(...args) {
    let ctx = this;
    return function(callback) {
      let result;
      try {
        result = fun.call(ctx, ...args, callback)
      } catch(e) {
        callback(e)
      }
      return result;
    }
  }
}

function f(a, b, callback) {
  let sum = a + b;
  throw new Error('抛出错误');
  callback(null, sum);
}

let thunkify = makeThunkify(f);
thunkify(1, 2)((err, res) => {
  if(err) {
    console.log(err);         //Error: 抛出错误
    return;
  } else {
    console.log(res);
  }
});
```

**3. 回调函数只能调用一次**

为什么回调函数只能执行一次，这样的设计是与Generator函数相关。

```js
function f(a, b, callback) {
  callback(null, 1);
  callback(null, 2);
}

let thunkify = makeThunkify(f);
thunkify(1, 2)((err, res) => {
  if(err) {
    console.log(err);         
    return;
  } else {
    console.log(res);            //1  2
  }
});
```

能抛出错误之后，又遇到了问题，上面的代码中有两个回调函数，但是回调只能调用一次，我想要的结果是只输出1，实际上输出了1和2。
所以需要判断callback函数是否已经执行过了，使其只能执行一次。

```js
function makeThunkify(fun) {
  return (...args) => {
    let ctx = this;
    return callback => {
      let result;
      var isUse;
      args.push(function() {
        if(isUse) return;
        isUse = true;
        callback.apply(null, arguments);
      }) 
      try {
        result = fun.call(ctx, ...args)
      } catch(e) {
        callback(e)
      }
      return result;
    }
  }
}

function f(fn) {
  fn(null, 1);
  fn(null, 2);
}

let thunkify = makeThunkify(f);
thunkify()((err, res) => {
  if(err) {        
    return;
  } else {
    console.log(res);            //1
  }
});
```

注意：再写上面的makeThunkify函数的时候注意arguments和箭头函数两者在使用上的问题。

到这里，基本上通过了Thunkify模块的所有测试，如果有不明白的地方可以参考源码，[Thunkify模块的源](https://github.com/tj/node-thunkify/blob/master/index.js)，算上注释不过30行左右，并且是用ES5写的，看起来比较简单。

### Generator函数的流程管理

你可能回问，Thunk函数有什么用？回答是以前确实没什么用，但是ES6有了Generator函数之后，Thunk函数现在可以用于Generator函数的自动流程管理。

以读取文件为例。下面的Generator函数封装了两个异步操作：

```js
const thunkify = require('thunkify');
const fs = require('fs');
const readFileThunk = thunkify(fs.readFile);

function* gen() {
  let r1 = yield readFileThunk('../data/a.txt');
  let r2 = yield readFileThunk('../data/b.txt');
}
```

上面的代码中，yield命令用于将程序的执行权移除Generator函数，那么就需要一种方法，将执行权交还给Generator函数。这种方法就是Thunk函数，以为他只返回回调函数，可以在callback中将执行权返回给Generator函数。为了便于理解，我们先手动执行上面这个Generator函数。

```js
let g = gen();
let res = g.next();
//先打印一下res.value，以免对下面的操作不明白
console.log(res.value);             //[Function]
res.value((err, data) => {
  if(err) return;
  console.log(data.toString());       //aaaaa
  let r2 = g.next();
  r2.value((err, data) => {
    if(err) return;
    console.log(data.toString());     //bbbbbbbbb
  })
})
```

从上面的代码中可以看出初始化Generator函数的next方法返回的res的生成器对象的value是一个函数，确切的说是一个Thunk函数。所以执行回调函数，进行文件的读取，并且在回调函数中将执行权交还给了Generator函数。

实际上，执行value方法本质上相当于注册一个回调函数，而Generator函数结合Thunk函数是一种更直观的注册回调函数的方式。Generator函数负责执行异步（交出执行权），而Thunk函数负责回调函数（交还执行权），两者结合则可以保证前者已经完成，才执行下一步。

### Thunk函数的自动流程管理

Thunk函数真正的威力，在于可以实现自动执行Generator函数。下面是一个基于Thunk函数的Generator执行器。

```js
function run(fun) {
  let g = fun();

  function next(err, data) {
    let result = g.next(data)
    if(result.done) return;
    result.value(next);
  }

  next();
}
```

上面的run函数就是Generator函数的自动执行器，内部的next函数就是Thunk函数的回调函数，next函数先将指针移到Generator函数的下一步（g.next方法），然后判断Generator函数是否结束（result.done属性），如果没有结束就将next回调函数传入result.value函数中，否则直接退出。

有了这个执行器，执行Generator函数就方便多了，不管内部有多少个异步操作，直接把Generator函数传入run函数即可。当然前提是每一个异步操作都是Thunk函数，也就是说，跟在yield命令后边的只能是Thunk函数。

Thunk函数并不是Generator函数自动执行的唯一方案。因为自动执行的关键是，必须有一种机制，自动控制Generator函数的流程，接收和交还程序的执行权。回调函数可以做到这一点，Promise对象也可以做到这一点。

## co模块

### 基本用法

co模块可以用于Generator函数的自动执行。co模块可以让你不用编写Generator函数的执行器。要注意的是co模块返回的是一个Promise对象，因此可以用then方法添加回调函数。

首先安装co模块：

> npm install co --save

使用方式：

```js
const fs = require('fs');
const co = require('co');

let readFile = function(fileName) {
  return new Promise((resolve, reject) => {
    fs.readFile(fileName, (err, data) => {
      if(err) reject(err);
      resolve(data); 
    })
  })
}

let gen = function* () {
  let r1 = yield readFile('../data/a.txt');
  let r2 = yield readFile('../data/b.txt');
}

co(gen).then(() => {
  console.log('执行完成');
});
```

上面的代码中，只要将Generator函数传入co函数，就会自动执行，上面代码，等到Generator函数执行结束，就会输出“执行完成”。

### co模块的原理

**为什么co可以自动执行Generator函数？**

前面有说过，Generator函数就是一个异步操作的容器，他的自动执行需要一种机制，当异步操作有了结果，能够自动交回执行权。

有两种方法可以做到这一点：

（1）回调函数。将异步操作包装成Thunk函数，在回调函数里面交回执行权。  
（2）Promise对象。将异步操作包装成Promise对象，用then方法交回执行权。

co模块其实就是将两种自动执行器（Thunk函数和Promise对象），包装成了一个模块。使用co前提条件就是，Generator函数的yield命令后面，只能是Thunk函数或Promise对象。如果数组或对象的成员，全部都是Promise对象，也可以使用。

上面已经介绍了Thunk函数，下面介绍一下基于Promise对象的自动执行器。

### 基于Promise对象的自动执行

在上面的例子的代码中其实已经涉及到了，将fs模块的readFile方法包装成了一个Promise对象。

```js
let readFile = function(fileName) {
  return new Promise((resolve, reject) => {
    fs.readFile(fileName, (err, data) => {
      if(err) reject(err);
      resolve(data); 
    })
  })
}
```

就是上面这部分。接下来我们不用co模块，来手动执行Generator函数。

```js
let gen = function* () {
  let r1 = yield readFile('../data/a.txt');
  let r2 = yield readFile('../data/b.txt');
  console.log(r1.toString());             //aaaaa
  console.log(r2.toString());             //bbbbbbbbb
}

let g = gen();
g.next().value.then(data => {
  g.next(data).value.then(data => {
    g.next(data);
  })
})
```

手动其实就是用then方法，层层添加回调函数。理解了这一点，就可以写出一个自动执行器。

```js
function run(gen) {
  let g = gen();

  function next(data) {
    let result = g.next(data);
    if(result.done) return;
    result.value.then((data) => {
      next(data);
    })
  }

  next();
}

run(gen)
```

### co模块的源码

co就是上面那个自动执行器的扩展，他的源码只有几十行，还是比较简单的。下面根据官文自己也试着写一下。[co模块源码](https://github.com/tj/co/blob/249bbdc72da24ae44076afd716349d2089b31c4c/index.js)

**1. co接受Generator函数作为参数，需要返回一个Promise对象**

```js
function co(gen) {
  let ctx = this;

  return new Promise((resolve, reject) => {});
}
```

**2.** 在返回的Promise对象里面，需要先检查参数gen是否为Generator函数，如果是，就执行该函数，得到一个内部指针对象；如果不是就返回，并将Promise对象的状态改为reject。

```js
function co(gen) {
  let ctx = this;

  return new Promise((resolve, reject) => {
    if(typeof gen == 'function') {
      let g = gen.call(ctx);
    }
    if(!gen || typeof gen.next != 'function') return reject(gen);
  });
}
```

**3.** 将Generator函数的内部指针对象的next方法包装成onFulfilled函数。这主要是为了捕获抛出的错误。

```js
function co(gen) {
  let ctx = this;

  return new Promise((resolve, reject) => {
    if(typeof gen == 'function') {
      let g = gen.call(ctx);
    }
    if(!gen || typeof gen.next != 'function') return reject(gen);

    function onFulfilled(data) {
      let result;
      try {
        result = g.next(data)
      } catch(e) {
        return reject(e)
      }

      next(result);
    }

    onFulfilled();
  });
}
```

**4.** 最后就是关键的next函数，他会反复调用自身

```js
function co(gen) {
  let ctx = this;

  return new Promise((resolve, reject) => {
    if(typeof gen == 'function') {
      let g = gen.call(ctx);
    }
    if(!gen || typeof gen.next != 'function') return reject(gen);

    onFulfilled();

    function onFulfilled(data) {
      let result;
      try {
        result = g.next(data)
      } catch(e) {
        return reject(e)
      }

      next(result);
    }

    function onRejected(err) {
      let result;
      try {
        result = g.throw(err)
      } catch(e) {
        return reject(e)
      }

      next(result);
    }

    function next(data) {
      if(data.done) return resolve(data.value);
      let value = toPromise.call(ctx, data.value);
      if(value && isPromise(value)) {
        return value.then(onFulfilled, onRejected);
      }
      return onRejected(new TypeError('错误信息'));
    }
  });
}
```

上面代码中的next函数中的代码，分为四部分：

第一部分：第一行，检查当前是否为Generator函数的最后一步，如果是就返回。

第二部分：第二行，确保每一步的返回值都是Promise对象。

第三部分：第三、四、五行，使用then方法，为返回值加上回调函数，然后通过onFulfilled方法再次调用next方法。

第四部分：第六行，在参数不符合要求的情况下（参数非Thunk函数或Promise对象），将Promise对象的状态改为reject，从而终止运行。实际上就是抛出错误。

### 处理并发的异步操作

co支持并发的异步操作，即允许某些操作的同时进行，等到他们全部完成，才会进入下一步。

这时，需要把并发的操作都放在数组或对象里面，跟在yield后面。

```js
const co = require('co');
//数组写法
function* gen() {
  let res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res);               //[ 1, 2 ]
}

co(gen).then(() => {});

//对象的写法
function* gen() {
  let res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2)
  };
  console.log(res);               //{ '1': 1, '2': 2 }
}

co(gen).then(() => {})
```

