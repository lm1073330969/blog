# ES6入门---Proxy

proxy通常我称他为代理或拦截。

## 概述

proxy用于修改某些操作的默认行为，等同于在语言层面上做出修改，所以属于一种“元编程”，即对编程语言进行编程。

可以理解成在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。proxy这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

ES6原生提供Proxy构造函数，用来生成proxy实例。

```js
let pro = new Proxy(target, handler);
```

Proxy构造函数可以接受两个对象，target表示所要拦截的目标对象，handler参数也是一个对象，用来定制拦截行为的对象。new Proxy()生成的是一个Proxy实例。详细的分析介绍看下面的例子的介绍：

下面这个例子是一个拦截读取属性行为的例子：

```js
let obj = {
    nama: 'lee'
};

let handler = {
    get(target, nama) {
        console.log('getting');
        return Reflect.get(target, nama);
    }
}

let newObj = new Proxy(obj, handler);
//getting
console.log(newObj.nama);       //lee
```

上面的例子中，proxy接受的两个参数分别是：第一个参数是所要代理的目标对象也就是上面例子中的obj对象，第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作。就像上面的代码中，配置对象中有一个get方法，用来拦截对目标对象属性的访问请求。get方法的两个参数分别是目标对象和所要访问的属性名。可以看到上面例子的输出访问属性的时候，打印出了get方法中的getting，证明了get方法起到了作用了。

**注意：** 要使proxy起作用，必须针对proxy实例进行操作，而不是针对目标对象进行操作，要分清proxy实例对象和目标对象的区别。

如果handler没有设置任何拦截，那就等同于直接通向原对象。下面的例子中没有做任何拦截操作，给proxy实例对象赋值之后，实际上操作的是原对象也就是目标对象，访问proxy实例对象等同于访问目标对象：

```js
let obj = {};
//没有进行任何拦截，传了一个空对象
let newObj = new Proxy(obj, {});
//对proxy实例对象的name属性赋值
newObj.name = 'lee';
//打印目标对象
console.log(obj.name);          //lee
```

## Proxy实例的拦截方法

#### get()

get方法用于拦截某个属性的读取操作，他可以接受三个参数，依次为目标对象、属性名和proxy实例本身，第三个参数是可选的。

**判断属性是否存在的例子**

下面的一个例子是一个对判断一个对象的里面是否有要读取的属性，要是有就输出，没有就抛出一个错误。

```js
let obj = {
    name: 'lee'
};

let handler = {
    get(target, name) {
        if(Reflect.has(target, name)) {
            return Reflect.get(target, name);
        } else {
            throw new Error('要获取的参数不存在')
        }
    }
};

let newObj = new Proxy(obj, handler);

console.log(newObj.name);       //lee
console.log(newObj.name1);      //Uncaught Error: 要获取的参数不存在
```

**get方法可以继承**

下面的例子中将拦截操作定义在了原型对象上面，如果读取obj对象继承的属性，这时拦截会生效。

下面的例子中，目标对象是个空对象，下面将拦截操作定义在了obj的原型对象上面了，因为拦截的get方法可以继承，obj上面没有所要查找的属性，就会在原型上查找，这时拦截会生效。

```js
let pro = new Proxy({}, {
    get(target, name) {
        console.log('getting ' + name);
        return Reflect.get(target, name)
    }
});

let obj = Object.create(pro);
//getting name
console.log(obj.name);           //undefined
```

**实现数组读取负数的索引** 

这个例子需要的注意的点在之前访问对象的时候用键名就是字符串，但是现在变成数组的访问形式了，它里面传的虽说是数字，但是到拦截的get方法里面打印一下就能知道他不是数字，它实际上是个数组，所以计算要注意。

```js
let createArray = function(...elements) {
    let handler = {
        get(target, proKey, receiver) {
            //这一步是确定array里面传进来的是一个字符串
            console.log(typeof proKey)       //string
            //所以需要将字符串转换成数字
            let index = Number(proKey);
            //如果小于0就用参数的长度加上传进来的数
            if(index < 0) {
                //再将结果转为string
                proKey = String(elements.length + index);
            }
            return Reflect.get(target, proKey, receiver);
        }
    }
    return new Proxy(elements, handler);
}

let array = createArray('a', 'b', 'c');
console.log(array[2]);                  //c
console.log(array[-1]);                 //c
```

**利用get拦截，实现一个生产各种DOM节点的通用函数**

