# ES6入门---Reflect

首先我先说明一下，在这里我选择了先学习Reflect，本来这节课按照阮大大的教程本该学习Proxy，但是我在看文档学习的过程中，发现了在Proxy中的学习涉及到了Reflect的内容，我之前对这两个方面了解的不是很深，又在网上搜索了一些文档，发现好多文档吧proxy和reflect放在一起学习的，我自己的学习步骤是先学习了reflect的内容，下面是我在学习reflect中总结的一些知识，不是全部都学到了，只是将常见的学习了一下，以后用到了再增加。

## 概述

reflect是一个内置的对象，可以提供拦截JavaScript操作的方法，reflect不是一个函数，所以他不是可构造的，也就是不能用new操作符来定义reflect。

通常我叫Reflect为反射，叫proxy为代理，这和java中的叫法很相似。

reflect对象是ES6新增加的API，reflect设计的目的有以下几点：

- 想要将object对象上的一些明显属于语言内部的方法，放到reflect对象上，现阶段，某些方法同时在object和reflect对象上部署，文档说了以后的新方法将只部署在reflect对象上，也就是说，reflect对象可以的方法是可以操作语言内部的方法。
- 修改了某些object方法返回的结果，让其变得更加合理

比如在Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而Reflect.defineProperty(obj, name, desc)则会返回false。

**先让我们了解一下Object.defineProperty的基本用法，如果了解过vue，会知道vue的特点之一就是双向数据绑定，双向数据绑定的原理就是Object.defineProperty，所以我之后会详细的学习一下Object.defineProperty，还有就是希望自己完整的敲一遍双向数据绑定，所以在这里只是先简单介绍一下**

**语法**

> Object.defineProperty(obj, prop, des)

**参数说明**

**obj**：要在这个对象上定义属性

**prop**：要定义或修改的属性的名称

**des**：将被定义或修改的属性的描述

**描述**

这个方法允许精确添加或修改对象的属性，通过赋值操作添加的普通属性是可以枚举的，能够被遍历出来，这些属性的值可以被改变，也可以被删除。但是使用Object.defineProperty方法添加的属性值是不可修改的。例子如下：

```js
let obj = {};

Object.defineProperty(obj, 'name', {
    value: 'lee'
});

console.log(obj);           //{name: "lee"}
//修改Object.defineProperty定义的属性值，结果显示没有修改成功，说明Object.defineProperty定义的属性值是不能修改的
obj.name = 'lisa';
console.log(obj);           //{name: "lee"}
```

**第三个参数（des）属性描述符**

属性描述里面的属性描述符里面有两种形式：

- 数据描述符：value、writable、configurable、enumerable
- 存取描述符：get、set

数据描述符是一个具有值的属性，该值可能是可写的，也可能是不可写的。

存取描述符是由getter-setter函数对描述的属性。

两种描述符的含义和具体的用法，修改的属性具体的操作，参考CSDN文档:https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty

注意：第三个参数对象里面的参数必须是两种形式之一，两者不能同时出现，如果同时出现就会抛出一个错误。

Object.defineProperty方法如果在遇到无法定义属性的时候会抛出一个错误和、Reflect.defineProperty方法不会抛出错误，而是会返回false，但是不会影响接下来的输出。

下面这个例子就是两种方法用法的例子：

```js
let obj = {};

//首先，先对name属性的值设置为不可以改写属性的值
Object.defineProperty(obj, 'name', {
    value: 'lee',
    writable: false
});                             
console.log(obj);               //{name: "lee"}

//上面声明了name属性的值是不可以修改的，但是这里修改了就会抛出一个错误，导致下面的代码无法执行，这样会让人觉得不是很合理
Object.defineProperty(obj, 'name', {
    value: 'lisa'
});                             //Uncaught TypeError: Cannot redefine property: name at Function.defineProperty
//打印这行代码没有执行
console.log(obj);                   

//所以在新的reflect中，修改了返回结果，不会抛出错误，而是返回false，下面的代码也能正常运行，想要修改的值没有修改成功，还是原数据
console.log(Reflect.defineProperty(obj, 'name', {
    value: 'lisa'
}));                                //false
//结果里面的值没有修改成功，但是没有影响下面的代码执行，这样给人的感觉是比较原来无缘无故的抛出个错误让人更加合理
console.log(obj);                   //{name: "lee"}
```

