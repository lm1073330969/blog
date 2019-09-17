# ES6入门---Promise对象

Promise对象一般我在实际的项目中用的时候，他的优点确实很大，通常我也是在调接口的时候用到的比较多，比如有的需求是先要拿到用户的id才能获取用户发布过的信息，这时候就需要确保先要拿到id才能调下一个接口。

所以我总觉得没有用过promise或不动手敲，只是学习他的文档没有什么大的收获，也感受不到他的优点，我系统的学习他的文档更多的是因为面试官总会问一个问题，能用ES5写一个promise吗？

所以我还是觉得多在项目中用，才能更好的掌握promise，如果你也是为了面试可能需要更标准的文档，这个就得自己慢慢理解，下面的总结的也不过是一部分。

## 概念

Promise是异步编程的一种解决方案，比传统的解决方法（回调函数、事件监听）更加合理和强大，因为传统的比如回调函数存在回调地狱的问题。

Promise是一个对象，他可以获取异步操作的消息。

**Promise对象的两个特点：**

1. 对象的状态不收外界影响。Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）、rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，他英语的意思是“承诺”，表示其他手段无法改变。
2. 一旦状态改变，就不会再变化，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled、从pending变为rejected。只要是这两种情况的其中一种发生了，状态就凝固了，不会再发生变化，会一直保持这个结果。如果改变已经发生了，再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（event）完全不同，如果你错过了它，再去监听，是得不到结果的。

**Promise优点**

有了Promise对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，Promise对象提供统一的接口，使得控制异步操作更加容易。

**缺点**

- 无法取消Promise，一旦新建它就会立即执行，无法中途取消。
- 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。
- 处于pending状态时，无法得知目前进展带哪一个阶段。

## 用法

ES6规定，Promise对象是一个构造函数，用来生成Promise实例。

Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。他们是两个==函数==，由JavaScript引擎提供，不用自己部署。

**创造Promise实例的例子**

```js
let promise = new Promise((resolve, reject) => {
    if('成功') {
        //异步操作成功返回的结果
        resolve();
    } else {
        //失败的结果
        reject();
    }
}) 
```

- resolve函数的作用：将Promise对象的状态从“未完成”变成“成功”（从pending变为resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去。
- reject函数的作用：将Promise对象的状态从“未完成”变为“失败”（从pending变为rejected），在异步操作失败的时候调用，并将异步操作报出的错误，作为参数传递出去。

Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

then方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为resolved时调用，第二个回调函数是Promise对象的状态变为rejected时调用。其中第二个函数是可选的，不一定提供。这两个函数都接受Promise对象传出的值作为参数。

```js
promise.then((res) => {
    //第一个回调函数，成功返回的结果
    console.log(res)
}, (err) => {
    //第二个回调函数，失败的结果，这个是可选的
    console.log(err);
})
```

下面的例子中，timeout方法返回一个Promise实例，表示一段时间以后才会发生结果，过了指定时间之后，Promise实例的状态变为resolved，就会触发then方法绑定的回调函数。

```js
function timeout(time) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, time, '延迟');
    })
}

timeout(1000).then(res => {
    console.log(res);           //1秒之后打印：延迟
})
```

Promise新建后就会立即执行，从下面的例子中可以看出，最先输出的是“promise”，所以Promise新建之后立即执行，然后then方法指定的回调函数会在当前脚本所有同步任务执行完成之后才会执行，所以就是“同步”，然后是“异步结果”。

```js
let promise = new Promise((resolve, reject) => {
    console.log('promise');
    return resolve();
})

promise.then(res => {
    console.log('异步结果');
})
console.log('同步');

//promise
//同步
//异步结果
```

一开始看见文档中的例子之后，我测试了上面的代码之后确实是promise最先输出，我就想是因为我下面调用之后，还是在我一声明的时候就执行了呢，所以我又测试了一下，得出的结论就是在我声明的时候promise就执行了，看下面的例子：

```js
let promise = new Promise((resolve, reject) => {
    console.log('promise');
    return resolve();
})
//promise
```

**用Promise对象实现Ajax操作的例子**

