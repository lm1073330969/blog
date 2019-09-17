# ES6入门---for...of循环、和其他遍历语法的比较

## for...of循环

ES6引入了for...of循环，作为遍历所有数据结构的统一方法。一个数据结构只要部署了Symbol.iterator属性，就会被视为具有Iterator接口，可以使用for...of对数据进行循环，也就是说for...of循环内部调用的是数据结构的Symbol.iterator方法。

for...of循环可以使用的范围包括：

- 数组
- Set、Map结构
- 计算生成的数据结构
- 某些类数组的对象（比如：arguments、DOM NodeList对象）
- Generator对象
- 字符串

### 数组

数组原生具备iterator接口（默认部署了Symbol.iterator属性），for...of循环本质上就是调用这个接口产生的遍历器。

**数组直接可以被for...of循环例子：**

```js
let arr = ['a', 'b', 'c'];

for(let value of arr) {
  console.log(value);         //a
                              //b
                              //c
}
```

**for...of循环本质上调用的是Symbol.iterator属性的遍历器对象生成器例子：**

下面这个例子，对象默认没有部署Symbol.iterator属性，数组默认有，所以将对象部署上数组的Symbol.iterator属性，再用for...of循环对象，会发现和循环数组的结果是一样的。

```js
let arr = ['a', 'b', 'c'];

for(let value of arr) {
  console.log(value);         //a b c
}

let obj = {};
obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);

for(let objValue of obj) {
  console.log(objValue);        //a b c
}
```

### Set和Map结构

Set和Map结构也原生具有Iterator接口，可以直接用for...of循环遍历。

下面的两个例子展示了，如何遍历Set结构和Map结构，但是有两点需要注意：  
（1）遍历的顺序是按照各个成员被添加进数据结构的顺序；  
（2）Set结构遍历时，返回的是一个值，而Map结构遍历时，返回是一个数组，数组的长度为2，分别是当前Map成员的键名和键值  

**Set结构遍历例子：**

```js
let set = new Set().add('a').add('b').add('c')

for(let setValue of set) {
  console.log(setValue);          //a  b  c
}
```

**Map结构遍历例子：**

```js
let map = new Map();

map.set('name', 'lee');
map.set('age', 12);
map.set('skill', 'web');

for(let mapValue of map) {
  console.log(mapValue);          //["name", "lee"]
                                  //["age", 12]
                                  //["skill", "web"]
}
```

### 计算生成的数据结构

有些数据结构是在现有数据结构的基础上，计算生成的。比如ES6的数组、Set数据结构、Map数据结构都部署了以下三个方法，调用后都返回遍历器对象。

- entries()返回一个遍历器对象，用来遍历[键名,键值]组成的数组。对于数组，键名就是索引；对于Set结构来说，键名和键值相同；对于Map结构来说，默认调用的就是entries方法。
- keys()返回一个遍历器对象，用来遍历所有的键名；
- values()返回一个遍历器对象，用来遍历所有的键值；

这三个方法调用后生成的遍历器对象，所遍历的都是计算所生成的数据结构。

```js
let arr = ['a', 'b', 'c'];

for(let value of arr.entries()) {
  console.log(value);         //[0, "a"]
                              //[1, "b"]
                              //[2, "c"]
}
```

### 某些类数组的对象

类似数组的对象包括好几类，一般多数会被提到的有arguments对象、DOM NodeList对象。这两种对象默认也是部署了Iterator接口的，可以直接使用for...of循环遍历的。

**arguments对象循环遍历例子：**

```js
function likeArray() {
  for(let value of arguments) {
    console.log(value);         //a  b  c
  }
}

likeArray('a', 'b', 'c');
```

**DOM NodeList对象遍历例子：**

```js
window.onload = function() {
  let params = document.querySelectorAll('div');

  for(let value of params) {
    console.log(value);       //<div>lalala</div>     <div>hehehe</div>
  }
}
```

注意：并不是所有的类数组都有Iterator接口，上面提到的两类是一定有的，当我们遇到除了上述两类的类数组的时候，以防没有默认部署Iterator接口，最好是使用Array.from方法将其转换为数组。

**没有默认部署Iterator接口的类数组：**

下面这个例子我会展示两种能进行循环的方式，第一种就是上面提到的简便的方法，用Array.from方法将其转换为真正的数组。

```js
let likeObj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
}

//对于上面这样没有部署Iterator接口的对象，使用for...of循环会报错
//报错信息：likeObj is not iterable
for(let value of likeObj) {
  console.log(value);
}

//第一种方法：使用Array.from方法
for(let value of Array.from(likeObj)) {
  console.log(value);         //a b c
}
```

第二种就是，我们在前面学习Iterator相关的知识的时候，有了解到如果类数组符合是数值键名和存在length属性，类数组的Symbol.iterator可以直接引用数组的Symbol.iterator属性，这种方法是对类数组有要求的，所以要符合使用条件才能使用。

```js
//第二种方法：根据学习Iterator知识了解到的
let likeObj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
}

for(let value of likeObj) {
  console.log(value);           //a b c
}
```

