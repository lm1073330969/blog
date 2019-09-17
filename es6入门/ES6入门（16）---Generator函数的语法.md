# ES6入门---Generator函数的语法

## 简介

Generator函数是ES6提供的一种解决异步编程的方法，我习惯将Generator函数叫成生成器，因为generator翻译过来就是生产的意思。

执行Generator函数会返回一个遍历器对象，我们在之前学习Iterator的时候，也有了解，最简单的遍历器对象的写法就是用Generator函数来写。

Generator函数的两个特征：

- function关键字和函数名之间有一个*；
- 函数体内部使用yield表达式，定义不同的状态（yield在英语里是“产出”的意思）

Generator函数的定义：

```js
function* gen() {
  yield 'hello';
  yield 'ES6';
  return 'ending';
}

let g = gen();
```

Generator函数的调用和普通的函数一样，在函数名后面加上一对圆括号。不同的是，调用Generator函数之后，该函数并不会执行，也不会返回函数的运行结果，而是一个指向内部状态的指针对象，这句话有没有感觉很熟悉，在之前学习的Iterator的时候，介绍Iterator运行的实质的时候有说过Iterator的实质也是指向内部状态的指针对象。

所以Generator函数和Iterator还是很相似的，也是运用next方法，来将指针移到下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield（或return语句）为止。换言之，Generator函数是分段执行的，yield是暂停执行的标记，而next方法可以恢复执行。

```js
console.log(g.next());          //{value: "hello", done: false}
console.log(g.next());          //{value: "ES6", done: false}
console.log(g.next());          //{value: "ending", done: true}
console.log(g.next());          //{value: undefined, done: true}
```
Generator函数，返回一个遍历器对象，代表Generator函数的内部指针，只用每调用遍历器对象的next方法，就会返回一个遍历器对象，里面的属性是value和done，value表示当前的内部状态的值，也就是yield表达式后面那个表达式的值。done属性是一个布尔值，代表是否遍历结束。

上面的代码和解释有没有感到很熟悉，没错，要是学习过之前的Iterator的知识，那一定不会感到陌生的，因为他也是这样的含义。

注意：ES6没有规定，function关键字和函数名之间的星号，写在具体哪个位置，所以导致下面的写法都能通过，但是建议的写法是星号紧跟在function关键字之后。

```js
function * gen() {...}
function* gen() {...}     //建议的写法
function *gen() {...}
function*gen() {...}
```

### yield表达式

由于Generator函数返回的遍历器对象，只有调用next方法才会遍历下一个内部状态，所以是提供了一种可暂停执行的函数。yield表达式就是暂停标志。

遍历器对象的next方法的运行逻辑如下：

（1）遇到yield表达式，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值。  
（2）下一次再调用next方法时，再继续往下执行，直到遇到下一个yield表达式。  
（3）如果没有再遇到新的yield表达式，就一直运行到函数的结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。  
（4）如果该函数没有return语句，则返回的对象的value属性值为undefined。 

需要注意的是，yield表达式后面的表达式，只有当调用next方法的时候、内部指针指向该语句的时候才会执行，因此等于为JavaScript提供了手动的“惰性求值”的语法功能。

```js
function* gen() {
  yield 123 + 456;
}

//像普通的函数那样直接调用。Generator函数其实是没有执行的。只有执行遍历器的next方法指向表达式的时候才会执行
console.log(gen());         //返回的是这么一个对象： gen {<suspended>}

//正确的执行方式
let g = gen();
console.log(g.next());      //{value: 579, done: false}
```

**yield表达式和return表达式：**

yield表达式和return表达式既有相似之处，也是有区别的。

相似之处：都能返回紧跟在语句后面的那个表达式的值。

区别：区别在于每次遇到yield，函数暂停执行，下一次再从该位置继续向后执行，而return语句不具备位置记忆的功能。在一个函数里面只能执行一次return语句，但是可以执行多次yield语句。正常的函数只能返回一个值，因为只能执行一次return语句。但是Generator函数可以返回一系列的值，因为可以执行任意多个yield。

**Generator函数变成一个单纯的暂缓执行函数：**

如果是普通的函数，再被调用或为变量赋值的时候就会执行了，但是Generator函数不会立即被执行，如果Generator函数里面不用yield表达式，还可以作为缓冲执行函数。

