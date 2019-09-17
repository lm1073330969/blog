# ES6入门---Iterator

## Iterator

首先需要说明一下遍历器这个知识点，在我通常写项目的时候遇到的时候比较少，并且只需要掌握和他有关的一小部分可能有用到过，但是这个知识点还是需要了解一下的，因为我们是为面试做准备的，但是我了解的也不会特别深入。

### Iterator的概念

Iterator是遍历器的意思，通常会把他认为是一个统一的接口，遍历器顾名思义就是在数据需要遍历的时候通常用到的。文档上也有说明，任何数据结构只要部署了iterator接口，就可以完成遍历操作。

在之前JavaScript中表示“集合”的数据结构，主要是数组和对象，ES6新增加了Map和Set两种数据结构，并且这四种数据结构可以组合使用他们，定义自己的数据结构。

Iterator的作用有三个：
- 为各种数据结构，提供一个统一的、简便的访问接口；
- 使数据结构的成员能够按照某种次序排列；
- ES6新增加了一种遍历的命令for...of循环，Iterator接口主要供for...of消费。

Iterator的遍历过程如下：  
（1）创建一个指针对象，指向当前数据结构的起始位置，也就是说遍历器对象本质就是一个指针对象。  
（2）第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。  
（3）第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。  
（4）不断调用指针对象的next方法，知道他指向数据结构的结束位置。  

每一次调用next方法，都会返回一个对象，对象里面的内容就是数据结构的当前成员的信息，对象包括两个属性value和done，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。true表示结束。

下面是一个模拟next方法返回值的例子：

```js
let itetator = makeIterator(['music', 'dance'])

function makeIterator(arr) {
  let index = 0
  return {
    next() {
      return index < arr.length
        ? { value: arr[index++], done: false }
        : { value: undefined, done: true }
    }
  }
}

console.log(itetator.next())        //{value: "music", done: false}
console.log(itetator.next())        //{value: "dance", done: false}
console.log(itetator.next())        //{value: undefined, done: true}
```

上面的这个例子定义了一个makeIterator函数，他是一个遍历器生成函数，作用就是返回一个遍历器对象，对数组执行这个函数，就会返回该数组的遍历器对象iterator。

指针的next方法，用来移动指针。开始时，指针指向数组的开始位置，然后每次调用next方法，指针就会指向数组的下一成员，第一次调用的时候指向music，第二次调用的时候指向dance。next方法返回一个对象，表示当前数据成员的信息，这个对象具有value和done两个属性，value属性返回当前位置的成员，done属性是一个布尔值，表示遍历是否结束，即是否还有必要再一次调用next方法。

总之，调用对象的next方法，就可以遍历事先给定的数据结构。

由于Iterator遍历器只是将接口部署在数据结构上面，所以遍历器与它所遍历的数据结构，实际上是分开的，完全可以写出没有任何数据结构的遍历器对象，或者说用遍历器对象模拟出数据结构。下面的例子就是一个无限循环遍历器对象的例子，下面的遍历器生成函数没有对应任何的数据结构，但是遍历器对象可以自己描述一个数据结构出来：

```js
let itetator = makeIterator()

function makeIterator() {
  let index = 0
  return {
    next() {
      return { value: index++, done: false }
    }
  }
}

console.log(itetator.next())    //{value: 0, done: false}
console.log(itetator.next())    //{value: 1, done: false}
console.log(itetator.next())    //{value: 2, done: false}
```

### 默认Iterator接口

Iterator接口的目的，就是为了所有数据结构提供一种统一的循环方式，就是ES6新增加的for...of循环，只要使用了for...of循环遍历某种数据结构时，该循环就会自动寻找Iterator接口。

如果某种数据结构部署了Iterator接口，我们就称它为“可遍历的”。

ES6规定，默认的Iterator接口，部署在数据结构的Symbol.iterator属性上，也就是说只要这个数据结构有Symbol.iterator属性，就认为是可遍历的。Symbol.iterator属性本身是一个函数，就是当前数据结构默认的遍历器函数。执行这个函数就会返回一个遍历器。