- 文档上说的是让Object操作都变成函数行为，其实我刚开始看的时候也没有看懂这句话的意思，看下面的举例你可能就会明白。

比如查找一个属性是否属于一个对象的时候，我们都会用 name in obj这样的判断，删除的时候会是 delete obj[name]这样，但是reflect将这样不是函数的操作都改成可函数的操作，在reflect里面，想要判断一个属性在不在一个对象里面用Reflect.has(obj, name)，删除的用法是Reflect.deleteProperty(obj, name)变成了这样的函数操作。

```js
let obj = {
    name: 'lee',
    age: 22,
    skill: 'web'
}

//老的写法
//查询某个属性是否属于这个对象的
console.log('name' in obj);             //true
//删除对象里面的某个属性
console.log(delete obj['age']);         //true
console.log(obj);                       //{name: "lee", skill: "web"}

//新的写法
//查询某个属性是否属于这个对象的
console.log(Reflect.has(obj, 'name'));      //true
//删除对象里面的某个属性
console.log(Reflect.deleteProperty(obj, 'age'));        //true
console.log(obj);                           //{name: "lee", skill: "web"}
```

- Reflect对象的方法与proxy对象的方法一一对应，只要是proxy对象的方法，就能在reflect对象上找到对应的方法。这就让proxy对象可以方便的调用对应的reflect方法，完成默认行为，作为修改行为的基础，也就是说，不管proxy怎么修改默认行为，你总可以在reflect上获取默认行为。

## 静态方法

reflect对象一共有13个静态方法。但是我不会全都介绍，我只会将自己常用或者以后可能会用到的介绍一下用法，也是为了面试的需要，其余的方法如果在项目中遇到，自己可以在官方文档上看他的具体用法。

- Reflect.apply(target, thisArg, args)
- Reflect.construct(target, ages)
- Reflect.get(target, name, receiver)
- Reflect.set(target, name, value, receiver)
- Reflect.deleteProperty(target, name)
- Reflect.defineProperty(target, name, desc)
- Reflect.has(target, name)
- Reflect.ownKeys(target)
- Reflect.getOwnPropertyDescriptor(target, name)
- Reflect.getPrototypeOf(target)
- Reflect.setPrototypeOf(target, prototype)
- Reflect.isExtensible(target)
- Reflect.preventExtensions(target)

**target：** 表示的是操作的对象

**name：** 表示的是操作的属性的名字

**receiver：** 指的是原始的操作行为所在的那个对象，一般这个参数用的比较少，自己对这个参数理解的也不是很透彻，如果用到了我会在具体的例子里面介绍，不用到就不会提。希望有对这个参数了解的人能帮忙解读一下。

上面这些方法的作用，大部分与object对象里面的方法同名的方法作用是相同的，而却他与proxy对象的方法是一一对应的。

下面的详细介绍也不会全都介绍，大概会在十个左右，也都是常用的或者以后将会用简单介绍。

#### Reflect.apply(target, thisArg, args)

Reflect.apply方法等同于Function.prototype.apply.call(fun, thisArg, args)

平时用apply时候时候就是改变this的指向的用法，一般用法都是fun.apply(obj, ages).

Reflect对象的apply的操作和之前的apply我的理解两者并没有什么区别，但是以后的话后者使用的会多。