```js
function* gen() {
  console.log('缓冲执行函数');
}

let g = gen();

setTimeout(function() {
  g.next();         //运行两秒之后输出 “缓冲执行函数”
}, 2000);
```

需要注意的是，yield表达式只能在Generator函数中使用，用在其他的地方都会报错。

下面这个例子就会报错，虽说定义的是Generator函数，但是函数里面有forEach方法，在forEach里面用yield表达式会报错，因为forEach方法的参数是一个普通函数，yield表达式只能用在Generator函数里面。

```js
let array = [1, [[2, 3], 4], 5];

let gen = function* (arr) {
  arr.forEach(element => {
    if(typeof element != 'number') {
      yield* gen(element);
    } else {
      yield element;
    }
  })
}

for(let value of gen(array)) {
  console.log(value);
}
//报错信息：Unexpected identifier
```

上面这种情况，可以使用普通的for循环来替代forEach循环，for循环不会报错。

```js
let array = [1, [[2, 3], 4], 5]

let gen = function*(arr) {
  for (let i = 0; i < arr.length; i++) {
    if (typeof arr[i] != 'number') {
      yield* gen(arr[i])
    } else {
      yield arr[i]
    }
  }
}

for (let value of gen(array)) {
  console.log(value);               //1  2  3  4  5
}
```

注意：如果yield表达式用在另一表达式之中，必须放在括号之中，yield表达式如果作为函数的参数或赋值表达式的右边，可以不加括号。：

```js
function* gen() {
  console.log('hello' + yield);             //报错：Unexpected identifier
  console.log('hello' + yield 123);         //报错：Unexpected identifier
  
  console.log('hello' + (yield));           //没有报错
  console.log('hello' + (yield 123));       //没有报错
  
  //作为参数
  demo(yield 'a', yield 'b');
  //放在赋值语句的右边
  let str = yield 'hello';
}
```

### 与Iterator接口的关系

在前面学习Iterator的时候，知道了任意一个对象的Symbol.iterator方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。

上面有提到过，Generator函数就是遍历器生成函数，因此可以把Generator函数赋给对象的Symbol.iterator属性，从而使得该对象具有Iterator接口。

```js
let obj = {}
obj[Symbol.iterator] = function* () {
  yield 'name';
  yield 'age';
  yield 'skill';
  return 'ending';
};

console.log([...obj]);            //["name", "age", "skill"]
```

## next方法的参数

yield表达式本身没有返回值，或者说总是返回undefined，next方法可以带一个参数，该参数会被当做上一个yield表达式的返回值。

通过下面这个小例子先来初步的了解一下next方法的参数：

```js
function* gen() {
  for(let i = 0; true; i++) {
    let result = yield i;
    if(result) {
      i = 10;
    }
  }
}

let g = gen();
console.log(g.next());          //{value: 0, done: false}
console.log(g.next());          //{value: 1, done: false}

console.log(g.next(true));         //{value: 11, done: false}
console.log(g.next());          //{value: 12, done: false}
```

上面的这个例子，在Generator函数gen中是一个可以无限循环的循环，如果next方法没有传参数的情况下会按照一开始设定的从0开始循环，一直循环下去没有结束的条件。为什么会这样呢，为什么执行不到if语句里面呢，是因为yield表达式没有返回值，调用next方法，他遇到yield就会暂停执行，会将yield表达式后面紧跟的值作为返回值返回，result的值是undefined，所以没有执行if语句。

要是想要让if语句执行，需要在next方法中传一个参数，这个参数会被当做上一个yield表达式的返回值。上面我传了一个true，也就是现在的result有值了，值为true，所以if语句其作用了，将i重置为10，那么就会从10进行开始循环。

这个功能有很重要的语法意义，Generator函数从暂停状态到恢复运行，他的上下文状态是不变的。通过next方法的参数，就有办法在Generator函数开始运行之后，继续向函数体内注入值。也就是说，可以在Generator函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。

看下面的一个例子：

```js
function* gen(x) {
  let y = 2 * (yield (x + 1));
  let z = yield (y / 3);
  return (x + y + z);
}

//next方法不带参数
let g = gen(5);
console.log(g.next());          //{value: 6, done: false}
console.log(g.next());          //{value: NaN, done: false}
console.log(g.next());          //{value: NaN, done: true}

//next方法带参数
let g1 = gen(5);
console.log(g1.next());         //{value: 6, done: false}
console.log(g1.next(12));       //{value: 8, done: false} 
console.log(g1.next(13));       //{value: 42, done: true}
```