ES6的有些数据结构原生具有Iterator接口，不用做任何处理，就可以被for...of循环遍历。原因在于，这些数据结构原生部署了Symbol.iterator属性，另外有些数据结构（比如对象）没有部署Symbol.iterator属性，就不可以被for...of循环遍历。

原生具备Iterator接口的数据结构如下：

- Array
- Map
- Set
- String
- arguments对象
- NodeList对象
- TypedArray

下面的例子是数组的Symbol.iterator属性，变量array是一个数组，原生就具有遍历器接口，部署在array的Symbol.iterator属性上面，所以，调用这个属性就可以得到遍历器对象了。详见下面的例子的代码：

```js
let array = ['a', 'b', 'c'];
let iterator = array[Symbol.iterator]();

console.log(iterator.next());     //{value: "a", done: false}
console.log(iterator.next());     //{value: "b", done: false}
console.log(iterator.next());     //{value: "c", done: false}
console.log(iterator.next());     //{value: undefined, done: true}
```

对于原生部署Iterator接口的数据结构，不用自己写遍历器生成函数，for...of循环会自动遍历他们。除此之外，其他没有原生部署Iterator接口的数据结构（主要是对象），想要用for...of循环遍历，需要自己在Symbol.iterator属性上面部署遍历器生成函数。

对象数据结构之所有没有默认部署Iterator接口，是因为对象在进行循环的时候，不确定先遍历哪一个属性后遍历哪一个属性，需要开发者手动指定，本质上，遍历器是一种线性处理，对于任何非线性的数据结构，部署遍历器接口就等于部署一种线性转换。不过，严格来说，对象部署遍历器接口并不是很重要，因为这时对象实际上被当做Map结构使用，ES6原生提供了。

下面是一个为对象添加Iterator接口的例子，有了遍历器接口之后就能进行for...of循环了，如果是普通的对象不能进行循环遍历，并且上面介绍了，对象需要指定需要遍历的属性：

```js
let obj = {
  data: ['a', 'b', 'c'],
  [Symbol.iterator]() {
    const _that = this
    let index = 0
    return {
      next() {
        return index < _that.data.length
          ? { value: _that.data[index++], done: false }
          : { value: undefined, done: true }
      }
    }
  }
}

for(let value of obj) {
  console.log(value)      //a b c
}
```

**对于类数组的对象，部署Iterator接口的情况**

（1）NodeList对象是类似数组的对象，并且本来就有原生的遍历接口，可以直接进行遍历

```js
window.onload = function() {
  let nodeList = document.querySelectorAll('div')
  for(let value of nodeList) {
    console.log(value);           //<div>lalala</div>
                                  //<div>hehehe</div>
  }
}
```

（2）对于存在数值键名和length属性的类数组，部署Iterator接口时，有一个简便的方法，就是Symbol.iterator方法直接引用数组的Iterator接口。

```js
let iterator = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
}

for(let value of iterator) {
  console.log(value);       //a、b、c
}
```

注意：普通对象部署数组的Symbol.iterator方法，并无效果。

```js
let iterator = {
  a: 'a',
  b: 'b',
  c: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
}

for(let value of iterator) {
  console.log(value);       //undefined、undefined、undefined
}
```

## 调用Iterator接口的场合

有一些场合会默认调用Iterator接口。

**（1）解构赋值**

- 对数组进行解构赋值的时候，默认调用Symbol.iterator方法。

```js
let [x, y] = [1, 2];
console.log('x = ', x);       //x =  1
console.log('y = ', y);       //y =  2
```
- 对Set结构进行解构赋值的时候，默认调用Symbol.iterator方法。

```js
let set = new Set().add('a').add('b').add('c');
let [x, ...end] = set;
console.log('x = ', x);     //x =  a
console.log(end);           //["b", "c"]
```

**（2）扩展运算符**

扩展运算符（...）也会调用默认的Iterator接口。

```js
let str = 'hello';
console.log([...str]);        //["h", "e", "l", "l", "o"]
```

