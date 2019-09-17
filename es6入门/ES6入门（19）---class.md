# ES6入门---class

## class的基本语法

### 类的由来

在JavaScript语言中，生成实例对象的传统方法是通过构造函数来实现的。在ES5之前用的是以下这种方式来封装类的。

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.printName = function() {
  return this.name;
}

let person = new Person('lee', 18);
console.log(person.printName());          //lee
```

但是，上面的这种写法和其他语言的类的形式有很大的差异，以前学习java，类最基本的也是用class关键字来声明的。

所以ES6提供了更加接近传统语言的写法，也就是运用了class关键字。但是实际上ES6的class是一个语法糖，他的绝大部分功能，ES5都可以做到，但是class写法能让对象原型的写法更加清晰、更像面向对象编程的语法而已。所以上面用ES5写的代码，用ES6改写如下：

```js
class Person{
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  printName() {
    return this.name;
  }
}

let person = new Person('lee', 18);
console.log(person.printName());          //lee
```

上面定义了一个Person类，可以看到类里面有一个constructor方法，这就是构造方法，而this关键字则代表实例对象。

Person类除了构造方法外，还定义了一个printName方法。

注意：

- 定义类的方法的时候，前面不需要加function关键字，直接写函数定义即可
- 类里面的都是函数，但是各个函数之间不需要==逗号==分隔，加了逗号会报错。

### 类的使用

**1. 类的使用和构造函数的使用方式一样，直接用new命令**

```js
class Person{
  //.....
}

let person = new Person();
```

**2. 类的数据类型就是函数，类本身就指向构造函数**

```js
class Person{
  //.....
}

console.log(typeof Person);           //function
console.log(Person === Person.prototype.constructor)      //true
```

**3. 构造函数的prototype属性，在ES6的类上面继续存在**

实际上，类的所有方法都定义在类的prototype属性上面。

```js
class Person{
  printName() {}

  printAge() {}

  printKill() {}
}

//等同于
Person.prototype = {
  printName() {},
  printName() {},
  printKill() {}
}
```

并且在类的实例上面调用方法，其实就是调用原型上面的方法。

```js
class Person{
  printName() {}
}

let person = new Person();
console.log(person.printName === Person.prototype.printName)      //true
```

上面的代码中，person是Person的实例，person上面的printName方法就是Person类原型上的printName方法。

**4. Object.assign方法**

因为类的方法都定义在prototype对象上面，可以用Object.assign方法很方便地一次向类添加多个方法。

```js
class Person{
  printName() {}
}

Object.assign(Person.prototype, {
  printAge() {},
  printKill() {}
})
```

**5. prototype对象的constructor属性，直接指向类的本身，这与ES5的行为是一致的。**

```js
class Person{
  printName() {}
}

console.log(Person === Person.prototype.constructor)      //true
```

**6. 用ES6写的类的内部所有定义的方法，都是不可枚举的，但是ES5用函数的方式封装的类的原型上的方法是不一样的，ES5方式的是可以枚举的。

```js
//ES6的方式
class Person{
  printName() {}
  printAge() {}
}

console.log(Object.keys(Person.prototype));         //[]

//ES5的方式
function Person() {}

Person.prototype.printName = function() {}
Person.prototype.printAge = function() {}

console.log(Object.keys(Person.prototype));           //["printName", "printAge"]
```

### constructor方法

constructor方法是类的默认方法，通过new命令声称对象实例时，自动调用该方法，一个类必须有constructor方法，如果没有显示定义，一个空的constructor方法会被默认添加。

constructor方法默认返回实例对象（即this），同时也可以完全指定返回另一个对象。

```js
class Person{
  constructor() {
    return Object.create(null);
  }
}

let person = new Person();
console.log(person instanceof Person);        //false
```

上面的代码中constructor函数返回一个全新的对象，结果导致实例对象person不是Person类的实例了。

类必须使用new调用，否则会报错。这是他和普通函数的一个最主要的区别。

```js
class Person{
  constructor() {
    return Object.create(null);
  }
}