上面的例子中有两种方式，一种是next方法不带参数的方式，可以看到当他第二次调用next方法的时候，返回的值为NaN，是因为，默认的第一次使用next方法是启动Generator函数，第二次调用的时候没有带参数，y的值是 2 * undefined，结果是NaN，所以除以3之后是NaN，第三次调用的时候也没有带参数，所以z等于undefined，最后返回的结果为 5 + NaN + undefined，等于NaN。

第二种方式是next方法带参数的方式，next方法的参数是表示上一个yield表达式的返回值，所以，第一次没有带参数执行next方法，是启动Generator函数，返回的值为6，因为x等于5，返回x + 1的结果。第二次执行next方法的参数为12，就是将上一次yield表达式的值设为12，所以y等于2 * 12，结果为24。z就等于24除以3的结果8，所以返回的值就为8，第三次next，参数值为13， 相当于将yield (y / 3)的值设为13赋给z，所以z的值就位13，最后的结果就是 5 + 24 + 13相加的结果。

注意：由于next方法的参数表示上一个yield表达式的返回值，所以在第一次使用next方法时，传递参数是无效的，V8引擎直接忽略第一次使用next方法时的参数，只有从第二次使用next方法开始，参数才是有效的。从语义上讲，第一个next方法是用来启动遍历器对象，所以不用带参数。

## for...of循环

for...of循环在之前也有提到过，想必也不会很陌生，用for...of循环的时候只有部署了Iterator接口才能用，Generator函数是Iterator接口的生成器对象，所以for...of循环可以遍历Generator函数，再运行时生成Iterator对象，并且不需要用next方法。

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  return 5;
}

for(let value of gen()) {
  console.log(value);       //1  2  3  4
}
```

上面的代码，用for...of循环，依次输出yield表达式的值，但是有一点需要注意，for...of循环能输出next方法返回的对象的done属性值是false的，遇到done值为true的时候就停止循环，并且不会返回此时对象的value属性的值。

我们在之前有一节介绍for...of循环和其他循环的区别的时候，有介绍过for...of循环在遍历对象上面功能不是很全面和for...in相比还是有差别的，但是我有介绍符合键值是数值并且包含length属性的对象的部署Iterator接口的方法，没有过多的介绍普通对象怎么部署Iterator接口的方法，下面我用Generator函数来写一下怎么可以用for...of遍历出任意对象的方法。

下面的代码中普通对象obj不具有Iterator接口，无法用for...of循环，我们通过Generator函数objectEntries为他加上遍历器接口，所以普通对象就可以用for...of遍历了。这是其中一种写法。

```js
function* objectEntries(obj) {
  let keys = Reflect.ownKeys(obj);

  for(let key of keys) {
    yield [key, obj[key]];
  }
}

let obj = {
  'name': 'lee',
  'age': 3,
  'skill': 'web'
}

for(let item of objectEntries(obj)) {
  console.log(item);          //["name", "lee"]
                              //["age", 3]
                              //["skill", "web"]
}
```

为普通对象加上遍历器接口，还有一种写法，就是将Generator函数加到对象的Symbol.iterator属性上面。

```js
function* objectEntries() {
  //如果用Reflect.ownKeys,
  // let keys = Reflect.ownKeys(this);       //最后输出的结果会将[Symbol(Symbol.iterator), ƒ]输出
  
  //注意要用Object.keys用对象自身的key
  let keys = Object.keys(this);

  for(let key of keys) {
    yield [key, this[key]];
  }
}

let obj = {
  'name': 'lee',
  'age': 3,
  'skill': 'web'
}

//将Generator函数加到对象的Symbol.iterator属性上面
obj[Symbol.iterator] = objectEntries;
for(let item of obj) {
  console.log(item);          //["name", "lee"]
                              //["age", 3]
                              //["skill", "web"]
}
```

除了for...of循环之外，扩展运算符、解构赋值、Array.from方法内部调用的都是遍历器接口，这意味着，他们也都可以操作Generator函数。

```js
function* gen() {
  yield 1;
  yield 2;
  return 3;
  yield 4;
}

