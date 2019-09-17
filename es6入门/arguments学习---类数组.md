# arguments学习
## arguments含义
arguments对象是所有函数中都可用用的局部变量（除了箭头函数），在箭头函数中使用arguments对象会报错。arguments中存放了所有传递过来的参数，arguments是一个类数组。

> 通过上面对arguments含义的描述，带出了我对以下的问题的想法：

### Q1：为什么箭头函数中不能使用arguments对象？
#### 先来了解一下箭头函数

- 产生箭头函数的原因

箭头函数的产生原因就在于==this==的指向问题，在箭头函数出现之前，每个新定义的函数都有它自己的==this==。和传统的函数相比，箭头函数没有自己的this、arguments、super或new.target。  

箭头函数不会创建自己的==this==，所以他没有自己的==this==，他只会从自己的作用域链的上一层继承==this==。

箭头函数==this==指向的固定化，并不是因为函数内部做了处理，而是因为箭头函数中this的指向在他被定义的时候已经确定了，之后永远不会改变。

并且在执行箭头函数的时候，他自己会捕获自己在定义时（注意是定义时，不是调用时）所处的外层执行环境的this，并继承这个this。所以箭头函数的this的指向，是在定义时就已经确定的。

> 看下面的一个例子，箭头函数没有自己的this，但是在定义箭头函数的时候他会继承自己的作用域链的上一层，要是上一层也是箭头函数他会一直往上找，一直找到最外层的window，下面这段代码，fun定义在全局变量下，所有他的上一层直接就是window。

```
let fun = () => {
    console.log(this);
}

fun('111');     //Window
```

> 再看下面这两个例子：  

1. 不适用箭头函数的情况：

```
const obj = {
    a: function() {
        console.log(this);
    }
}

obj.a();    //打印出的obj对象

```

打印出来的结果，this是指向obj这个对象的

![EjnhcD.png](https://s2.ax1x.com/2019/05/19/EjnhcD.png)

2. 使用箭头函数的情况

```
const obj = {
    a: () => {
        console.log(this);
    }
}

obj.a();    //打印出的是：window对象
```

在使用箭头函数的例子中，因为箭头函数默认不会有自己的this，而是会去作用域链的上一层去继承，obj定义在window下，所以就会顺其自然的去继承window

- 使用箭头需要注意的点

1 函数内的this对象是在定义时所在的对象，而不是调用该函数的对象。

2 整因为自身没有this对象，因此不可以作为构造函数，不能用call(),apply(),bind()这些方法去改变this的指向。

3 不能在箭头函数中使用arguments对象，可以使用rest参数来解决。

4 不能使用new操作符，否则会报错

```
var fun = () => {};
var func = new fun();
// TypeError: Func is not a constructor
```

5 没有prototype属性

6 不能用作生成器。yield关键字通产不能在箭头函数中使用（除非是嵌套在允许使用的函数内）

### Q2：类数组是什么？
类数组有两个特性：  
1. 具有length属性
2. 可以使用下标访问每个元素

除了以上两个特性之外，没有任何Array属性。所以它叫类数组，但是可以将类数组转换成一个真正的数组。转换之后就可以用所有属于Array的方法了。

> 注意：人为造类数组的数据的时候，注意必须有length这个属性，并且length这个属性的值必须是大于0的正整数，如果没有length这个属性，算不上类数组。

> 类数组示例：

```
let obj1 = {
    0: '张三',
    1: '李四',
    2: '王五',
    length: 3
};
```

> 非类数组示例

缺少length属性，所以不是类数组
```
let a = {
    '0': '张三'
}
```

不是类数组，因为键值不是数字索引，字符串数字或number都可以

```
let a = {
    'A': 'a'
}
```

JavaScript中常见的类数组有arguments对象和DOM方法返回的结果。比如：document.getElementsByTagName();

### Q3：如何判断数组是真正的数组

- 使用instanceof判断

instanceof运算符可以用来判断某个构造函数的prototype属性所指向的对象是否存在于另外一个要检测对象的原型链上。

```
let obj1 = {
    0: '张三',
    1: '李四',
    2: '王五',
    length: 3
};

let obj2 = ['张三', '李四', '王五'];

console.log(obj1 instanceof Array);     //false
console.log(obj2 instanceof Array);     //true
```

- 用Array对象的isArray方法判断

这可能是目前为止最靠谱的判断数组的方法了，当参数为数组是时，返回true，不是数组时为false。

```
let obj1 = {
    0: '张三',
    1: '李四',
    2: '王五',
    length: 3
};

let obj2 = ['张三', '李四', '王五'];

console.log(Array.isArray(obj1));       //false
console.log(Array.isArray(obj2));       //true
```

### Q4：怎么将类数组转换成一个真正的数组？

将类数组转换为数组的方法有以下几种：

#### Array.prototype.slice.call()

首先用Array.prototype.slice表示数组的原型上的slice方法，注意这个slice方法返回的是一个Array类型的对象。

实际上不是只能用slice方法，只要是能返回原数组就可以，比如splice也是可以的，只不过用slice方法看起来更加一目了然。

```
//人为的构造类数组
let classArray = {
  '0': '张三',
  '1': '李四',
  '2': '王五',
  length: 3
}

//使用slice方法
console.log(Array.prototype.slice.call(classArray));    //["张三", "李四", "王五"]
//使用splice方法
//splice删除或添加的方法，如果只传入start值，表示从开始下标删除到数组的结尾，所有传入0表示删除整个数组，他也能返回一个新的数组。
console.log(Array.prototype.splice.call(classArray, 0));    //["张三", "李四", "王五"]
```

#### Array.from()

Arrat.from()是ES6中新增加的方法，可以将两类对象转换成真正的数组：类数组对象和可遍历对象（包括ES6中新增加的数据结构Set和Map）。

```
//人为的构造类数组
let classArray = {
  '0': '张三',
  '1': '李四',
  '2': '王五',
  length: 3
}

console.log(Array.from(classArray));    //["张三", "李四", "王五"]
```

#### 扩展运算符(...)

扩展运算符也是ES6新增加的内容，可以将某些数据结构转换为数组。

```
function classArray() {
  console.log([...arguments]);    //[2, 3, 4]
}

classArray(2, 3, 4);
```