### 字符串

字符串默认也部署了Iterator接口，所以字符串可以直接使用for...of循环，下面是例子：

```js
let str = 'hello';

for(let value of str) {
  console.log(value);         //h e l l o
}
```

## for...of循环与其他遍历语法的比较

### for...in循环和for...of循环之间的区别

（1）for...in循环能遍历所有的对象，但是for...of不能遍历普通的对象，会报错，for...of只能遍历部署了Iterator接口的对象。for...in循环默认的情况下输出的键名，for...of默认直接输出了循环的数据的键值。

for...in循环普通对象，正常输出，没有报错，但是注意输出的结果是对象的**键名**，注意就是for...in循环还有一个**缺点**，循环出来的键名是字符串，为什么说他是一个缺点呢，可以看下面的例子我是用数值来作为键名的，但是循环之后出来的键名却是字符串格式，这点在对象上面不是最严重的，最严重的是循环数组的时候：

```js
let obj = {
  0: 'a',
  1: 'b',
  2: 'c'
}

//for...in循环
for(let value in obj) {
  console.log(value);         //0  1  2
}
```

for...of循环普通对象会直接报错：

```js
let obj = {
  0: 'a',
  1: 'b',
  2: 'c'
}

//for...of循环
for(let value of obj) {
  console.log(value);           //报错信息：obj is not iterable
}
```

对两种循环输出的对比的例子，for...in输出的**键值**，for...of循环的是值：

```js
let arr = ['a', 'b', 'c'];

//for...in 循环输出的是键名
for(let key in arr) {
  console.log(key);         //0  1  2
}

//for...of 循环输出的是键值
for(let value of arr) {
  console.log(value);       //a  b  c
}
```

（2）for...in循环能遍历手动添加的其他的键，甚至包括原型链上面的键都可以输出来。

他不会管你用不用的到，也不会管他是否会出问题，会选择全部给你。这个也是他的一个问题，也是和for...of的一个区别。但是for...of循环就不会这样，他只会输出原有的值，比如下面这个例子：

```js
Object.prototype.objFun = function() {};

Array.prototype.arrFun = function() {};

let arr = ['a', 'b', 'c'];
arr.foo = 'hello';

//for...in循环
for(let key in arr) {
  console.log(key);
}
//输出的结果：
//0
//1
//2
//foo
//arrFun
//objFun

//for...of循环
for(let value of arr) {
  console.log(value);       //a  b  c
}
```

for...in这个问题，对于原型链上面的对象是可以进行剥离的，用hasOwnProperty可以进行剥离。但是对于手动添加的属性还是没有办法剥离。但是对于这样的数据在下面会介绍另一种循环forEach可以解决。下面会介绍。

```js
Object.prototype.objFun = function() {};

Array.prototype.arrFun = function() {};

let arr = ['a', 'b', 'c'];
arr.foo = 'hello';

for(let key in arr) {
  if(arr.hasOwnProperty(key)) {
    console.log(key);
  }
}
//输出的结果：
//0
//1
//2
//foo
```

（3）for...in在循环字符串、arguments对象的时候输出的是键值，for...of循环的结果是字符串的值。

```js
//字符串
let str = 'hello';

//for...in循环
for(let key in str) {
  console.log(key);           //0  1  2  3  4
}

//for...of循环
for(let value of str) {
  console.log(value);         //h  e  l  l  o
}

//arguments对象
//for...in循环
function likeArray() {
  for(let value in arguments) {
    console.log(value);         //0  1  2
  }
}

likeArray('a', 'b', 'c');

//for...of循环
function likeArray() {
  for(let value of arguments) {
    console.log(value);         //a  b  c
  }
}

likeArray('a', 'b', 'c');
```

综合上面的介绍和例子，两个的区别一个是对象上面的，总结起来for...in更加适合对象的循环，其余的只要之部署了Iterator接口的，比如像数组、字符串、arguments对象、NodeList对象、类数组等，建议还是用for...of循环，这也是另一个区别就是for...of会直接的到值，会使操作更加便利和符合需求。

### for...of循环和forEach循环之间的区别

- 在上面也有提到forEach循环，他遍历只会输出本属于自己的键，不会输出手动添加和原型链上的。

```js
Object.prototype.objFun = function() {};

Array.prototype.arrFun = function() {};

let arr = ['a', 'b', 'c'];
arr.foo = 'hello';

arr.forEach(value => {
  console.log(value);         //a  b  c
})
```

- forEach循环不能中途跳出，return语句不能起作用，for...of循环可以使用break中途跳出循环.

```js
let arr = [3, 5, 7];

//forEach循环不能中途结束循环
arr.forEach(value => {
  console.log(value);         //3 5 7
  if(value === 5) {
    return false;             //return语句没有起到任何作用
  }
})

//for...of循环可以使用break语句跳出循环
for(let value of arr) {
  console.log(value);         //3  5
  if(value === 5) {
    break;                    //break语句执行了，跳出了循环
  }
}
```