//for...of循环
for(let value of gen()) {
  console.log(value);           //1  2
}
//扩展运算符
console.log([...gen()]);        //[1, 2]
//解构赋值
let [x, y] = gen();
console.log('x = ', x);         //x =  1
console.log('y = ', y);         //x =  1
//Array.from函数
console.log(Array.from(gen())); //[1, 2]
```

## Generator.prototype.throw()

Generator函数返回的遍历器对象都有一个throw方法，可以在函数体外抛出错误，然后在Generator函数体内捕获。

```js
function* gen() {
  try {
    yield 1;
  } catch(e) {
    console.log('内部捕获', e);
  }
}

let g = gen();
//注意：要想被Generator函数内部捕获，需要先启动Generator函数，如果忘记了，就只会有外部捕获
g.next();

try {
  g.throw('a');
  g.throw('b');
} catch(e) {
  console.log('外部捕获', e);
}

//内部捕获 a
//外部捕获 b
```

上面的代码中，遍历器对象g连续抛出了两个错误，第一个错误被Generator函数体内的catch语句捕获，第二次抛出的错误，由于抛出第一次的时候已经将Generator函数内的catch语句已经执行过了，所以就不会再捕获到第二次的错误，所以这个错误被抛出了Generator函数体内，被函数体外的catch语句捕获到了。

注意：  
（1）如果没有用next方法启动Generator函数，第一个错误就不能被Generator函数捕获，会被Generator函数体外的catch语句捕获。

（2）并且，throw方法可以接受一个参数，该参数会被catch语句接收，所以建议抛出Error对象的实例。

（3）不要混淆遍历器对象的throw方法和全局的throw语句。throw命令抛出的错误只能被Generator函数体外的catch语句捕获。

（4）如果Generator函数体内没有部署try...catch代码块，那么就算是遍历器对象的throw方法抛出的错误，也将会被Generator函数体外的catch语句捕获。

（5）如果Generator函数体内和体外均没有写try...catch代码块，那么程序将报错，直接中断执行。

（6）Generator函数返回的遍历器对象的throw方法抛出的错误想要被Generator函数的内部捕获，前提是必须至少执行过一次next方法。这个在上面的代码中也有体现和说明。

（7）遍历器对象的throw方法被捕获以后，会附带执行下一条yield表达式。也就是说会附带执行一次next方法。

```js
function* gen() {
  try {
    yield 1;
  } catch(e) {
    console.log('内部捕获', e);
  }
  yield 3;
  yield 4;
}

let g = gen();
console.log(g.next());        //{value: 1, done: false}
console.log(g.throw(2));      //内部捕获 2
                              //{value: 3, done: false}
console.log(g.next());        //{value: 4, done: false}
```

上面的代码，第一次执行next方法的时候，正确的输出了yield表达式的返回值，接下来就执行了throw方法，会被gen函数里面的catch语句捕获，但是在输出的结果很明显的看出来了，不光打印了catch语句里面的内容，同时自动执行了一次next方法。所以第二次执行的next方法变成了正常情况下第三次执行next方法的结果了。但是从这点可以看出遍历器的throw和全局的throw的区别，遍历器对象的throw方法执行完成之后，不会影响下一次的遍历

（8）遍历器对象的throw方法执行完成之后，不会影响下一次的遍历。

这种函数体内捕获错误的机制，大大方便了对错误的处理，多个yield表达式，可以只用一个try...catch代码块来捕获错误

Generator函数体外用遍历器对象的throw方法抛出的，可以在Generator函数体内捕获，反过来，函数体内抛出的错误，也可以被函数体外的catch语句捕获。

但是，一旦Generator函数在执行过程中抛出错误，并且没有被内部捕获，就不会执行下去了，如果在抛出错误之后还调用了next方法，将会返回遍历器对象的value值为undefined，done为true的对象，并且认为Generator函数已经运行结束了。

```js
function* gen() {
  console.log('starting');
  yield 1;
  throw new Error('抛出错误');
  yield 2;
  yield 3;
}

let g = gen();
try {
  console.log('第一次运能next方法：', g.next())
} catch(e) {
  console.log('第一次运行---捕获错误', e);
}

try {
  console.log('第二次运行next方法：', g.next());
} catch(e) {
  console.log('第二次运行---捕获错误', e);
}

try {
  console.log('第三次运行next方法：', g.next());
} catch(e) {
  console.log('第三次运行---捕获错误', e);
}