```js
let getJSON = function(url) {
    return new Promise((resolve, reject) => {
        let handler = function() {
            if(this.readyState != 4) {
                return;
            }
            if(this.status === 200) {
                return resolve(this.response);
            } else {
                reject(new Error(this.statusText))
            }
        }
        let xml = new XMLHttpRequest();
        xml.open('GET', url);
        xml.onreadystatechange = handler;
        xml.responseType = 'json';
        xml.setRequestHeader('Accept', 'application/json');
        xml.send();
    })
}

getJSON("/posts.json").then(res => {
    console.log(res);
})
.catch(err => {
    console.log(err);
})
```

## Promsie.prototype.then()

Promise实例具有then方法，也就是说。then方法是定义在原型对象Promise.prototype上的。他的作用是为Promise实例添加状态改变时的回调函数。

上面说过，then方法可以接受两个回调函数的参数，第一个参数是resolved状态的回调函数，第二个参数（可选）是rejected状态的回调函数。

then方法返回的是一个新的Promise实例（注意：不是原来的那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。

采用链式的then，可以指定一组按照次序调用的回调函数，这时，前一个回调函数，有可能返回的还是一个Promise对象（有异步操作），这时后一个回调函数，就会等该Promise对象的状态发生变化，才会被调用。

下面的例子依次指定了三个回调函数。只有当第一个回调函数完成以后，将返回结果作为参数，传入第二个回调函数，第二个回调函数的返回结果传入第三个回调函数。

```js
let state = 1;

function step1() {
    return new Promise((resolve, reject) => {
        console.log('1.开始洗菜做饭');
        if(state === 1) {
            resolve('洗菜做饭-完成');
        } else {
            reject('洗菜做饭-失败')
        }
    })
}

function step2() {
    return new Promise((resolve, reject) => {
        console.log('2.吃饭');
        if(state === 1) {
            resolve('吃饭-完成');
        } else {
            reject('吃饭-失败')
        }
    })
}

function step3() {
    return new Promise((resolve, reject) => {
        console.log('3.洗碗');
        if(state === 1) {
            resolve('洗碗-完成');
        } else {
            reject('洗碗-失败')
        }
    })
}

step1().then(res => {
    console.log(res);
    return step2();
}).then(res => {
    console.log(res);
    return step3();
}).then(res => {
    console.log(res);
})

//1.开始洗菜做饭
//洗菜做饭-完成
//2.吃饭
//吃饭-完成
//3.洗碗
//洗碗-完成
```

## Promsie.prototype.catch()

Promsie.prototype.catch方法是用于指定发生错误时的回调函数。

一般来说，不建议在then方法的第二个回调参数里面定义Reject状态的回调函数，建议使用catch方法，自己几乎从没有写过then的第二个参数，都是用catch方法。

下面是两种reject状态的写法，第二种写法要好于第一种写法，理由是第二种写法可以捕获前面then方法执行中的错误，也更接近同步的写法（try/catch）。因此，建议总是使用catch方法，而不是用then方法的第二个参数。

```js
//第一种
promise.then(res => {
    //resolved成功状态
    console.log(res);
},(err) => {
    //第二个回调函数，rejected状态
    console.log(err);
})

//第二种
promise.then((res) => {
    //成功状态
    console.log(res);
}).catch((err) => {
    //捕获错误
    console.log(err);
})
```

Promise.property.catch的catch方法和传统的try/catch代码块不同，如果没有使用catch方法指定错误处理的回调函数，Promise对象抛出的错误不会传递到外层，即不会影响外面的代码运行。

```js
let promise = function() {
    return new Promise((resolve, reject) => {
        resolve(x + 2);
    })
}

promise().then(res => {
    console.log(res);
})

setTimeout((x = 1, y = 2) => {
    console.log(x + y);
}, 2000);

/**
 * 运行的结果：
 * 首先会promise对象会抛出一个错误，因为在promise对象的resolve返回中没有x，所以报错信息：Uncaught (in promise) ReferenceError: x is not defined
 * 但是抛出的错误没有影响外面的setTimeout的运行，在2秒之后输出了x + y 的结果3
 */
```

上面的例子也通常被称为“Promise 会吃掉错误”。就是说，Promise内部的错误不会影响到Promise外部的代码。