```js
let array = [1, 2, 3, 4];
//老的写法
//js没有专门的api来获得数组的最大值或最小值，所以结合Math内置函数的max和min方法
//apply的用法，接收两个参数，第一个参数传的是将要指向的对象，第二个参数是要比较的参数，参数的个数就是类似数组的格式
console.log(Math.max.apply(null, array));           //4
console.log(Math.min.apply(null, array));           //1

//新的用reflect的写法
console.log(Reflect.apply(Math.max, null, array));      //4
console.log(Reflect.apply(Math.min, null, array));      //1
```

还有一种常用的方式，是和proxy对象的apply的方法的结合用，在下面学习proxy是详细的介绍和介绍相关的例子。

#### Reflect.get(target, name, receiver)

Reflect.get方法查找并返回target对象的name属性，如果没有该属性则会返回undefined。

**第一个例子**

下面这个例子简单的介绍一下Reflect.get方法的用法：

```js
let obj = {
    name: 'lee',
    age: 22,
    skill: 'web'
};

console.log(Reflect.get(obj, 'name'));          //lee
console.log(Reflect.get(obj, 'name1'));         //undefined
```

**第二个例子**

如果对象的属性部署了读取函数（getter），则读取函数绑定的是第三个参数receiver。下面这个例子将分别测试一下指定第三个参数和不指定第三个参数的结果，用来帮助我们了解一下第三个参数在这个方法里面的具体用处：

```js
let obj = {
    name: 'lee',
    age: 22,
    skill: 'web',
    get description() {
        return (`名字：${this.name}；年龄：${this.age}；技能：${this.skill}`);
    }
};

let newObj = {
    name: 'lisa',
    age: 20,
    skill: 'sing'
}

console.log(Reflect.get(obj, 'description'));                   //名字：lee；年龄：22；技能：web
console.log(Reflect.get(obj, 'description', newObj));           //名字：lisa；年龄：20；技能：sing
```

通过上面的这个例子我自己的理解第三个参数可能就是改变this的指向，像我没有指定第三个参数的输出，默认就是输出第一个参数也就是我所指定的对象，还有就是指定了第三个参数为一个newObj输出的结果就是我所指定的第三个参数对象里面的内容。这是我通过这个例子得出的结论。

**第三个例子**

如果Reflect.get的第一个参数不是对象会报错。

```js
console.log(Reflect.get(1, 'name'));            //报错信息：Reflect.get called on non-object at Object.get
```

#### Reflect.set(target, name, value, receiver)

**第一个例子**

Reflect.set方法设置target对象的name属性等于value

```js
let obj = {
    name: 'lee',
    age: 22,
    skill: 'web'
};

console.log(Reflect.set(obj, 'name', 'lisa'));      //true
console.log(Reflect.set(obj, 'age', 20));           //true
console.log(obj);                                   //{name: "lisa", age: 20, skill: "web"}
```

**第二个例子**

如果对象的属性设置了赋值函数（setter），则赋值函数的this绑定receiver，这和上面Reflect.get方法中的getter用法一致。

这个例子最后的结果改的是receiver的newObj对象。

```js
let obj = {
    name: 'lee',
    set description(value) {
        this.name = value
    }
};

let newObj = {
    name: 'lisa'
}

console.log(Reflect.set(obj, 'name', 'tom', newObj));      //true
console.log(obj)                //{name: "lee"}
console.log(newObj);            //{name: "tom"}  
```

**第三个例子**

下面这个例子涉及到了proxy（代理）对象，如果一遍看不明白，可以结合下面对proxy的介绍，看完了之后再回头来看这个例子，我也会尽我的理解一步步的解释下面这个例子，首先先让我们看例子：

注意：如果proxy对象和reflect对象联合使用，前者拦截赋值操作，后者完成赋值的默认行为，而且传入了receiver，那么Reflect.set会触发Proxy.defineProperty拦截。