//运行结果：
//starting
//第一次运能next方法： {value: 1, done: false}
//第二次运行---捕获错误 Error: 抛出错误
//第三次运行next方法： {value: undefined, done: true}
```

## Generator.prototype.return

Generator函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历Generator函数。

在调用return方法后，返回的遍历器对象的value为return方法的参数，done被改为true，Generator函数的遍历就被终止了。以后再调用next方法，返回对象的value值永远是undefined，done为true。

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

let g = gen();

console.log(g.next());            //{value: 1, done: false}
console.log(g.return('ending'));  //{value: "ending", done: true}
console.log(g.next());            //{value: undefined, done: true}
```

如果调用return方法的时候，没有传参数，那返回的value值就是undefined。

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

let g = gen();

console.log(g.next());            //{value: 1, done: false}
console.log(g.return());          //{value: undefined, done: true}
console.log(g.next());            //{value: undefined, done: true}
```

如果Generator函数内有try...finally代码块，并且正在执行try代码块，在这段期间调用return方法会结束当前try代码块内的代码执行，但是return结束遍历的语句需要等到finally代码块里面的代码执行完毕之后才会执行。

```js
function* gen() {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}

let g = gen();

console.log(g.next());            //{value: 1, done: false}
console.log(g.next());            //{value: 2, done: false}
//有finally代码块，执行finally代码块里面的内容
console.log(g.return(7));         //{value: 4, done: false}
console.log(g.next());            //{value: 5, done: false}
//执行完finally代码块，执行return语句
console.log(g.next());            //{value: 7, done: true}
//到这位置，已经结束了Generator函数的遍历，在调用next方法，value值为undefined，done为true
console.log(g.next());            //{value: undefined, done: true}
```

## yield* 表达式

如果想要在Generator函数内部，调用另外一个Generator函数，不用yield* 表达式的情况下，需要在前者函数中手动循环第二个Generator函数。这样会操作起来麻烦，所以yield* 表达式就是来解决这个问题的，只需用yield* 表达式来调用需要调用的Generator函数即可。在yield表达式后面加上一个*，表明他返回的是一个遍历器对象，这被称为yield*表达式。

```js
//1.手动循环
function* gen1() {
  yield 1;
  yield 2;
  for(let value of gen2()) {
    yield value;
  }
}

function* gen2() {
  yield 'a';
  yield 'b';
}

for(let value of gen1()) {
  console.log(value);       //1  2  a  b
}

//2.使用yield*表达式
function* gen1() {
  yield 1;
  yield 2;
  yield* gen2();
}

function* gen2() {
  yield 'a';
  yield 'b';
}

for(let value of gen1()) {
  console.log(value);       //1  2  a  b
}
```

从上面的代码中可以看出，如果yield* 表达式后面的Generator函数中没有return语句时，等同于在Generator中部署了一个for...of循环。如果有return表达式的时候，需要用let val = yield* gen2()的形式获得return语句的值。

实际上，任何数据结构只要有Iterator接口，就可以被yield*遍历。

**数组**

如果用yield后面跟这一个数组，则会将整个数组返回，如果yield* 后面跟一个数组，会遍历数组的成员。

```js
//yield表达式
function* gen() {
  yield [1, 2, 3];
}

let g = gen();
console.log(g.next().value);      //[1, 2, 3]

//yield*表达式
function* gen() {
  yield* [1, 2, 3];
}

let g = gen();
console.log(g.next().value);          //1
console.log(g.next().value);          //2
console.log(g.next().value);          //3
```

和上面的数组类似的，字符串也是这样的，如果yield表达式后面是一个字符串，则会返回一个字符串。要是yield*表达式，则会遍历的值返回。

## 作为对象属性的Generator函数

如果一个对象的属性是Generator函数，可以写成下面的形式。

```js
//完整形式
let obj = {
    gen: function* () {}
}
//简写形式
let obj = {
    *gen() {}
}
```

## Generator函数的this

Generator函数总是返回一个遍历器，ES6规定这个遍历器是Generator函数的实例，同时也继承了Generator函数prototype对象上的方法。我在项目中一般没有这样用过，了解了就可以，如果以后用到了再举用例。

他和普通的构造函数有区别：
- 不能用new来声明；
- 在构造函数中使用this无效；

如果在Generator函数的prototype声明一个方法，能被返回的实例继承，但是如果用this声明的属性则不能。

```js
function* gen() {
  this.x = 'hello';
}

gen.prototype.fun = function() {
  return 'hello';
}

let obj = gen();