实际上，这提供了一种简便机制，可以将任何部署了Iterator接口的数据结构，转为数组，也就是说，只要某个数据结构部署了Iterator接口，就可以对它使用扩展运算符，将其转为数组。（所以，在学到这里的时候，也就和前面总结的如何将类数组，比如arguments、nodeList、具有length属性的类数组，转为真正的数组，其中一种方法就是用扩展运算符）

**（3）yield***

yield*后面跟的是一个可遍历的解构，他会调用该解构的遍历器接口。

```js
let generator = function* (){
  yield 1;
  yield* [2, 3, 4];
  yield 5;
}

let iterator = generator();
console.log(iterator.next());       //{value: 1, done: false}
console.log(iterator.next());       //{value: 2, done: false}
console.log(iterator.next());       //{value: 3, done: false}
console.log(iterator.next());       //{value: 4, done: false}
console.log(iterator.next());       //{value: 5, done: false}
console.log(iterator.next());       //{value: undefined, done: true}
```

**（4）数组遍历场合**

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口，比如下面：

- for...of
- Array.from
- Set()、Map()
- Promise.all()
- Promise.race()

### 字符串的Iterator的接口

字符串是一个类似数组的对象，也原生具有Iterator接口

```js
let str = 'hello';
console.log(typeof str[Symbol.iterator]);       //function
let iteratorStr = str[Symbol.iterator]();

console.log(iteratorStr.next());            //{value: "h", done: false}
console.log(iteratorStr.next());            //{value: "e", done: false}
console.log(iteratorStr.next());            //{value: "l", done: false}
console.log(iteratorStr.next());            //{value: "l", done: false}
console.log(iteratorStr.next());            //{value: "o", done: false}
console.log(iteratorStr.next());            //{value: undefined, done: true}
```

字符串可以进行改写Symbol.iterator方法，可以达到修改遍历器的目的。

下面这个例子就是改写遍历器的，注意在声明字符串的时候要用new的方法，不能用用直接声明的方式，用直接声明的方式在测试的时候发现他没有执行改写函数。具体为什么要用new而不能使用直接声明的方式，在下面有介绍。

```js
let str = new String('hello')
console.log([...str]);          //["h", "e", "l", "l", "o"]

str[Symbol.iterator] = function() {
  return {
    _key: true,
    next() {
      if(this._key) {
        this._key = false;
        return {value: 'bye', done: false};
      } else {
        return {done: true}
      }
    }
  }
}

console.log([...str]);          //["bye"]
```

上面我有提到，在声明字符串的时候要用new而不能用直接声明的方式，我查看了一些资料，介绍字符串声明的方式和区别的，https://juejin.im/post/5b70334c5188256128593f43 这篇文章我觉的介绍的还挺详细的，最起码能让我明白，如果有和我一样的不清楚为什么不能用直接声明的方式的可以看看这篇文章，我在后面也会写一篇根据这篇文章自己操作一遍的文章。

### Iterator接口与Generator函数

Symbol.iterator方法最简单的实现方式，就是用Generator函数，这个我会在下一章学习，如果你了解Generator函数可以直接向下学习，如果你不熟悉，可以选择和我一样，先空过这个知识点，先详细学习Generator函数的知识后，再回来补，我这里就是，在学完下一章之后又回来补的这个例子。

### 遍历器对象的return()，throw()

遍历器对象除了有next方法，还可以具有return方法和throw方法。如果自己写遍历器对象生成函数，那么next方法是必须部署的，return和throw方法部署是可选的。

return方法的使用场合是，如果for...of循环提前退出（通常是因为出错，或者有break语句），就会调用return方法。如果一个对象在遍历完成前，需要清理或释放资源，就可以部署return方法。

在循环遍历的过程中有两种情况会触发return函数：

- 在循环遍历中使用break语句；
- 在循环遍历中抛出错误（比如：throw new Error()）

注意：在自己写遍历器对象生成函数的return函数的时候，return函数一定要返回一个对象。这是Generator规格决定的。

throw方法主要是配合Generator函数使用，一般的遍历器对象用不到这个方法。




