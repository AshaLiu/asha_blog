---
title: js的数据类型转换
date: 2019-08-18 20:40:39
tags: 类型转换
---
### 前言
js是编译型语言，在运算的时候，会对数据类型做转换再进行计算。

### 基本概念
#### js类型
7 种原始类型：(值不会变)
`Boolean`
`Null`
`Undefined`
`Number`
`BigInt`
`String`
`Symbol `

引用类型：
`Object`

#### `valueOf()` 与 `toString()`
##### valueOf()
返回对象的原始值。默认情况下，valueOf方法由Object后面的每个对象继承。 每个内置的核心对象都会覆盖此方法以返回适当的值。
如果对象没有原始值，则valueOf将返回对象本身。但是！！！！JavaScript的许多内置对象都重写了该函数，以实现更适合自身的功能需要。
因此，不同类型对象的valueOf()方法的返回值和返回值类型均可能不同。

不同类型对象的valueOf()方法的返回值：

对象 | 返回
----|------
Array | 返回数组对象本身。
Boolean	| 布尔值。
Date	|	存储的时间是从 1970 年 1 月 1 日午夜开始计的毫秒数 UTC。
Function | 函数本身。
Number	| 数字值。
Object		| 对象本身。这是默认情况。
String		| 字符串值。

##### toString()
方法返回一个表示该对象的字符串。
默认情况下，toString() 方法被每个 Object 对象继承。如果此方法在自定义对象中未被覆盖，toString() 返回 `[object type]`，其中 type 是对象的类型。

小技巧：
```javascript
Object.prototype.toString.call([1,2]) // [object Array]
```

### 强制转换
#### 1. Number()
##### 规则：
1. 调用对象自身的valueOf方法。如果返回原始类型的值，则直接对该值使用Number函数，不再进行后续步骤。
2. 如果valueOf方法返回的还是对象，则改为调用对象自身的toString方法。如果toString方法返回原始类型的值，则对该值使用Number函数，不再进行后续步骤。
3. 如果toString方法返回的是对象，就报错，说自己不能转换

##### 针对原始类型的转换
```javascript
// 数值：转换后还是原来的值
Number(324) // 324

// 字符串：如果可以被解析为数值，则转换为相应的数值
Number('324') // 324

// Number函数将字符串转为数值，要比parseInt函数严格很多。
// 基本上，只要有一个字符无法转成数值，整个字符串就会被转为NaN。
// 但是！parseInt和Number函数都会自动过滤一个字符串前导和后缀的空格。

// 字符串：如果不可以被解析为数值，返回 NaN
Number('324abc') // NaN

// 空字符串转为0
Number('') // 0

// 布尔值：true 转成 1，false 转成 0
Number(true) // 1
Number(false) // 0

// undefined：转成 NaN
Number(undefined) // NaN

// null：转成0
Number(null) // 0
```
##### 对于Object的转换
Number方法的参数是对象时，将返回NaN，除非是包含单个数值的数组。
因为 `[5].toString()` 返回的是 `'5'`。刚好 `'5'`可以被Number()
`[1,3].toString()` 返回的是 `'1,3'`，`Number()` 认为不是数字，返回 `NaN`

#### 2. String()
规则：

String方法背后的转换规则，与Number方法基本相同，只是互换了valueOf方法和toString方法的执行顺序。
1. 先调用对象自身的toString方法。如果返回原始类型的值，则对该值使用String函数，不再进行以下步骤。
2. 如果toString方法返回的是对象，再调用原对象的valueOf方法。如果valueOf方法返回原始类型的值，则对该值使用String函数，不再进行以下步骤。
3. 如果valueOf方法返回的是对象，就报错自己转不了。

原始类型
```javascript
String(123) // "123"  数值：转为相应的字符串。
String('abc') // "abc"  字符串：转换后还是原来的值。
String(true) // "true"  布尔值：true转为字符串"true"，false转为字符串"false"。
String(undefined) // "undefined"   undefined：转为字符串"undefined"。
String(null) // "null"   null：转为字符串"null"。
```

对象类型
String方法的参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。
```javascript
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
````

#### 3. Boolean()
规则：
除了以下六个值的转换结果为false，其他的值全部为true。

```javascript
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean('') // false
Boolean(false) // false
```

注意：所有对象（包括空对象）的转换结果都是true，甚至连false对应的布尔对象new Boolean(false)也是true
```javascript
Boolean({}) // true
Boolean([]) // true
Boolean(new Boolean(false)) // true
```
所有对象的布尔值都是true，这是因为 JavaScript 语言设计的时候，出于性能的考虑，如果对象需要计算才能得到布尔值，对于obj1 && obj2这样的场景，
可能会需要较多的计算。为了保证性能，就统一规定，对象的布尔值为true。

### 自动转换（隐形转换）
自动转换的规则是这样的：预期什么类型的值，就调用该类型的转换函数。
比如，某个位置预期为字符串，就调用String函数进行转换。

遇到以下三种情况时，JavaScript 会自动转换数据类型，即转换是自动完成的，用户不可见。
1. 不同类型的数据互相运算。

```javascript
123 + 'abc' // "123abc" 
```

2. 对非布尔值类型的数据求布尔值。

```javascript
if ('abc') {
  console.log('hello')
}  // "hello"
```

3. 对非数值类型的值使用一元运算符（即+和-）。

```javascript
+ {foo: 'bar'} // NaN
- [1, 2, 3] // NaN
```
#### 自动转为数值
JavaScript 遇到预期为数值的地方，就会将参数值自动转换为数值。系统内部会自动调用Number函数。

除了加法运算符（+）有可能把运算子转为字符串，其他运算符都会把运算子自动转成数值。

```javascript
'5' - '2' // 3
'5' * '2' // 10
true - 1  // 0
false - 1 // -1
'1' - 1   // 0
'5' * []    // 0
false / '5' // 0
'abc' - 1   // NaN
null + 1 // 1
undefined + 1 // NaN
```

除此之外**一元运算符**也会把运算子转成数值。
```javascript
+'abc' // NaN
-'abc' // NaN
+true // 1
-false // 0
```

#### 自动转换为字符串
avaScript 遇到预期为字符串的地方，就会将非字符串的值自动转为字符串。
具体规则是，先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串。

字符串的自动转换，主要发生在字符串的加法运算时。当一个值为字符串，另一个值为非字符串，则后者转为字符串。

```javascript
'5' + 1 // '51'
'5' + true // "5true"
'5' + false // "5false"
'5' + {} // "5[object Object]"
'5' + [] // "5"
'5' + function (){} // "5function (){}"
'5' + undefined // "5undefined"
'5' + null // "5null"
```

#### 自动转为布尔值
```javascript
// 写法一
expression ? true : false

// 写法二
!! expression

// 否定运算符
if ( !undefined
  && !null
  && !0
  && !NaN
  && !''
) {
  console.log('true');
} // true

```