一般建议在Promise对象处理成功的then后面要跟上catch方法，这样就可以处理Promise内部发生的错误。catch方法返回的还是一个Promise对象，因此后面还可以接着调用then方法。

下面的例子中用catch方法捕获Promise中抛出的错误，catch方法捕获了错误，没有影响后面的then方法的执行：

```js
let promise = function() {
    return new Promise((resolve, reject) => {
        resolve(x + 2);
    })
}

promise().catch((err) => {
    console.log(err);
}).then(res => {
    console.log('success');
})

//ReferenceError: x is not defined
//success
```

如果Promise对象中没有抛出错误则会跳过catch方法，直接执行then方法，如果此时then方法里面再有报错，就与前面的catch无关：

```js
let promise = function() {
    return new Promise((resolve, reject) => {
        resolve(2);
    })
}

promise().catch((err) => {
    console.log(err);
}).then(res => {
    console.log('success', res);
})

//success 2
```

## Promise.prototype.finally()

finally方法用于指定不管Promise对象最后状态如何，都会执行的操作。

不管promise最后的状态，在执行完then或catch指定的回调函数以后，都会执行finally方法指定的回调函数。看下面的一个简单的例子：

```js
let promise = function() {
    return new Promise((resolve, reject) => {
        resolve(2);
    })
}

promise().catch((err) => {
    console.log(err);
}).then(res => {
    console.log('success', res);
}).finally(() => {
    console.log('finally');
})

//success 2
//finally
```

finally方法的回调函数不接受任何参数，这意味着没有办法知道，前面的Promise状态到底是fulfilled还是rejected。这表明，finally方法里面的操作，应该是与状态无关的，不依赖于Promise的执行结果。

## Promise.all()

Promsie.all方法用于将多个Promise实例，包装成一个新的Promise实例。

Promise.all方法接受一个数组作为参数，p1、p2、p3都是Promise实例。（Promise.all方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise实例）。

```js
let promise = Promise.all([p1, p2, p3]);
```

promise的状态由p1、p2、p3决定，分成两种情况：

- 只有p1、p2、p3的状态都变成fulfilled，promise状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给promise的回调函数。
- 只要p1、p2、p3之中有一个被rejected，promise的状态就变成了rejected，此时第一个被rejected的实例的返回值，会传递给promise的回调函数。

看下面的一个例子，promise1、promise2两个异步操作，只有等到他们的结果都返回了，才会触发

```js
let status = 1;

let promise1 = new Promise((resolve, reject) => {
    if(status === 1) {
        return resolve('promise1-完成');
    } else {
        return reject('promise1-失败');
    }
})

let promise2 = new Promise((resolve, reject) => {
    if(status === 1) {
        return resolve('promise2-完成');
    } else {
        return reject('promise2-失败');
    }
})

Promise.all([promise1, promise2]).then(res => {
    console.log('res->', res);              //res-> (2) ["promise1-完成", "promise2-完成"]
})
```

## Promise.race()

Promise.race方法同样是将多个Promise实例，包装成一个新的Promise实例。

但是Promise.race和all方法的不同之处就是race方法只要有一个实例率先改变状态，结果的状态也就跟着改变了。那个率先改变的Promise实例的返回值，就是传递给结果的返回值。

下面的例子，是上面all方法例子的一个改写，能看出两个方法的差别，promise1率先改变了状态，结果就是promise1返回的结果，只要有一个改变了，就会输出结果：

```js
let status = 1;

let promise1 = new Promise((resolve, reject) => {
    if(status === 1) {
        return resolve('promise1-完成');
    } else {
        return reject('promise1-失败');
    }
})

let promise2 = new Promise((resolve, reject) => {
    if(status === 1) {
        return resolve('promise2-完成');
    } else {
        return reject('promise2-失败');
    }
})

Promise.race([promise1, promise2]).then(res => {
    console.log('res->', res);              //res-> promise1-完成
})
```

## Promise.resolve()

有时候需要将一个对象转为Promise对象，Promise.resolve方法就起到这个作用。

**Promise.resolve等价于下面的写法：**

```js
Promise.resolve('name');
//等价于
new Promise((resolve, reject) => resolve('name'));
```