console.log(obj.fun());           //hello
console.log(obj.x);               //undefined  
```

那么，有没有办法让Generator函数返回一个正常的实例对象，既可以用next方法，又可以获得正常的this？下面让一步步去实现。

第一步：首先实现可以访问this声明的属性：

```js
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

let obj = {};
let g = gen.call(obj);

//next方法
g.next();
g.next();

//输出对象属性
console.log(obj);   //{a: 1, b: 2, c: 3}
```

首先，生成一个空对象，用call方法绑定Generator函数内部的this。然后调用Generator函数，返回一个Iterator对象，这个对象最少需要执行两次（因为有两个yield表达式）next方法。因为只有将gen函数内的所有代码执行完，这是，所有的内部属性才绑定在obj对象上了，此时obj对象也就成了gen的实例了。

第二步：将遍历器g和生成的实例对象obj对象统一：

将obj对象换成gen.prototype;

```js
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

let g = gen.call(gen.prototype);

//next方法
g.next();
g.next();

//输出对象属性
console.log(g.a);   //1
```

第三步：将gen改成构造函数，就可以对它执行new命令了：

```js
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

function fun() {
  return gen.call(gen.prototype);
}

let g = new fun();

//next方法
g.next();
g.next();

//输出对象属性
console.log(g.a);   //1
```

## 含义

**1.Generator函数与状态机**  


Generator是实现状态机的最佳结构，因为用ES5实现的话，需要用一个外部变量保存状态。用Generator函数就不需要额外的变量，这样会更简洁、更安全（状态不会被非法修改），更符合函数式编程，在写法上也更加的优雅。Generator函数之所以可以不用外部变量保存状态，是因为它本身就包含了一个状态信息，即目前是否处于暂停状态。

**2.Generator与上下文**

JavaScript代码运行时，会产生一个全局的上下文环境（又称运行环境），包含了当前所有的变量和对象。然后，执行函数的时候，又会在当前上下文环境的上层，产生一个函数运行的上下文。变成当前的上下文，由此会形成一个上下文环境的堆栈。

这个堆栈是“后进先出”的数据结构，最后产生的上下文环境首先执行完成，退出堆栈，然后再执行完成它下层的上下文，直至所有的代码执行完成，堆栈清空。

Generator函数不是这样的，他执行产生的上下文环境，一旦遇到yield表达式，就会暂时退出堆栈，但是并不消失，里面所有的变量和对象会冻结当前状态。等到对它执行next命令的时，这个上下文环境又会重新加入调用栈，冻结的对象和变量恢复执行。

## 应用

Generator函数可以暂停函数执行，返回任意表达式的值。这种特点使得Generator有多种应用场景。

**（1）异步操作同步化**

Generator函数的暂停执行的的效果，意味着可以把异步操作写在yield表达式里面，等到调用next方法时，再往后执行，这实际上是等同于不需要写回调函数了，因为异步操作的后续操作可以放在yield表达式的下面，反正要等到调用next方法时再执行。所以Generator函数的一个重要实际意义就是用来处理异步操作，不用回调函数的方式了。

最能体现异步的例子可能就是调用接口的时候，下面用一个简单的axios来实现异步：

```js
//首先需要引入axios的cdn:<script src="https://unpkg.com/axios/dist/axios.min.js"></script>

//需要调用的接口：https://api.github.com/users/lm1073330969
  function* gen() {
    let val = yield 'lm1073330969'
    yield axios.get(`https://api.github.com/users/${val}`)
  }

  let g = gen()
  let username = g.next().value
  //axios返回的是一个Promise对象
  g.next(username).value.then(res => {
    console.log(res.data)
  })
```

上面代码运行的结果：
![e2UELT.png](https://s2.ax1x.com/2019/08/05/e2UELT.png)


**（2）部署Iterator接口**

上面有介绍过，利用Generator函数，可以在任意对象上面部署Iterator接口，部署完成过后，对象就可以被for...of遍历了。具体的例子，上面有两种介绍，介绍的很详细，这里就不做介绍了。

**（3）将嵌套数组铺平**

Generator函数还有一个很实用的一个用法，就是将嵌套数组铺平。

```js
function* paveArray(arr) {
  if(Array.isArray(arr)) {
    for(let i = 0; i < arr.length; i++) {
      yield*paveArray( arr[i]);
    }
  } else {
    yield arr;
  }
}

let array = [1, [2, 3, [4], 5], 6];

console.log([...paveArray(array)]);        //[1, 2, 3, 4, 5, 6]
```