```js
let obj = {
    name: 'lee'
};

let handler = {
    set(target, nama, value, receiver) {
        console.log('set');
        return Reflect.set(target, nama, value, receiver);
    },
    defineProperty(target, nama, description) {
        console.log(target, nama, description);         //{name: "lee"} "name" {value: "lisa"}
        console.log('defineProperty');
        return Reflect.defineProperty(target, nama, description);
    }
}

let proObj = new Proxy(obj, handler);
proObj.name = 'lisa';

//set
//defineProperty
console.log(proObj);            //Proxy {name: "lisa"}
```

上面的这个例子接下来我一步步分析：

**了解proxy的基本用法**

proxy一般我称为代理，proxy对象里面接收两个参数，并且返回一个新对象，第一个参数就是要代理的对象，第二个参数就是拦截要进行的操作，也是一个对象。里面具体有什么会在proxy对象学习的时候详细介绍。

这里例子要代理的就是obj对象的赋值操作，就是对对象的属性进行改写就会触发handler里面的set方法。

handler里面的set是对对象的拦截，Reflect对象里面的set方法是对对象完成赋值的默认操作。

**触发Proxy.defineProperty拦截**

Proxy.set拦截里面使用了Reflect.set，而且传入了receiver，所以触发了Proxy.defineProperty拦截。这是因为Proxy.set里面的receiver参数总是指向当前的proxy实例也就是上面例子中的proObj，所以将proObj当receiver参数传进Reflect.set的receiver。

所以最后导致的结果就是设置obj的name属性赋值到proObj。导致触发了Proxy.defineProperty拦截，我理解的是传receiver之后之所以会触发Proxy.defineProperty是因为正好操作的是proxy实例对象，也就是操作对象的属性，所以Object.defineProperty会触发，所以Proxy.defineProperty拦截。

**不触发Proxy.defineProperty拦截**
然而要是不传receiver，就不会触发Proxy.defineProperty的拦截，因为没有传Reflect.set默认指向的就是target，操作的还是拦截的对象，没有涉及到proxy的实例对象。下面的例子没有触发defineProperty的拦截：

```js
let obj = {
    name: 'lee'
};

let handler = {
    set(target, name, value, receiver) {
        console.log('set');
        //和上面的触发的相对比，只有没有传receiver参数
        return Reflect.set(target, name, value);
    },
    defineProperty(target, name, description) {
        console.log(target, name, description);         
        console.log('defineProperty');
        return Reflect.defineProperty(target, name, description);
    }
}

let proObj = new Proxy(obj, handler);
proObj.name = 'lisa';

//set
console.log(proObj);            //Proxy {name: "lisa"}
```

**如果没有对属性拦截，默认是会触发defineProperty拦截的**

```js
let obj = {};

let newObj = new Proxy(obj, {
    defineProperty(target, nama, description) {
        console.log('defineProperty');
        return Reflect.defineProperty(target, nama, description);
    }
})

newObj.nama = 'lee';
//defineProperty
console.log(newObj);        //Proxy {nama: "lee"}
```

**第四个例子**

如果Reflect.set的第一个参数不是对象会报错。

```js
console.log(Reflect.set(1, 'lisa'));        //报错信息：Reflect.set called on non-object at Object.set
```

#### Reflect.has(obj, name)

Reflect.has方法对应的就是name in obj的操作。返回一个布尔值，如果对象里有这个属性就会返回true，没有返回false。

```js
let obj = {
    name: 'lee',
    age: 20,
    skill: 'web'
}

console.log(Reflect.has(obj, 'name'));          //true
console.log(Reflect.has(obj, 'name1'));         //false
```

如果Reflect.has方法的第一个参数不是对象报错。

```js
console.log(Reflect.has(1, 'name'));            //报错信息：Reflect.has called on non-object at Object.has
```

#### Reflect.deleteProperty(obj, name)

Reflect.deleteProperty方法等同于delete obj[name]，用于删除，返回一个布尔值，如果删除成功，或者要删除的属性不存在都属于删除成功，删除失败，就是被删除的属性还存在，则会返回false。

```js
let obj = {
    name: 'lee',
    age: 20,
    skill: 'web'
}

console.log(Reflect.deleteProperty(obj, 'name'));       //true
console.log(obj);                                       //{age: 20, skill: "web"}
```

