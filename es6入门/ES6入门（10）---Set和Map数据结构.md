# ES6入门---Set和Map数据结构

## set

#### 基本用法

set是ES6新增加的一种数据结构，划重点是数据结构，set里面的成员的值都是唯一的，没有重复的值。和之前学的java的set含义是一样的，里面的值是唯一的。

生成set数据结构的例子，new Set()，里面放数组，结果是set类型的数据结构，注意参数是数组，但是输出的结果不是数组，但是一般我们都将set生成的数据转换成数组进行操作，看下面的例子：

```js
let setArr = new Set(['first', 'second', 'element']);

console.log(setArr);            //Set(3) {"first", "second", "element"} 
```

set最大的特点和最常用的就是他有去重的作用，返回的值是唯一的，相同的值已经被去掉了。看下面的例子，‘lee’出现两次，但是经过set函数的处理，最后的结果只有一个lee了：

```js
let setArr = new Set(['lee', '20', 'web', 'lee']);

console.log(setArr);            //Set(3) {"lee", "20", "web"}
```

#### Set实例的属性和方法

**Set结构的实例属性有以下两个**

- Set.prototype.constructor：构造函数，默认就是set函数。（平时用到的比较少）
- Set.prototype.size：返回的是set实例的成员的总数。（我理解的就是和数组的length含义相同）。

对size用法的实例：

```js
let setArr = new Set(['lee', '20', 'web', 'lee']);

console.log(setArr.size);            //3
```

**Set实例的操作方法（用于操作数据）**

- add(value)：添加某个值，返回set结构本身。
- delete(value)：删除某个数据，返回一个布尔值，表示删除是否成功
- has(value)：返回一个布尔值，表示该值是否为set的成员
- clear()：清除所有的成员，没有返回值。

**add()方法**

add方法可以向set结构加入成员，并且不会添加重复的值。下面的这个例子1被添加了2次，结果只有一个，说明不会添加重复的值：

```js
let setArr = new Set();
setArr.add(1).add(1).add(2);

console.log(setArr);                //Set(2) {1, 2}
console.log(setArr.size);           //2
```

**delete()方法**

delete方法删除set结构里面的值，返回的是布尔值，表示删除的是否成功。

```js
let setArr = new Set([1, 2, 3, 4, 5]);

console.log(setArr.delete(1));      //true
console.log(setArr);                //Set(4) {2, 3, 4, 5}
console.log(setArr.size);           //4
```

**has()方法**

has方法表示查找set结构里面是否有该值，返回的是布尔值。

```js
let setArr = new Set([1, 2, 3, 4, 5]);

console.log(setArr.has(1));         //true
console.log(setArr);                //Set(5) {1, 2, 3, 4, 5}
```

**clear()方法**

清除set结构里面的所有数据。没有返回值。

```js
let setArr = new Set([1, 2, 3, 4, 5]);

setArr.clear();       
console.log(setArr);                //Set(0) {}
```

可以用Array.from将set结构转为数组：

```js
let setArr = new Set([1, 2, 3, 4, 5]);
let array = Array.from(setArr);       
console.log(array);                //[1, 2, 3, 4, 5]
```

**Set实例的遍历操作（用于遍历成员）**

set结构的实例有四个遍历方法，可以用于遍历成员。

- keys()：返回键名的遍历器
- values()：返回键值的遍历器
- entries()：返回键值对的遍历器
- forEach()：使用回调函数遍历每个成员

需要特别指出的是set的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用set保存一个回调函数列表，调用时就能保证按照添加顺序调用。

**1. keys()、values()**

由于set结构没有键名，只有键值（我觉得是说键名和键值是同一个值比较合理也好理解），所以keys方法和values方法的行为完全一致。

```js
let setArr = new Set([1, 2, 3, 4, 5]);

//keys()键名的调用
for(let key of setArr.keys()) {
    console.log(key);               //1、2、3、4、5
}

//values()键值的调用
for(let value of setArr.values()) {
    console.log(value);             //1、2、3、4、5
}
```

**entries()**

entries方法是对set结构里面的值的键值对的遍历

```js
let setArr = new Set(['red', 'yellow', 'blue']);

for(let item of setArr.entries()) {
    console.log(item);                  //["red", "red"]、["yellow", "yellow"]、["blue", "blue"]
}
```

**forEach()**

