# ES5+ES6入门---数值的扩展

## ES5中Number对象

### Number常用到的对象属性

- MAX_VALUE

表示可以获取到的最大的数

```
console.log(Number.MAX_VALUE);      //1.7976931348623157e+308
```

- MIN_VALUE

表示可以表示的最小的值

```
console.log(Number.MIN_VALUE);      //5e-324
```

### Number常用到的对象方法

- toFixed()

将数字转换为字符串，并且可以指定输出的小数点后的小数的位数

```
let num = 3.1415926;

console.log(typeof num.toFixed(2));        //string
console.log(num.toFixed(2));        //'3.14'
```

- toString()

将数字转换为字符串，可以指定要转换的基数，默认为10进制，如果要转换为二进制，在括号里面传2.

```
let num = 10;

console.log(typeof num.toString());     //string
console.log(num.toString(2));           //1010
```

- valueOf()

返回一个Number对象的基本数字值。

```
let num = 10;
console.log(num.valueOf());     //10
```

还可用于字符串返回数字，但是有一点要注意，如果字符串中是纯数字可以返回，但是返回的类型依然是string类型，要是字符串中不是纯数字，会一起返回。

```
let num = 'hello 3.14';

console.log(num.valueOf());             //hello 3.14
console.log(typeof num.valueOf())       //string
```

## ES6对数值的扩展

### 二进制和八进制的表示法

- 二进制的声明

二进制的英文单词Binary,二进制的开始是0，然后第二个位置是B/b，然后跟上二进制的值就可以了。

```
let binary = 0b1010;
console.log(binary);        //10
```

- 八进制的声明

八进制的英文单词Octal，八进制的开始是0，然后第二个位置是O/o，然后跟上八进制的值就可以了。

```
let octal = 0o777;
console.log(octal);         //512
```

> 注意：以下ES6中的Number.isFinite()和Number.isNaN()方法和传统的isFinite()和isNaN()方法是有区别的，在传统的方法中，都会先调用Number()将非数值的值转为数值，在进行判断，所有对那些能转换为数字的还是有效的。但是ES6是在Number对象上提供的这两个方法，不是全局的，所以对字符串不会进行转换。


### 检查一个数值是否为有限的---Number.isFinite()

1. 如果参数不是数值，直接返回false；

注意：即使是数字字符串也不可以，也会直接返回false。

传统的方法和新的方法的对比，证明上面的注意点，可以直观的进行解释说明。

```
//传入普通字符串
console.log(Number.isFinite('foo'));        //false
//ES6的方法，传入数值字符串、能转换成数字的参数true
console.log(Number.isFinite('123'));        //false
console.log(Number.isFinite(true));         //false
//传统的方法，传入数值字符串、能转换成数字的参数true
console.log(isFinite('123'));               //true
console.log(isFinite(true));                //true
```

2. 如果参数为NaN，则返回false；

```
//传入的为NaN
console.log(Number.isFinite(NaN));          //false
```

3. 如果参数为正无穷或负无穷，返回false；

```
//传入正无穷
console.log(Number.isFinite(Infinity));     //false
//传入负无穷
console.log(Number.isFinite(-Infinity));    //false
```

只有传入正确的数值，才会返回true

```
//传入数值
console.log(Number.isFinite(123));          //true
```

### 检查是否为NaN方法---Number.isNaN()

只要参数不是NaN，一律返回false。

1. Number对象上的这个新方法，要是传入字符串‘NaN’，也会返回false

注意和传统方法的区别：

```
//Number对象上的isNaN
console.log(Number.isNaN('NaN'));       //false
//传统方法的isNaN
console.log(isNaN('NaN'));              //true
```

2. 参数只有是NaN才会返回true

```
console.log(Number.isNaN(NaN));         //true
```

### Number.parseInt()和Number.parseFloat()

ES6将全局方法parseInt和parseFloat移植到Number对象上，行为完全保持不变。

这样做的目的，是减少全局性方法。更好的进行维护。

```
console.log(Number.parseInt('12.33'));      //12
console.log(Number.parseFloat('0.22##'));   //0.22
```

### 判断一个数值是否为整数---Number.isInteger()

1. 如果传入的参数不是数值，直接返回false。数值字符串也会返回false。

```
console.log(Number.isInteger('123'));           //false
console.log(Number.isInteger('lala'));          //false
console.log(Number.isInteger(true));            //false
console.log(Number.isInteger(null));            //false
```

2. 在JavaScript内部，整数和浮点数采用的是同样的存储方法，所有3.0和3被视为同一个值。

```
console.log(Number.isInteger(3.0));             //true
console.log(Number.isInteger(3));               //true
```

### 安全整数范围---Number.isSafeInteger()

JavaScript能够准确表示的证书范围在-2^53~2^53之间（不包括两个端点），一旦超过这个范围，就无法精确表示这个值。

```
//获得安全值的区间
let num = Math.pow(2, 53) - 1;

console.log('max:', num);           //9007199254740991
console.log('min:', -num);          //-9007199254740991

//在之前需要用一个值和计算出来的num作比较，现在ES6提供一个直接判断一个数值是否在安全值范围内
console.log(Number.isSafeInteger(num));         //true
console.log(Number.isSafeInteger(num + 1));     //false
```