如果Reflect.deleteProperty方法的第一个参数不是对象会报错：

```js
console.log(Reflect.deleteProperty(1, 'name'));            //报错信息：Reflect.has called on non-object at Object.deleteProperty
```

#### Reflect.construct(target, args)

Reflect.construct方法等同于new target(...args)，这提供了一种不使用new，来调用构造函数的方法。

```js
//ES5的写法
function target(name) {
    this.name = name;           
    console.log(this.name);         //lee
}

let newFun = new target('lee');
console.log(newFun);                //target {name: "lee"}

//Reflect写法
console.log(Reflect.construct(target, ['lee']));            //target {name: "lee"}
```

#### Reflect.getPrototypeOf(obj)

Reflect.getPrototypeOf方法用于读取对象的__proto__属性，对应Object.getPrototypeOf(obj).

```js
function target() {}

let newFun = new target();

//旧的写法
console.log(Object.getPrototypeOf(newFun));     //constructor: ƒ target()
console.log(target.prototype);                  //constructor: ƒ target()
console.log(Object.getPrototypeOf(newFun) === target.prototype);        //true
//新的写法
console.log(Reflect.getPrototypeOf(newFun) === target.prototype);       //true
```

Reflect.getPrototypeOf方法和Object.getPrototypeOf()方法有一个区别，如果第一个参数不是对象，Object.getPrototypeOf方法会将这个参数转为对象，然后再运行，而Reflect.getPrototypeOf会报错。

```js
//会将参数转换成对象
console.log(Object.getPrototypeOf(1));          //Number {0, constructor: ƒ, toExponential: ƒ, toFixed: ƒ, toPrecision: ƒ, …}
//会直接报错
console.log(Reflect.getPrototypeOf(1));         //报错信息：Uncaught TypeError: Reflect.getPrototypeOf called on non-object at Object.getPrototypeOf
```

#### Reflect.setPrototypeOf(obj, newProto)

Reflect.setPrototypeOf方法用于设置目标对象的原型，对应Object.setPrototypeOf方法。Object.setPrototypeOf方法返回的是设置成功的原型，Reflect.setPrototypeOf方法返回一个布尔值，表示是否设置成功。

```js
let obj = {};
//旧的写法
console.log(Object.setPrototypeOf(obj, Array.prototype));           //Array {}
//新的写法
console.log(Reflect.setPrototypeOf(obj, Array.prototype));          //true
```

如果第一个参数不是对象，Object.setPrototypeOf会返回第一个参数本身，但是Reflect.setPrototypeOf第一个参数要不是对象的话方法会报错。

```js
//Object.setPrototypeOf第一个参数不是对象，会直接返回参数本身
console.log(Object.setPrototypeOf(1, {}));          //1
//Object.setPrototypeOf第一个参数不是对象，会报错
console.log(Reflect.setPrototypeOf(1, {}));         //TypeError: Reflect.setPrototypeOf called on non-object at Object.setPrototypeOf
```

如果第一个参数是undefined或null，Object.setPrototypeOf和Reflect.setPrototypeOf都会报错。

```js
console.log(Object.setPrototypeOf(undefined, {}));      //Object.setPrototypeOf called on null or undefined at Function.setPrototypeOf
console.log(Reflect.setPrototypeOf(null, {}));          // Reflect.setPrototypeOf called on non-object at Object.setPrototypeOf
```

#### Relect.defineProperty(target, name, desciptor)

Reflect.defineProperty方法基本等同于Object.definProperty, 用来为对象定义属性。但是我一般接触这个方法都是了解关于双向数据绑定的知识的，里面的get、set方法一般都会用到。要是单纯的为对象的属性定义的话，一般只会用到属性的值、是否可写、描述符是否能改变、是否可以枚举。

未来，Object.definProperty方法主键废除，请从现在开始使用Reflect.defineProperty代替他。