set结构的实例和数组一样，也拥有forEach方法，用于对每个成员执行某种操作，没有返回值。

forEach方法的参数就是一个处理函数。该函数的参数与数组的forEach一致，依次为键值、键名、集合本身。这里需要注意的是set结构的键名和键值的值是一个，因此第一个参数与第二个参数的值永远都是一样的。

另外forEach方法还可以有第二个参数，表示绑定处理函数内部的this对象。

```js
let setArr = new Set(['red', 'yellow', 'blue']);

setArr.forEach((value, key, arr) => {
    console.log(`${value} : ${key}`);       //red : red
                                            //yellow : yellow
                                            //blue : blue
})
```

#### 遍历的应用

我这里只介绍了自己在项目中最常用的几种，以后用到会继续更新。

扩展运算符和set结构相结合，可以去除数组的成员。

```js
let setArr = new Set([1, 1, 1, 2, 2, 3, 4, 5]);
let array = [...setArr];
console.log(array);         //[1, 2, 3, 4, 5]
```

Array.from方法也可以将set结构的数据转换为数组。

## WeakSet

**含义**

WeakSet结构类似于Set结构，也是里面没有重复的值，但是两者的区别在于WeakSet的成员只能是对象，不能是其他类型的值。

**语法**

创建WeakSet结构的数据结构和创建set结构的一样，都是使用new，但是有一点需要注意，不能直接在WeakSet构造函数里面直接加数据，会报错，比如下面的例子：

```js
let weakSet = new WeakSet({name: 'lee'});
console.log(weakSet);       //object is not iterable (cannot read property Symbol(Symbol.iterator)) at new WeakSet (<anonymous>)
```

但是作为构造函数，WeakSet可以接受一个数组或类似数组的对象作为参数（实际上就是具有Iterable接口的对象）都可以，其余的类型都不可以，添加对象类型只能通过WeakSet实例的添加方法。

```js
//因为定义的obj没有Iterable接口，所以不能直接在WeakSet构造函数中添加
let obj = {
    name: 'lee',
    age: 22,
    skill: 'web'
}

let weakSet = new WeakSet(obj);
console.log(weakSet);       //报错信息：object is not iterable

//可以添加的
//之所以这个例子能添加是因为添加的是array数组里面的成员，里面的值还是个数组，也就是符合添加的是对象。如果是一维数组的话就不能用构造函数的方式添加，只能是用实例add方法添加。
let array = [[1, 2],[3, 4]];
let weakSet = new WeakSet(array);

console.log(weakSet);           //{[1, 2],[3, 4]}
```

不能直接像set那样直接添加数据是因为weak的意思是懒惰，理解就是和懒加载性质相似不会自己主动的添加数据，只有你把数据都写好，将变量名添加进去他才添加数据，说只能添加对象，我们都知道数组也是特殊的对象，也可以添加数组。

从上面的例子可以看出数组可以通过构造函数的参数添加，但是没有Iterable接口的对象就不可以添加，只能通过WeakSet实例的add添加方法添加数据。

```js
let obj = {
    name: 'lee',
    age: 22,
    skill: 'web'
}

let weakSet = new WeakSet();
weakSet.add(obj);
console.log(weakSet);       //WeakSet {{name: "lee", age: 22, skill: "web"}}

let array = [[1, 2],[3, 4]];
let weakSet = new WeakSet();

weakSet.add(array);
console.log(weakSet);           //WeakSet {[1, 2],[3, 4]}
```

#### WeakSet操作的方法

WeakSet结构有三个方法，分别是：

- add(value)：向WeakSet实例添加一个新成员
- delete(value)：删除一个指定的成员
- has(value)：返回一个布尔值，表示某个值是否存在实例中

```js
let array = [[1, 2],[3, 4]];
let weakSet = new WeakSet();

let add = [5];

//添加
weakSet.add(array);
weakSet.add(add);
console.log(weakSet);           //WeakSet {[[1, 2],[3, 4]], [[5]]}

//删除
console.log(weakSet.delete(array));         //true
console.log(weakSet);                       //{[[5]]}

//判断是否存在一个值
console.log(weakSet.has(array));            //false
```

注意：WeakSet没有size属性，也不能遍历他的成员，是因为成员都是弱引用，随时可能消失，遍历机制无法保证成员的存在，很可能刚刚遍历结束，成员就取不到了。