**Promise.resolve方法的参数分成四种情况：**

**1.参数是一个Promise实例**

如果参数是一个Promise实例，那么Promise.resolve将不做任何改变，原封不动的返回这个实例。

```js
let promise = function() {
    return new Promise((resolve, reject) => {
        resolve('success');
    })
}

console.log(Promise.resolve(promise));          //Promise
```

**2.参数是一个thenable对象**

thenable对象指的是具有then方法的对象，比如下面这个对象：

```js
let thenable = {
    then(resolve, reject) {
        resolve('thenable obj');
    }
}
```

Promsie.resolve方法会将这个对象转为Promise对象，然后立即执行thenable对象里面的then方法：

```js
let thenable = {
    then(resolve, reject) {
        resolve('thenable obj');
    }
}

let promise = Promise.resolve(thenable);
console.log(promise);               //Promise {<pending>}
                                    //[[PromiseStatus]]: "resolved"
                                    //[[PromiseValue]]: "thenable obj"

promise.then(res => {
    console.log(res);               //thenable obj
})
```

**3.参数不是具有then方法的对象，或根本就不是对象**

如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为resolved。并且回调函数会立即执行。Promise.resolve方法的参数，会同时传给回调函数。

参数不是一个对象，是一个字符串时：

```js
let promise = Promise.resolve('hello');
console.log(promise);               //Promise {<resolved>: "hello"}
                                    //[[PromiseStatus]]: "resolved"
                                    //[[PromiseValue]]: "hello"
```

参数是一个不带then方法的对象时：

```js
let obj = {
    name: 'lee',
    age: 22
};

let promise = Promise.resolve(obj);
console.log(promise);               //Promise
                                    //[[PromiseStatus]]: "resolved"
                                    //[[PromiseValue]]: Object
                                    //{age: 22, name: "lee"}
promise.then(res => {
    console.log(res)            //{name: "lee", age: 22}
})
```

**4.不带任何参数**
Promise.resolve方法允许调用时不带参数，直接返回一个resolved状态的Promise对象。

所以，如果希望得到一个Promise对象，比较方便的就是直接调用Promise.resolve()方法。

需要注意的是，立即resolve的promise对象，是在本轮“事件循环（event loop）”的结束时执行，不是在下一轮“事件循环”的开始时。其实这是官方文档的说法。

因为resolve方法就是方便得到promise实例，其实我自己的理解就是根据事件循环的规则，在遇到promise实例的时候，promise构造函数的第一个参数，是在new的时候执行，上面其实也有介绍，resolve方法的和用new的等价，用new的时候只不过第一个参数就执行了，不会进入任何队列，是直接在当前任务执行了，后续的then则会被分发到微队列的promise队列中去，然而直接用resolve生成promise实例和用new也没有什么区别。但是文档上说的在本轮的结束时执行确实没有怎么很明白，因为从下面的这个例子也只能说明和用new生成的promise实例没差别。

下面就分别用resolve和new方法来执行一下，我们看看：

这是用resolve的：

```js
setTimeout(() => {
    console.log('three');
}, 0);

Promise.resolve('two').then(res => {
    console.log(res);
})

console.log('one');
//one
//two
//three
```

这是用new的：

```js
setTimeout(() => {
    console.log('three');
}, 0);

let promise = new Promise((resolve, reject) => {
    resolve('two')
})

promise.then(res => {
    console.log(res);
})

console.log('one');
//one
//two
//three
```

我们可以看上面的两个例子，分别用resolve和new的方法，但是结果是一样的，没有看出用resolve的差别，所有我觉得官方文档中说的在本轮事件循环结束时执行的意思就是事件循环的宏任务和微任务，resolve方法是在等宏任务执行完成之后，才执行微任务，和promise的没有差别。

下面可以分析一下事件循环的顺序，根据上面的用new创建的实例：

首先，事件循环从宏任务队列开始，这个时候，宏任务队列中，只有script（整体代码）任务。每一个任务的执行顺序，都依靠函数调用栈来搞定，而当遇到任务源时，则会先分发任务到对应的队列中去，所以，上面代码的例子的第一步执行如下图所示：