```js
let obj = {
    name: 'lee'
};

//旧的写法
Object.defineProperty(obj, 'name', {
    value: 'tom'
})
console.log(obj);           //{name: "tom"}

//新的写法
Reflect.defineProperty(obj, 'name', {
    value: 'lisa'
});
console.log(obj);           //{name: "lisa"}
```

Reflect.defineProperty方法和Object.definProperty方法如果对已存在的属性进行修改，修改成功Reflect.defineProperty会返回true，Object.definProperty会返回修改之后的对象，如果属性名在对象中不存在，两个方法都会执行将属性添加到对象中。

```js
let obj = {
    name: 'lee'
};

//旧的写法
console.log(Object.defineProperty(obj, 'name1', {
    value: 'tom'
}));                        //{name: "lee", name1: "tom"}
console.log(obj);           //{name: "lee", name1: "tom"}

//新的写法
console.log(Reflect.defineProperty(obj, 'name1', {
    value: 'lisa'
}));                        //true
console.log(obj);           //{name: "lee", name1: "lisa"}
```

如果Reflect.defineProperty和Object.defineProperty方法第一个参数不是对象都会报错

```js
Reflect.defineProperty(1, 'name', {});          //Uncaught TypeError: Reflect.defineProperty called on non-object at Object.defineProperty
Object.defineProperty(1, 'name', {});           //Uncaught TypeError: Object.defineProperty called on non-objectat Function.defineProperty
```

**这个方法可以与Proxy.defineProperty配合使用。**

比如下面的这个例子, Proxy.definProperty对属性赋值设置了拦截，然后使用Reflect.defineProperty完成了赋值：

```js
let obj = {};

let newObj = new Proxy(obj, {
    defineProperty(target, nama, description) {
        console.log('defineProperty');
        return Reflect.defineProperty(target, nama, description);
    }
})

newObj.nama = 'lee';
//defineProperty
console.log(newObj);        //Proxy {nama: "lee"}
```

#### Reflect.getOwnPropertyDescriptor(target, name)

Reflect.getOwnPropertyDescriptor和Object.getOwnPropertyDescriptor的作用基本一致，用于得到指定属性的描述对象，将来前者会替代掉后者。

```js
let obj = {
    name: 'lee'
}
//旧的写法
console.log(Object.getOwnPropertyDescriptor(obj, 'name'));          //{value: "lee", writable: true, enumerable: true, configurable: true}
//新的写法
console.log(Reflect.getOwnPropertyDescriptor(obj, 'name'));         //{value: "lee", writable: true, enumerable: true, configurable: true}
```

**区别**

Reflect.getOwnPropertyDescriptor和Object.getOwnPropertyDescriptor的区别在：如果第一个参数不是对象Object.getOwnPropertyDescriptor不报错，会返回一个undefined，但是Reflect.getOwnPropertyDescriptor会抛出错误，表示是非法参数。

```js
console.log(Object.getOwnPropertyDescriptor(1, 'name'));               //undefined

console.log(Reflect.getOwnPropertyDescriptor(1, 'name'));           //报错信息：index.js:1549 Uncaught TypeError: Reflect.getOwnPropertyDescriptor called on non-object at Object.getOwnPropertyDescriptor
```

#### Reflect.ownKeys(target)

Reflect.ownKeys方法用于返回对象的所有属性，基本等同于Object.getOwnPropertyNames与Object.getOwnPropertySymbols之和。

```js
let obj = {
    name: 'lee',
    age: 22,
    [Symbol.for('bar')]: 'bar',
    [Symbol.for('foo')]: 'foo'
}

console.log(Object.getOwnPropertyNames(obj));           //["name", "age"]
console.log(Object.getOwnPropertySymbols(obj));         // [Symbol(bar), Symbol(foo)]

console.log(Reflect.ownKeys(obj));                      //["name", "age", Symbol(bar), Symbol(foo)]
```