所以他有一个用处，就是储存DOM节点，而不用担心这些节点从文档移除时，会引发内存泄漏。

## Map

map也是ES6新增加的数据结构。他是类似于对象的，也是键值对的集合。

在平常使用中感觉map用到的会更加广泛，你会在一些源码中见到map的写法，相对来说map用的更加多。

他和对象的类型很相似，map出现的原因是解决对象的键名只能是字符串，map的键名可以是字符串、各种类型的值（包括对象）都可以当做键名。

所以，文档上总结了，object结构提供的是“字符串——值”的对应。map结构是“值——值”的对应，是一种更加完善的hash结构实现，如果需要“键值对”的数据结构map比object更加合适。

**map不支持直接向构造函数中添加数据**

如下面的例子，直接向map结构里面添加数据会报错：

```js
//不能直接将数据添加到map结构里面
let mapObj = new Map('name', 'lee');
console.log(mapObj);        //报错信息：Iterator value n is not an entry object at new Map
```

**通过内置的set方法向map结构中添加数据和map存储数据的格式**

set方法接受两个参数，第一个参数表示添加的键名，第二个参数是键值，例子如下：

```js
let key = 'name';
let mapObj = new Map();
mapObj.set(key, 'lee');
console.log(mapObj);            //Map(1) {"name" => "lee"}
```

注意：上面的这个例子同时说明了map存储数据的格式，键名和键值之间用=>来连接，左面为键名，右面的为键值

**作为构造函数，map也可以接受一个数组作为参数**

```js
let mapObj = new Map([
    ['nama', 'lee'],
    ['age', 22],
    ['skill', 'web']
]);

console.log(mapObj);            //{"nama" => "lee", "age" => 22, "skill" => "web"}
console.log(mapObj.size);       //3
```

**下面的例子是不同值作为键名的例子：**

目前我你能想到的就是这三种：以字符串、数字、对象为键名的例子，以后遇到再添加。

```js
//以字符串为键名
let key = 'name';
let mapObj = new Map();
mapObj.set(key, 'lee');
console.log(mapObj);            //Map(1) {"name" => "lee"}

//以对象为键名
let key = {
    name: 'lee',
    age: 20
};
let mapObj = new Map();
mapObj.set(key, 'obj');
console.log(mapObj);            //Map(1) {{name: "lee", age: 20} => "obj"}

//以数字为键值
let mapObj = new Map();
mapObj.set(1, 'first');
console.log(mapObj);             //Map(1) {1 => "first"}
```

**get()根据键名获取值的方法**

map数据结构获取键值的方法用get方法，传入一个键值。

```js
let key = {
    name: 'lee',
    age: 20
};
let mapObj = new Map();
mapObj.set(key, 'obj');
mapObj.set('obj', key);

console.log(mapObj.get(key));           //obj
console.log(mapObj.get('obj'));         //{name: "lee", age: 20}
```

**map的size属性**

map数据结构也有size属性，用户获取map结构里面数据的长度

```
let key = {
    name: 'lee',
    age: 20
};
let mapObj = new Map();
mapObj.set(key, 'obj');
mapObj.set('obj', key);

console.log(mapObj.size);               //2
```

**delete()方法，根据键名删除map结构里面的数据**

delete方法，会返回一个布尔值，表示是否删除成功。

```js
let key = {
    name: 'lee',
    age: 20
};
let mapObj = new Map();
mapObj.set(key, 'obj');
mapObj.set('obj', key);

console.log(mapObj.delete('obj'));          //true
console.log(mapObj);                        //Map(1) {{name: "lee", age: 20} => "obj"}
console.log(mapObj.size);                   //1
```

**has()方法，用于确认map结构里面是否有要查找的数据**

```js
let key = {
    name: 'lee',
    age: 20
};
let mapObj = new Map();
mapObj.set(key, 'obj');
mapObj.set('obj', key);

console.log(mapObj.has('obj'));             //true
console.log(mapObj.has('obj1'));            //false
```

**clear()方法，清除map里面的数据**

clear方法清除所有的数据，没有返回值

```js
let mapObj = new Map();
mapObj.set('name', 'tom');
mapObj.set('name', 'lisa');
mapObj.set('age', 20);

console.log(mapObj);            //Map(2) {"name" => "lisa", "age" => 20}
mapObj.clear();
console.log(mapObj);            //Map(0) {}
```