首先script任务开始执行，全局上下文入栈

![ZnO9MD.png](https://s2.ax1x.com/2019/06/27/ZnO9MD.png)

第二步：script任务执行时首先遇到setTimeout，setTimeout为一个宏任务源，任务源的作用就是将任务分发到它对应的队列中。

```js
setTimeout(() => {
    console.log('three');
}, 0);
```

宏任务three进入setTimeout队列：

![ZnX2hq.png](https://s2.ax1x.com/2019/06/27/ZnX2hq.png)

第三步：script执行时遇到promise实例。Promise构造函数中的第一个参数，是new的时候执行，因此不会进入人格队列，而是直接在当前任务直接执行了，我们上面的例子第一个参数中没有打印任何东西，所以就没有输出，后续的then的执行会被分发到微队列的Promise队列中去。

执行Promise实例时，resolve入栈：

![ZnjsxK.png](https://s2.ax1x.com/2019/06/27/ZnjsxK.png)

构造函数执行完毕的过程中，resolve执行完毕出站，then执行时，Promise任务two进入对应的队列。

![ZnvhmF.png](https://s2.ax1x.com/2019/06/27/ZnvhmF.png)

script任务继续往下执行，最后只有一句输出了one，然后，全局任务就执行完毕了。

第四步：第一个宏任务script执行完毕后，就开始执行所有的可执行的微任务。这个时候，微任务中，只有Promise队列中的一个任务two，因此直接执行就可以了，执行结果输出two，当然他的执行，也是进入函数调用栈中执行的。

![Znvv0e.png](https://s2.ax1x.com/2019/06/27/Znvv0e.png)

第五步：当所有的微任务执行完毕之后，表示第一轮的循环就结束了，这个时候就得开始第二轮的循环。第二轮循环仍然从宏任务开始，这个时候我们发现宏任务中，只有setTimeout队列中还有一个three的任务等待执行。因此直接执行就可以了。

three入栈执行：

![Znx1XT.png](https://s2.ax1x.com/2019/06/27/Znx1XT.png)

这时候我门可以看见宏任务队列与微任务队列中都没有任务了，所以整个循环就结束了。这是一个简单的例子。

关于事件循环的执行的过程或者是相关的知识还没有明白的话，还会有一篇关于事件循环的专门的学习笔记。那个里面可能介绍的比较详细。在这个里面之所以提到事件循环，是因为关于官方的例子中有写到，而且感觉和我理解的有偏差所以就试着验证，简单介绍一下，如果对于事件循环的知识了解的话可以略过这段知识的介绍。

## Promise.reject()

Promise.reject(reason)方法也会返回一个新的Promise实例，该实例的状态为失败rejected。

下面是一个Promise对象的实例promise，状态为rejected，回调函数会立即执行。

```js
let promise = Promise.reject('出错啦！');

promise.then(res => {
    //因为上面定义的失败的状态rejected，所以不会输出
    console.log(res);
}).catch(err => {
    //catch方法是建议的写法，可以捕获promise实例抛出的错误，失败状态也算
    console.log(err);           //出错啦！
})
```

Promise.reject方法和new是等价的：

```js
let promise = Promise.reject('出错啦！');

//等同于
let promise = new Promise((resolve, reject) => {
    reject('出错啦！')
})
```

注意，Promise.reject()方法的参数，会原封不动的作为reject的理由，变成后续方法的参数。这一点与Promise.resolve方法不一致。

下面这个例子reject方法的参数是一个thenable对象，执行以后，后面的catch方法的参数不是reject抛出的“出错了”这个字符串，而是thenable对象。

```js
let thenable = {
    then(resolve, reject) {
        reject('出错啦！');
    }
}

Promise.reject(thenable).catch(err => {
    //不是上面抛出的错误信息“出错了”，而是thenable整个对象
    console.log(err);               //{then: ƒ}
})
```

## Promise.try()

其实我自己也没有用到过Promise.try方法，他就是模拟try代码块。

```js
//可以捕获里面调取接口可能遇到的错误，比如数据库连接抛出的错误
Promise.try(() => database.users.get({id: userId}))
.then()
.catch()
```