```js
let DOM = new Proxy({}, {
    get(target, proKey) {
        //返回一个对象
        return function(obj, ...text) {
            let el = document.createElement(proKey);
            //添加属性
            for(let key of Object.keys(obj)) {
                el.setAttribute(key, obj[key]);
            }
            //添加元素
            for(let value of text) {
                if(typeof value == 'string') {
                    value = document.createTextNode(value);
                }
                el.appendChild(value);
            }
            //返回元素
            return el;
        }
    }
})

console.log(DOM.div({id: 'lee', class: 'name'}, 'lalala'));           //<div id="lee" class="name">lalala</div>
```

**receiver参数**

第三个参数receiver，他总指向读操作get的target对象，一般情况下就是proxy实例。总是指向原始的读操作所在的那个对象。

下面的例子中name属性是由pro实例对象提供的，所以receiver指向pro实例对象

```js
let pro = new Proxy({}, {
    get(target, name, receiver) {
        return receiver;
    }
})

console.log(pro.name === pro);          //true
```

下面这个例子，将proxy实例创建在了obj对象的原型上，obj对象本身没有name属性，所以读取obj.name的时候会去obj的原型上找这个属性，所以在这时，receiver就指向了obj，代表原始的读操作所在的那个对象。

```js
let pro = new Proxy({}, {
    get(target, name, receiver) {
        return receiver;
    }
})

let obj = Object.create(pro);
console.log(obj.name === obj);          //true
```

注意：如果一个属性不可配置并且不可写，则proxy不能修改该属性，否则通过proxy对象访问该属性会报错。

#### set()

set方法用来拦截某个属性的赋值操作，他可以接受四个参数，依次为目标对象、属性名、属性值和proxy实例本身，其中最后一个参数可选。

**set()拦截的例子**

假定obj对象有一个age属性，该属性应该是一个不大于200的整数，对应的拦截的操作如下：

```js
let handler = {
    set(target, name, value) {
        if(name == 'age') {
            if(!Number.isInteger(value)) {
                throw new Error('超出了有效值的范围');
            }
            if(value > 200) {
                throw new Error('年龄不能大于200')
            }
        }
        return Reflect.set(target, name, value);
    }
}

let obj = new Proxy({}, handler);
obj.age = 201;                  //年龄不能大于200
```

**receiver参数**

set方法的第四个参数receiver，指的是原始的操作行为所在的那个对象，一般情况下是proxy实例本身。

下面的例子中设置obj.name属性的值的时候，obj并没有name属性，所以会到obj的原型上去找，obj的原型设置的是pro实例对象，所以设置name属性会触发set拦截方法，这时，第四个参数receiver就指向了原始赋值行为所在的对象obj了。

```js
let pro = new Proxy({}, {
    set(target, name, value, receiver) {
        return target[name] = receiver;
    }
});

let obj = Object.create(pro);

obj.name = 'lee';
console.log(obj.name === obj);          //true
```

注意：如果目标对象自身的某个属性，不可写并且不可配置，那么set方法将不起作用。

#### apply()

apply方法拦截函数的调用、call和apply操作

apply方法可以接受三个参数，分别是目标对象、目标对象的上线文、目标对象的参数数组。

**简单的一个例子**

下面的例子中，变量pro是proxy的实例，当pro作为函数调用时（pro()），就会被apply方法拦截。

```js
let fun = function() {
  console.log('this is fun')
}

let handler = {
  apply(target, ctx, args) {
    console.log('this is apply')
  }
}

let pro = new Proxy(fun, handler)
pro() //this is apply
```

**三种会引起拦截的操作**

下面的例子中，直接调用、call、apply三种任何一种调用方式，都会被apply方法拦截。

```js
let sum= function(x, y) {
    return x + y;
}

let handler = {
    apply(target, ctx, args) {
        //传参数的写法
        return Reflect.apply(target, ctx, args) * 2;
        //简便的写法
        return Reflect.apply(...arguments);
    }
}

let pro = new Proxy(sum, handler);

console.log(pro(1, 2));                 //6
console.log(pro.call(null, 2, 3));      //10
console.log(pro.apply(null, [2, 3]));   //10
```

#### has()

has方法用来拦截HasProperty操作，即判断对象是否具有某个属性，这个方法会生效。典型的操作就是in运算符。

has可以接受两个参数，分别是目标对象，需要查询的属性名。

**使用has拦截方法隐藏某些属性，不被in运算符发现**

下面的例子，如果obj对象的属性名的第一个字符是下划线，has方法就会进行拦截，从而不被in运算符发现。

```js
let obj = {
    name: 'lee',
    _age: 22
};

let handler = {
    has(target, name) {
        if(name[0] == '_') {
            return false;
        }
        return Reflect.has(target, name);
    }
}

let pro = new Proxy(obj, handler);
console.log('name' in pro);          //true
console.log('_age' in pro);          //false
```

注意：如果原对象不可配置或者禁止扩展，这时has拦截会报错。