let person = Person();              //Class constructor Person cannot be invoked without 'new'
```

### 类的实例

与ES5一样，实例的属性除非显示定义在其本身（即定义在this对象上面），否则都是定义在原型上。

```js
class Person{
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  printName() {
    return this.name;
  }
}

let person = new Person('lee', 18);

console.log(person.hasOwnProperty('name'));               //true
console.log(person.hasOwnProperty('age'));                //true
console.log(person.hasOwnProperty('printName'));          //false
console.log(person.__proto__.hasOwnProperty('printName'));//true
```

上面的代码中，name和age都是实例对象person自身的属性（因为定义在this变量上），所以hasOwnProperty方法返回true，而printName方法是原型对象上面的属性（因为定义在Person类上），所以hasOwnProperty方法返回false。

与ES5一样，类的所有实例共享一个原型对象。

```js
class Person{
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

let person1 = new Person('lee', 18);
let person2 = new Person('tony', 3);

console.log(person1.__proto__ === person2.__proto__);         //true
```

上面代码中person1和person2都是Person的实例，他们的原型都是Person.prototype，所以__proto__属性是相同的。

这也意味着可以通过实例__proto__属性为类添加方法。

```js
class Person{
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

let person1 = new Person('lee', 18);
let person2 = new Person('tony', 3);

person1.__proto__.printName = function() {
  return '默认';
}

console.log(person1.printName());           //默认
console.log(person2.printName());           //默认

let person3 = new Person('lisa', 18);
console.log(person3.printName());           //默认
```

上面代码中在person1的原型上添加了一个printName方法，由于person1的原型就是person2的原型，因此person2也有此方法。并且在后面新建的person3上面也可以调用这个方法。这意味着，使用实例的__proto__属性改写原型，必须相当谨慎，不推荐使用，因为这会改变类的原始定义，影响到所有实例。

### 取值函数（getter）和存值函数（setter）

与ES5一样在类的内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为，这个存取行为一般很少在项目中用到，如果不是进行底层封装。

```js
class Person{
  set showName(val) {
    this.name = val;
    console.log(`设置的名字为: ${val}`);        //设置的名字为: lee
  }

  get showName() {
    return this.name;
  }
}

let person = new Person();
//存储name
person.showName = 'lee';
//读取
console.log(person.showName);                   //lee
```

### 属性表达式

类的属性名，可以采用表达式。

```js
let methodName = 'printName';
class Person{
  constructor(name) {
    this.name = name;
  }

