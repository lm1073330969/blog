# ES6入门---Symbol

## 概述

ES5的对象属性名都是字符串，这样在团队中多个人写代码是容易造成属性名的冲突。

ES6引入了一种新的原始数据类型Symbol,表示独一无二的值，他也是JavaScript第七种数据类型。null、undefined、number、Boolean、string、object、symbol

symbol值通过symbol函数生成。对象的属性名现在有两种类型，一种是原来就有的字符串，另一种就是新增的symbol类型。凡是属性名属于symbol类型的，就都是独一无二到的，可以保证不会与其他属性名产生冲突。

```js
let str = Symbol();

console.log(typeof str);        //symbol
```

注意：symbol函数前不能使用new命令，否则会报错。这是因为生成的symbol是一个原始类型的值，不是对象。也就是说，由于symbol值不是对象，所以不能添加属性，基本上，他是一种类似于字符串的数据类型。

symbol函数可以接受一个字符串作为参数，表示对symbol实例的描述，作用主要是在控制台显示的时候比较容易区分。如果不加参数，在控制台输出的结果都是symbol()这样就分不清是哪个参数的symbol，参数就相当于他的描述信息，在输出的时候就能分清了。

```js
let str1 = Symbol('first');
let str2 = Symbol('second');

console.log(str1);          //输出的结果是红色的字：Symbol(first)
console.log(str1.toString());   //输出的结果是黑色的字：Symbol(first)

console.log(str2);          //输出的结果是红色的字：Symbol(second)
console.log(str2.toString());   //输出的结果是黑色的字：Symbol(second)
```

如果symbol的参数是一个对象，就会调用该对象的toString方法，将其转为字符串，然后生成一个symbol值。

```js
let obj = {
    toString() {
        return '123';
    }
}

let sym = Symbol(obj);
console.log(sym);           //Symbol(123)
```

注意：symbol函数的参数只是表示对当前symbol值的描述，因此相同参数的symbol函数的返回值是不相等的，用symbol声明变量，没有任何两个变量相等。

```js
//没有参数
let sym1 = Symbol();
let sym2 = Symbol();
console.log(sym1 === sym2);         //false

//有相同参数
let sym3 = Symbol('first');
let sym4 = Symbol('first');
console.log(sym3 === sym4);         //false
```

symbol值不能与其他类型的值进行计算，会报错。

## 属性名的遍历

symbol作为属性名，该属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringif()返回。但是。他也不是私有属性，有一个Object.getOwnPropertySymbols方法，可以获取指定对象的所有的symbol属性名。

## Symbol.for(),Symbol.keyFor()

有时我们希望重新使用同一个symbol值，symbol.for方法可以做到这一点。他同样接收一个字符串作为参数，然后会在全局搜索有没有以该参数作为名称的symbol值，如果有，就返回这个symbol值，否则会新建并返回一个以该字符串为名称的symbol值。

注意：symbol.for()与symbol()这两种写法，都会生成新的symbol，但是他们之间存在差别，前者会被登记在全局环境中共搜索，后者则不会。symbol.for()不会每次调用就返回一个新的symbol类型的值，而是会先检查给定的key是否已经存在，如果不存在才会新建一个值。

例如：调用30次Symbol.for('first'),每次返回的会是同一个symbol值，但是要是调用30次Symbol('first')，会返回30个不同的symbol值。

```js
console.log(Symbol.for('first') === Symbol.for('first'));       //true

console.log(Symbol('first') === Symbol('first'));               //false
```

上面这个例子之所以每次symbol返回的值都不一样，是因为symbol()写法没有登记机制，所以每次调用都会返回一个不同的值。

Symbol.keyFor方法返回一个已经登记的symbol类型值的key。

```
let sym1 = Symbol.for('first');
console.log(Symbol.keyFor(sym1));           //first

//Symbol()没有登记机制
let sym2 = Symbol();
console.log(Symbol.keyFor(sym2));           //undefined
```
## Symbol在项目中的实际应用

### Symbol在对象中的应用

下面的例子是如何用Symbol构建对象的key，并调用和赋值：

```js
let sym = Symbol('first');

let obj = {
    [sym]: '1'
};

//调用对象中的sym，不能用. 只能用[]取值
console.log(obj[sym]);          //1

//对symbol声明的变量赋值
obj[sym] = 2;
console.log(obj[sym]);          //2
```

### Symbol对象元素的保护作用

在对象中有很多值，但是循环输出时，并不希望全部输出，我们可以使用symbol进行保护。

看下面的例子：

```js
//对象中的属性没有保护的
let obj1 = {name: 'lee', age: 22, skill: 'web'};

for(let item in obj1) {
    console.log(obj1[item]);        //lee  22   web
}

//可以选择对某一个属性进行保护，比如对年龄属性进行保护
let age = Symbol('age');
let obj2 = {
    name: 'lee',
    [age]: 22,
    skill: 'web'
};

for(let item in obj2) {
    console.log(obj2[item]);        //结果只有lee  web，没有年龄说明保护成功
}
```

总结：实际上我在写前端项目的时候很少用到需要Symbol声明的值，但是由于面试的需要，还是应该对基本的知识进行了解，其实Symbol有关的知识还有很多，只是没有深入了解。因为用的少，所以如果问到的话，可以将基本的知识掌握进行回答。其余的知识可能在用到的时候再进行了解，毕竟自己用到这个知识点比较少。