**for...in循环也用到了in运算符，但是has拦截对for...in循环不生效**

```js
let stu1 = {
    name: '张三',
    score: 59
}

let stu2 = {
    name: '李四',
    score: 80
}

let handler = {
    has(target, prop) {
        if(prop == 'score' && target[prop] < 60) {
            console.log(`${target.name}成绩不及格`);
            return false
        }
        return Reflect.has(target, prop);
    }
}

let pro1 = new Proxy(stu1, handler);
let pro2 = new Proxy(stu2, handler);
// xxx in xxx操作生效
console.log('score' in pro1);       //张三成绩不及格、false
console.log('score' in pro2);       //true

//对for...in操作不生效
for(let key in pro1) {
    console.log(pro1[key]);         //张三
                                    //59
}
```

#### construct()

construct方法用于拦截new命令。

construct方法可以接受两个参数。
- target：目标对象
- args：构造函数的参数对象
- newTarget：创造实例对象时，new命令作用的构造函数，可选的

```js
let target = function() {}

let handler = {
  construct(target, args) {
    console.log('args = ' + args.join(', '))        //args = 1, 2
    return {value: args[0] * 10}
  }
}

let pro = new Proxy(target, handler)
console.log(new pro(1, 2))          //{value: 10}
```

construct方法返回的必须是一个对象，否则会报错。

```js
let target = function() {}

let handler = {
  construct(target, args) {
    console.log('args = ' + args.join(', '))        //args = 1, 2
    return args[0]
  }
}

let pro = new Proxy(target, handler)
console.log(new pro(1, 2))          //报错信息：'construct' on proxy: trap returned non-object ('1')
```

#### deleteProperty()

deleteProperty方法用于拦截delete操作，如果这个方法抛出错误或者返回false，当前属性就无法被delete命令删除。

**默认第一个字符为下换线的属性不能操作**

下面的例子中，如果删除第一个字符为下划线的属性会报错。

```js
let handler = {
    deleteProperty(target, key) {
        invariant(key);
        return Reflect.deleteProperty(target, key);
    }
}

function invariant(key) {
    if(key[0] == '_') {
        throw new Error('第一个字符为下划线的属性不能进行任何操作！')
    }
}

let obj = {
    name: 'lee',
    _age: 22
}

let pro = new Proxy(obj, handler);
console.log(delete pro.name);           //true
console.log(delete pro._age);           //Uncaught Error: 第一个字符为下划线的属性不能进行任何操作！
```

注意：目标对象自身的不可配置的属性不能被deleteProperty方法删除，否则报错。

#### defineProperty()

defineProperty方法拦截了Object.defineProperty操作。

下面的例子中defineProperty方法返回false，导致添加的新属性总是无效的。

```js
let handler = {
    defineProperty(target, key, description) {
        return false;
    }
}

let obj = {};

let pro = new Proxy(obj, handler);
pro.name = 'lee';
//name属性没有添加进去
console.log(pro);           //Proxy {}
```

注意：如果目标对象不可扩展，则defineProperty不能增加目标对象上不存在的属性 ，否则会报错。另外，如果目标对象的某个属性不可写或不可配置，则defineProperty方法不得改变这两个设置。

## Proxy.revocable()

Proxy.revocable方法返回一个可取消的proxy实例。

Proxy.revocable方法返回一个对象，该对象的proxy属性是Proxy实例，revoke属性是一个函数，可以撤销Proxy实例。

下面的例子中，当执行revoke函数之后，再访问Proxy实例，就会抛出一个错误。

```js
let obj = {
    name: 'lee',
    age: 22
};

let {proxy, revoke} = Proxy.revocable(obj, {});

console.log(proxy.name);            //lee

revoke();
console.log(proxy.name);            //Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
```

上面的例子在自己写的时候要注意Proxy.revocable方法返回的对象的里面的属性的变量名为proxy、revoke，这两个属性名不能写错。

**使用场景**

Proxy.revocable的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。

## this的指向问题

有些原生对象的内部属性，只有通过正确的this才能拿到，所以Proxy也无法代理这些原生对象的属性。

```js
let target = new Date();

let pro = new Proxy(target, {});
console.log(pro.getDate());         //Uncaught TypeError: this is not a Date object. at Proxy.getDate
```

上面的例子，getDate方法只能在Date对象实例上面才能拿到，如果this不是Date对象实例就会报错，这时，this绑定原始对象，就可以解决这个问题。

```js
let target = new Date();

let handler = {
    get(target, name) {
        if(name == 'getDate') {
            return target.getDate.bind(target)
        }
        return Reflect.get(target, name)
    }
}

let pro = new Proxy(target, handler);
console.log(pro.getDate());         //24
```