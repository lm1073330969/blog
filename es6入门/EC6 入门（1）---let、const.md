# ECMAScript 6 入门---let、const

## 基本用法

let和const用法类似于var，用于声明变量

## let、const与var的区别

### 1. let和const声明的变量，只会在所声明的代码块内有效，而var声明的变量是全局的

> var全局变量，在代码块内声明的变量，在全局下也能正常打印出来

```
{
    var a = 'lala';
}

console.log(a);     //lala
```

> let和const局部变量，在let和const所声明的代码块内能被访问到，出了代码块就不能被访问到了。

```
{
    let a = 'lala';
    let b = 'hehe';

    console.log(a);     //lala
    console.log(b);     //hehe
}

console.log(a);     //a is not defined
console.log(b);     //b is not defined
```

### 2. let和const不存在变量提升，var存在变量提升

var声明的变量，即使变量的使用在声明之前也是不会报错的，因为var声明的变量存在变量提升，只要一开始运行程序，所有var声明的变量都会提前到程序的最前面，并且有undefined初始值。

但是let和const不会存在这样的问题，它所声明的变量只能在他声明之后使用，否则就会报错

> var存在变量提升，即使在声明a之前使用了变量a，程序也没有报错，会打印出undefined，因为var存在变量提升。

```
console.log(a);     //undefined
var a;
```

上面的代码实际的运行程序是如下这样的：

```
var a = undefinde;

console.log(a); 
var a;
```

> let和const不存在变量提升，变量只能在他声明之后使用。否则会报错。

```
{
    // console.log(a);     //报错：Cannot access 'a' before initialization
    let a = 'lala';
    console.log(a);     //lala

    // console.log(b);     //报错：Cannot access 'a' before initialization
    let b = 'hehe';
    console.log(b);     //hehe
}
```

### 3. let和const存在暂时性死区

只要是在块级作用域内存在let和const声明的变量，那么这个变量必须在声明之后使用，否则就会报错，把这种情况称之为暂时性死区。

> 暂时性死区示例，存在一个全局变量a，在块级作用域中使用a，但是同时块级作用域内有一个用let声明的变量a，所以此时a，整个块级作用域是被封锁的，所以就相当于，let没有提升变量的作用。

```
var a = 'lala';

{
    a = 'hehe';
    let a;
    console.log(a);     //报错：Cannot access 'a' before initialization    
}
```

ES6 明确规定了，如果在区块中存在let或const命令，这个区块对这些命令声明的变量，从一开始就形成封闭作用域。凡是在声明之前就使用这些变量的就会报错。在语法上被叫做“暂时性死区”。

暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现之后，才可以获取和使用该变量。


### 4. let和const不允许重复声明

let和const不允许在相同的作用域内，重复声明同一个变量。

## let实际上是为JavaScript新增了块级作用域

ES6引入了块级作用域之后，明确允许了在块级作用域中声明函数，块级作用域中函数语句的声明相当于let行为，在块级作用域之外不能使用。但是由于对之前的代码会有影响，所以为了兼容。浏览器的ES6环境中，块级作用域中声明的函数，行为类似于var声明的变量，但是只有对ES6的浏览器有效，在其余的环境中仍然是遵循let的这种声明。

```
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }
(function () {
  // var f = undefined; 这是会将代码块里面的函数按照var声明变量进行变量提升
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

考虑到环境的行为有差异，应该尽量避免在代码块内声明函数，如果非要声明，可以写成函数表达式，而不是函数声明的语句。

## let能解决最常见的使用var循环遇到的问题。
在使用var声明循环变量时，声明的变量在全局范围内都是有效的，所以在循环以外打印的结果，是循环之后保存的变量的结果。

但是let声明的变量，只会在当前循环的块级作用域内有效。

### for循环还有一个特别之处

就是设置循环的那一部分是一个父作用域，而循环体内部是一个单独的子作用域。

```
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

打印了三次“abc”，表明了循环定义变量和循环体不在同一个区域，同时let声明的同一个变量也印证了这一点。

## const声明的常数变量不能被修改的实质

const声明的变量，一旦声明了就必须立即初始化吧，不然会报错

实质上：  
const声明变量，保证的并不是变量的值不能改变，要是定义的基本类型的话，值是不能被改变，但是要是定义的是复杂类型时，只要变量指向的那个栈内存中保存的指向堆内存的地址不变就可以。既可以添加数据又可以修改数据。所以，要将一个复杂类型的变量声明为一个常量是要非常小心的。

```
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```