  [methodName]() {
    return this.name;
  }
}

let person = new Person('lee');
console.log(person[methodName]());             //lee
```

### class表达式

与函数一样，类也可以用使用表达式的形式定义。一般不建议这样写。

```js
const Person = class {
  //....
}
```

采用class表达式，可以写出立即执行的class。

```js
const Person = new class {
  constructor(name) {
    this.name = name
  }
  printName() {
    return this.name;
  }
}('lee');

console.log(Person.printName());        //lee
```

### 注意点

**1. 严格模式**

类和模块的内部，默认就是严格模式，所以不需要使用use strict指定运行模式。只要你的代码写在类或模块之中，就只有严格模式可用。考虑到未来所有的代码，其实都是运行在模块之中，所以ES6实际上把整个语言升级到了严格模式。

**2. 不存在提升**

ES6的class类不存在变量提升，这一点需要注意，因为这和在ES5的时候用函数封装的类有所不同，因为函数的原因，所以存在函数提升。

```js
let person = new Person();            //报错信息：Cannot access 'Person' before initialization
class Person{}
```

上面代码中，Person类使用在定义的的前面，这样会报错，因为ES6不会把类的声明提升到代码头部。这种规定的原因与继承有关，必须保证子类在父类之后定义。

**3. name属性**

由于本质上，ES6的类只是ES5的构造函数的一层包装，所以函数的许多特性都被class继承，包括name属性。

```js
class Person{}

console.log(Person.name);             //Person
```

name属性总返回紧跟在class关键字后面的类名。

**4. Generator方法**

如果某个方法之前加上*，就表示该方法是一个Generator函数。

```js
class Person{
  constructor(...names) {
    this.names = names;
  }
  * gen() {
    for(let name of this.names) {
      yield name;
    }
  }
}

let getNames = new Person('lee', 'lisa', 'tony');
for(let name of getNames.gen()) {
  console.log(name);          //lee   lisa  tony
}
```

上面代码中gen函数前面有一个*,表示gen函数是一个Generator函数，会返回一个遍历器对象，能使用for...of循环。

同样也可以直接使这个类称为一个Generator函数。

```js
class Person{
  constructor(...names) {
    this.names = names;
  }
  * [Symbol.iterator]() {
    for(let name of this.names) {
      yield name;
    }
  }
}

for(let name of new Person('lee', 'lisa', 'tony')) {
  console.log(name);          //lee   lisa  tony
}
```

上面代码中，不像使用gen函数那样在外面调用gen函数，Symbol.iterator方法可以直接使Person类称为一个Generator函数，直接调用，就能用for...of循环。

**5. this的指向**

类的方法内部如果含有this，他默认指向类的实例，通常如果规矩使用this，则不会出现问题，但是要注意，一旦单独使用，就有可能报错。

比如说，将返回的实例进行解构，就会报错：

```js
class Person{
  constructor(name) {
    this.name = name;
  }
  
  printName() {
    return this.name;
  }
}

let person = new Person('lee');
let { printName } = person;
console.log(printName());           //Cannot read property 'name' of undefined
```

上面代码中printName方法，默认是指向person实例的，但是，如果将这个方法提取出来单独使用，this会指向该方法运行时所在的环境（由于class内部是严格模式，所以this实际指向的是undefined），从而导致导致找不到name属性而报错。

**解决办法：**

第一种：在构造方法中绑定this

```js
class Person{
  constructor(name) {
    this.name = name;
    this.printName = this.printName.bind(this);
  }
  
  printName() {
    return this.name;
  }
}

let person = new Person('lee');
let { printName } = person;
console.log(printName());           //lee
```

第二种：使用箭头函数，但是这种没有像第一种直接得到想要的结果。

```js
class Person{
  constructor(name) {
    this.name = name;
    this.printName = () => this;
  }
  
  printName() {
    return this.name;
  }
}

let person = new Person('lee');
let { printName } = person;
console.log(printName());           //Person {name: "lee", printName: ƒ}
```

箭头函数内部的this总是指向定义时所在的对象。上面的代码中，箭头函数位于构造函数内部，他的定义生效的时候，是在构造函数执行的时候。这时，箭头函数所在的运行环境，肯定是实例对象，所以this总是指向实例对象。但是，我们也可以看见，如果结构的是一个函数的话，不能直接返回结果，而是返回一个对象，所以一般我会直接使用第一种方法，因为简单，结果也直接就是想要的。

### 静态方法

类相当于实例的原型，所以在类中定义的方法，都会被实例继承。但如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

```js
class Person{
  static printName() {
    return 'hello';
  }
}

let person = new Person();
console.log(person.printName());            //person.printName is not a function
console.log(Person.printName());            //hello
```

上面的代码中，Person类中的printName方法前面有static关键字，表明该方法是一个静态方法，不会被实例继承，只能在类上面直接调用Person.printName()，在实例上面调用静态方法会报错，表示不存在该方法。

注意：如果静态方法包含this关键字，这个this指的是类，而不是实例。

```js
class Person{
  static print() {
    return this.printAge();
  }