**对简单数据结构的键名多次赋值，后面的值会覆盖前面的值**

会覆盖的只是那些字符串或数字为键名的值，复杂数据类型情况会在下面介绍。

```js
let mapObj = new Map();
mapObj.set('name', 'tom');
mapObj.set('name', 'lisa');
mapObj.set('age', 20);

console.log(mapObj);            //Map(2) {"name" => "lisa", "age" => 20}
```

**键名为复杂类型的情况**

当键名为复杂类型的时候，注意，只有对同一个对象的引用，map结构才将其视为同一个键。这一点一定要注意，看一下下面几个例子之间的差别：

```js
//地址不同的情况
let mapObj = new Map();
mapObj.set([1], 1);

console.log(mapObj);                //Map(1) {Array(1) => 1}
//结果之所以是undefined，是因为键名是复杂类型，存的时候和取的时候的地址指向的不是内存中同一的地址
console.log(mapObj.get([1]));       //undefined

//声明两个对象做键名，两个对象的值是相同的，也就是键名的值相同，但是两个的存储地址是不同的
let obj1 = {nama: 'tom'};
let obj2 = {nama: 'tom'};
let mapObj = new Map();
mapObj.set(obj1, 'first');
mapObj.set(obj2, 'second');
               
console.log(mapObj.get(obj1));       //first
console.log(mapObj.get(obj2));       //second
```

#### 遍历方法

map结构原生提供了三个遍历器生成函数和一个遍历方法。

- keys()：返回键名的遍历器
- values()：返回键值的遍历器
- entries()：返回键值对的遍历器
- forEach()：遍历map的所有成员

需要注意的就是，map的遍历顺序就是插入顺序，下面是对三个遍历器生成函数和一个遍历方法的例子：

```js
let mapObj = new Map();
let obj = {
    name: 'lee',
    age: 20
};

mapObj.set('obj', obj);
mapObj.set('kill', 'web');
mapObj.set(1, 1);

//keys：键名遍历器
for(let key of mapObj.keys()) {
    console.log(key);           //obj、kill、1
}

//values: 键值遍历器
for(let value of mapObj.values()) {
    console.log(value);         //{name: "lee", age: 20}、web、1
}

//entries：键值对遍历器
for(let item of mapObj.entries()) {
    console.log(item);              //["obj", {name: "lee", age: 20}]
                                    //["kill", "web"]
                                    //[1, 1]
}

//forEach()：循环的方法
mapObj.forEach((value, key) => {
    console.log(`${key} : ${value}`)    //obj : [object Object]
                                        //kill : web
                                        //1 : 1
})

//map默认遍历的是entries遍历器
for(let item of mapObj) {
    console.log(item);              //["obj", {name: "lee", age: 20}]
                                    //["kill", "web"]
                                    //[1, 1]
}
```

#### map数据结构转为数组

map结构转为数组结构，比较方便的方法是使用扩展运算符（...）

```js
let mapObj = new Map();
let obj = {
    name: 'lee',
    age: 20
};

mapObj.set('obj', obj);
mapObj.set('kill', 'web');
mapObj.set(1, 1);

//将键名转为数组
console.log([...mapObj.keys()]);        //["obj", "kill", 1]
//将键值转为数组
console.log([...mapObj.values()]);      //[{name: "lee", age: 20}, "web", 1]
//将键值对转为数组
console.log([...mapObj.entries()]);     //[["obj", {name: "lee", age: 20}], ["kill", "web"], [1, 1]]
//map结构默认转为数组为entries方式
console.log([...mapObj]);               //[["obj", {name: "lee", age: 20}], ["kill", "web"], [1, 1]]
```

## WeakMap

WeakMap结构与map结构类似，也是用于生成键值对的集合。

但是他与map结构有两点的区别：

- WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名。
- WeakMap的键名所指向的对象，不计入垃圾回收机制。

WeakMap的设计目的在于，也就是我们常说的在用document获取内容的时候，又在别的地方引用了document的值，如果一旦不再需要这个值的时候需要手动删除这个引用，否则垃圾回收机制就不会回收这个值。

所以这个是WeakMap的作用，常用到的地方，我在项目中用到的还是比较少的，最常用的也就是set和map，要是没有特殊的需求，基本上用不到。