  static printAge() {
    return 18;
  }
}
console.log(Person.print());            //18
```

上面代码中，静态方法print调用了this.printAge，这里的this指的是类，等同于调用Person.printAge，另外，还有一点静态方法和非静态方法可以重名。还有就是在调用静态方法的时候，constructor函数的this的属性不起作用，因为constructor函数只有类在new操作时才会执行。

并且，父类的静态方法可以被子类继承。静态方法也可以从super对象上调用。

```js
class Person{
  static print() {
    return 'hello';
  }
}

class Student extends Person{
  static print() {
    return super.print() + ' lee';
  }
}
console.log(Student.print());            //hello lee
```

### 实例属性的新写法

实例属性除了定义在constructor方法里面的this上面，也可以定义在类的最顶层。

```js
class Person{
  model = 'hello '

  constructor(name) {
    this.name = name;
  }

  printName() {
    return this.model + this.name;
  }
}

let person = new Person('lee');
console.log(person.printName());            //hello lee
```

上面代码中，实例属性model与函数constructor和printName，属于同一个层级，这时，不需要在实例属性前面加this。

这种新写法的好处是，所有实例自身的对象都定义在类的顶部，看上去比较整齐，一眼就能看出这个类有哪些实例属性。注意上面代码中Person类的model实例属性不用外面传进来的参数可以这样使用，但是如果需要Person传的变量，最好还是用constructor函数。

## class的继承

### 简介

class可以通过extends关键字实现继承，这也和java中的继承不谋而合了，这比ES5通过修改原型链实现继承，要清晰和方便的多。

```js
class Student extends Person{
  constructor(name, age) {
    super(name);
    this.age = age;
  }

  printName() {
    return super.printName() + ' ' + this.age;
  }
}

let student = new Student('lee', 18);
console.log(student.printName());            //lee 18
```

上面的代码中constructor函数和printName方法中都出现了super关键字，他在这里表示父类的构造函数，用来新建父类的this对象。

注意：子类必须在constructor方法中调用super方法，否则在新建实例时会报错。这是因为子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类相同的实例属性和方法，然后再对其加工，加上子类自己的实例属性和方法。如果不调用super方法，子类就得不到this对象。

```js
class Person{
  constructor(name) {
    this.name = name;
  }
  printName() {
    return this.name;
  }
}

class Student extends Person{
  constructor() {}
}

let student = new Student();      //Must call super constructor in derived class before accessing 'this' or returning from derived constructor
```

上面的代码中，用到了constructor函数，但是没有在constructor函数中调用super方法，导致子类得不到自己的this对象，会报错。

如果子类没有定义constructor方法，这个方法会被默认添加，代码如下，也就是说，不管有没有显示定义，任何一个子类都有constructor方法。

```js
class Student extends Person{

}

//等同于
class Student extends Person{
  constructor(...args) {
    super(...args);
  }
}
```

另外一个需要注意的就是，在子类的构造函数中，只有在调用了super方法之后，才能使用this关键字，否则会报错。这是因为子类的实例的构建是基于父类的实例，然而，只有super方法才能调用父类实例。

```js
class Student extends Person{
  constructor(name, age) {
    this.age = age;         //报错
    super(name);
    this.age = age;         //正确
  }
  printName() {
    return super.printName() + ' ' + this.age;
  }
}

let student = new Student();      
```

ES5的继承，实质实现创造子类的实例对象this，然后再将父类的方法添加到this上面。而ES6的继承机制完全不同，实质是先将父类实例属性和方法，加到this上面，然后再用子类的构造函数修改this。

最后，父类的静态方法也会被子类继承。

### Object.getPrototypeOf()

Object.getPrototypeOf方法可以用来从子类上获取父类。

```js
console.log(Object.getPrototypeOf(Student) === Person);           //true
```

### super关键字

super这个关键字，既可以当做函数使用，也可以当做对象使用。在这两种情况下，他的用法完全不同。

**第一种情况：super作为函数调用时，代表父类的构造函数。ES6要求，子类的构造函数必须执行一次super函数**

如果子类有constructor方法，必须在constructor方法中调用super()方法，代表调用父类的构造函数。这是必须的，否则JavaScript引擎会报错。

注意：super虽然代表了父类的构造函数，但是返回的是子类的实例，即super内部的this指的是子类的实例，因此super在这里相当于父类.prototype.constructor.call(this)。

```js
class Person{
  constructor() {
    console.log(new.target.name);
  }
}
class Student extends Person{
  constructor() {
    super();
  }
}

console.log(new Person());        //Person
console.log(new Student());       //Student
```

上面代码中，new.target指当前正在执行的类，可以看到，在super()执行时，他指向的是子类的构造函数。也就是说super内部的this指向的是Student。

作为函数时，super()只能用在子类的构造函数中，用在其他的地方会报错。

**第二种情况：super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类**

```js
class Person{
  constructor(name) {
    this.name = name;
  }
  printName() {
    return this.name;
  }
}
class Student extends Person{
  constructor(name) {
    super(name);
    console.log(super.printName());           //lee
  }
}

let student = new Student('lee')
```

上面代码中，子类中的super.printName()，就是将super当做对象使用，这时，super在普通方法中，指向Person.prototype，所以super.printName()相当于Person.prototype.printName()。

这里需要注意，由于super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的。

```js
class Person{
  constructor() {
    this.name = name;
  }
}
class Student extends Person{
  constructor(name) {
    super(name);
  }

  printName() {
    return super.name;
  }
}

let student = new Student();
console.log(student.printName());           //undefined
```

上面代码中name属性是父类的实例上的属性，所以不能通过super获得，因为super获取的是父类实例原型对象。所以说只有属性定义在父类的原型对象上，super才可以获取到，如下：。

```js
class Person{}
Person.prototype.name = 'lee';

class Student extends Person{
  printName() {
    return super.name;
  }
}

let student = new Student();
console.log(student.printName());           //lee
```

ES6规定，在子类普通方法中通过super调用父类的方法时，方法内部的this，指向的是当前子类的实例。

```js
class Person{
  name = 'hello';
  print() {
    console.log(this.name);
  }
}

class Student extends Person{
  constructor() {
    super();
    this.name = 'lee';
  }
  printName() {
    return super.print();
  }
}

let student = new Student();
student.printName();           //lee
```

上面代码中，super.print()虽然调用的是Person.prototype.print()，但是Person.prototype.print()内部的this指向子类的实例，导致输出的是lee，而不是hello。也就是说，实际上执行的是super.print.call(this)。

由于this指向的是子类实例，所以如果通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性。

```js
class Person{
  x = 1;
}

class Student extends Person{
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
  }
  print() {
    console.log(super.x);           //undefined
    console.log(this.x);            //3
  }
}

let student = new Student();
student.print();  
```

上面代码中super.x赋值为3，这时相当于this.x赋值为3。而当读取super.x的时候，读的是Person.prototype.x，所以返回undefined，打印this.x的时候，结果是super.x赋值的3。

如果super作为对象，用在静态方法中，那么super指向的是父类，而不是父类的原型对象。

```js
class Person{
  static print(msg) {
    console.log('static:', msg);
  }

  print(msg) {
    console.log('instance:', msg);
  }
}

class Student extends Person{
  static printMsg(msg) {
    super.print(msg);
  }
  printMsg(msg) {
    super.print(msg);
  }
}

Student.printMsg('1');            //static: 1

let student = new Student();
student.printMsg('2');            //instance: 2
```

上面代码中，super在静态方法中指向父类，在普通方法中指向父类的原型对象。

另外，在子类的静态方法中通过super调用父类的方法时，方法内部的this指向当前的子类，而不是子类实例。

```js
class Person{
  x = 1;

  static print() {
    console.log(this.x);
  }
}

class Student extends Person{
  constructor() {
    super();
    this.x = 2;
  }

  static printMsg() {
    super.print();
  }
}

Student.x = 3;
Student.printMsg();         //3
```

上面代码中，静态方法printMsg里面super.print()指向了父类的静态方法，这个方法里面的this指向的是Student，注意不是他的实例。

注意，在使用super的时候，必须显示指定是作为函数还是对象使用，否则会报错。


