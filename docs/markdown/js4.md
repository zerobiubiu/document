[返回](./JavaScript教程.md)

# 数据类型的转换

## 概述

JavaScript 是一种动态类型语言，变量没有类型限制，可以随时赋予任意值。

```javascript
var x = y ? 1 : 'a';
```

上面代码中，变量`x`到底是数值还是字符串，取决于另一个变量`y`的值。`y`为`true`时，`x`是一个数值；`y`为`false`时，`x`是一个字符串。这意味着，`x`的类型没法在编译阶段就知道，必须等到运行时才能知道。

虽然变量的数据类型是不确定的，但是各种运算符对数据类型是有要求的。如果运算符发现，运算子的类型与预期不符，就会自动转换类型。比如，减法运算符预期左右两侧的运算子应该是数值，如果不是，就会自动将它们转为数值。

```javascript
'4' - '3' // 1
```

上面代码中，虽然是两个字符串相减，但是依然会得到结果数值`1`，原因就在于 JavaScript 将运算子自动转为了数值。

本章讲解数据类型自动转换的规则。在此之前，先讲解如何手动强制转换数据类型。

## 强制转换

强制转换主要指使用`Number()`、`String()`和`Boolean()`三个函数，手动将各种类型的值，分别转换成数字、字符串或者布尔值。

### Number()

使用`Number`函数，可以将任意类型的值转化成数值。

下面分成两种情况讨论，一种是参数是原始类型的值，另一种是参数是对象。

**（1）原始类型值**

原始类型值的转换规则如下。

```javascript
// 数值：转换后还是原来的值
Number(324) // 324

// 字符串：如果可以被解析为数值，则转换为相应的数值
Number('324') // 324

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

`Number`函数将字符串转为数值，要比`parseInt`函数严格很多。基本上，只要有一个字符无法转成数值，整个字符串就会被转为`NaN`。

```javascript
parseInt('42 cats') // 42
Number('42 cats') // NaN
```

上面代码中，`parseInt`逐个解析字符，而`Number`函数整体转换字符串的类型。

另外，`parseInt`和`Number`函数都会自动过滤一个字符串前导和后缀的空格。

```javascript
parseInt('\t\v\r12.34\n') // 12
Number('\t\v\r12.34\n') // 12.34
```

**（2）对象**

简单的规则是，`Number`方法的参数是对象时，将返回`NaN`，除非是包含单个数值的数组。

```javascript
Number({a: 1}) // NaN
Number([1, 2, 3]) // NaN
Number([5]) // 5
```

之所以会这样，是因为`Number`背后的转换规则比较复杂。

第一步，调用对象自身的`valueOf`方法。如果返回原始类型的值，则直接对该值使用`Number`函数，不再进行后续步骤。

第二步，如果`valueOf`方法返回的还是对象，则改为调用对象自身的`toString`方法。如果`toString`方法返回原始类型的值，则对该值使用`Number`函数，不再进行后续步骤。

第三步，如果`toString`方法返回的是对象，就报错。

请看下面的例子。

```javascript
var obj = {x: 1};
Number(obj) // NaN

// 等同于
if (typeof obj.valueOf() === 'object') {
  Number(obj.toString());
} else {
  Number(obj.valueOf());
}
```

上面代码中，`Number`函数将`obj`对象转为数值。背后发生了一连串的操作，首先调用`obj.valueOf`方法, 结果返回对象本身；于是，继续调用`obj.toString`方法，这时返回字符串`[object Object]`，对这个字符串使用`Number`函数，得到`NaN`。

默认情况下，对象的`valueOf`方法返回对象本身，所以一般总是会调用`toString`方法，而`toString`方法返回对象的类型字符串（比如`[object Object]`）。所以，会有下面的结果。

```javascript
Number({}) // NaN
```

如果`toString`方法返回的不是原始类型的值，结果就会报错。

```javascript
var obj = {
  valueOf: function () {
    return {};
  },
  toString: function () {
    return {};
  }
};

Number(obj)
// TypeError: Cannot convert object to primitive value
```

上面代码的`valueOf`和`toString`方法，返回的都是对象，所以转成数值时会报错。

从上例还可以看到，`valueOf`和`toString`方法，都是可以自定义的。

```javascript
Number({
  valueOf: function () {
    return 2;
  }
})
// 2

Number({
  toString: function () {
    return 3;
  }
})
// 3

Number({
  valueOf: function () {
    return 2;
  },
  toString: function () {
    return 3;
  }
})
// 2
```

上面代码对三个对象使用`Number`函数。第一个对象返回`valueOf`方法的值，第二个对象返回`toString`方法的值，第三个对象表示`valueOf`方法先于`toString`方法执行。

### String()

`String`函数可以将任意类型的值转化成字符串，转换规则如下。

**（1）原始类型值**

- **数值**：转为相应的字符串。
- **字符串**：转换后还是原来的值。
- **布尔值**：`true`转为字符串`"true"`，`false`转为字符串`"false"`。
- **undefined**：转为字符串`"undefined"`。
- **null**：转为字符串`"null"`。

```javascript
String(123) // "123"
String('abc') // "abc"
String(true) // "true"
String(undefined) // "undefined"
String(null) // "null"
```

**（2）对象**

`String`方法的参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。

```javascript
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
```

`String`方法背后的转换规则，与`Number`方法基本相同，只是互换了`valueOf`方法和`toString`方法的执行顺序。

1. 先调用对象自身的`toString`方法。如果返回原始类型的值，则对该值使用`String`函数，不再进行以下步骤。

2. 如果`toString`方法返回的是对象，再调用原对象的`valueOf`方法。如果`valueOf`方法返回原始类型的值，则对该值使用`String`函数，不再进行以下步骤。

3. 如果`valueOf`方法返回的是对象，就报错。

下面是一个例子。

```javascript
String({a: 1})
// "[object Object]"

// 等同于
String({a: 1}.toString())
// "[object Object]"
```

上面代码先调用对象的`toString`方法，发现返回的是字符串`[object Object]`，就不再调用`valueOf`方法了。

如果`toString`法和`valueOf`方法，返回的都是对象，就会报错。

```javascript
var obj = {
  valueOf: function () {
    return {};
  },
  toString: function () {
    return {};
  }
};

String(obj)
// TypeError: Cannot convert object to primitive value
```

下面是通过自定义`toString`方法，改变返回值的例子。

```javascript
String({
  toString: function () {
    return 3;
  }
})
// "3"

String({
  valueOf: function () {
    return 2;
  }
})
// "[object Object]"

String({
  valueOf: function () {
    return 2;
  },
  toString: function () {
    return 3;
  }
})
// "3"
```

上面代码对三个对象使用`String`函数。第一个对象返回`toString`方法的值（数值3），第二个对象返回的还是`toString`方法的值（`[object Object]`），第三个对象表示`toString`方法先于`valueOf`方法执行。

### Boolean()

`Boolean()`函数可以将任意类型的值转为布尔值。

它的转换规则相对简单：除了以下五个值的转换结果为`false`，其他的值全部为`true`。

- `undefined`
- `null`
- `0`（包含`-0`和`+0`）
- `NaN`
- `''`（空字符串）

```javascript
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean('') // false
```

当然，`true`和`false`这两个布尔值不会发生变化。

```javascript
Boolean(true) // true
Boolean(false) // false
```

注意，所有对象（包括空对象）的转换结果都是`true`，甚至连`false`对应的布尔对象`new Boolean(false)`也是`true`（详见《原始类型值的包装对象》一章）。

```javascript
Boolean({}) // true
Boolean([]) // true
Boolean(new Boolean(false)) // true
```

所有对象的布尔值都是`true`，这是因为 JavaScript 语言设计的时候，出于性能的考虑，如果对象需要计算才能得到布尔值，对于`obj1 && obj2`这样的场景，可能会需要较多的计算。为了保证性能，就统一规定，对象的布尔值为`true`。

## 自动转换

下面介绍自动转换，它是以强制转换为基础的。

遇到以下三种情况时，JavaScript 会自动转换数据类型，即转换是自动完成的，用户不可见。

第一种情况，不同类型的数据互相运算。

```javascript
123 + 'abc' // "123abc"
```

第二种情况，对非布尔值类型的数据求布尔值。

```javascript
if ('abc') {
  console.log('hello')
}  // "hello"
```

第三种情况，对非数值类型的值使用一元运算符（即`+`和`-`）。

```javascript
+ {foo: 'bar'} // NaN
- [1, 2, 3] // NaN
```

自动转换的规则是这样的：预期什么类型的值，就调用该类型的转换函数。比如，某个位置预期为字符串，就调用`String()`函数进行转换。如果该位置既可以是字符串，也可能是数值，那么默认转为数值。

由于自动转换具有不确定性，而且不易除错，建议在预期为布尔值、数值、字符串的地方，全部使用`Boolean()`、`Number()`和`String()`函数进行显式转换。

### 自动转换为布尔值

JavaScript 遇到预期为布尔值的地方（比如`if`语句的条件部分），就会将非布尔值的参数自动转换为布尔值。系统内部会自动调用`Boolean()`函数。

因此除了以下五个值，其他都是自动转为`true`。

- `undefined`
- `null`
- `+0`或`-0`
- `NaN`
- `''`（空字符串）

下面这个例子中，条件部分的每个值都相当于`false`，使用否定运算符后，就变成了`true`。

```javascript
if ( !undefined
  && !null
  && !0
  && !NaN
  && !''
) {
  console.log('true');
} // true
```

下面两种写法，有时也用于将一个表达式转为布尔值。它们内部调用的也是`Boolean()`函数。

```javascript
// 写法一
expression ? true : false

// 写法二
!! expression
```

### 自动转换为字符串

JavaScript 遇到预期为字符串的地方，就会将非字符串的值自动转为字符串。具体规则是，先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串。

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

这种自动转换很容易出错。

```javascript
var obj = {
  width: '100'
};

obj.width + 20 // "10020"
```

上面代码中，开发者可能期望返回`120`，但是由于自动转换，实际上返回了一个字符`10020`。

### 自动转换为数值

JavaScript 遇到预期为数值的地方，就会将参数值自动转换为数值。系统内部会自动调用`Number()`函数。

除了加法运算符（`+`）有可能把运算子转为字符串，其他运算符都会把运算子自动转成数值。

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

上面代码中，运算符两侧的运算子，都被转成了数值。

> 注意：`null`转为数值时为`0`，而`undefined`转为数值时为`NaN`。

一元运算符也会把运算子转成数值。

```javascript
+'abc' // NaN
-'abc' // NaN
+true // 1
-false // 0
```

## 参考链接

- Axel Rauschmayer, [What is {} + {} in JavaScript?](http://www.2ality.com/2012/01/object-plus-object.html)
- Axel Rauschmayer, [JavaScript quirk 1: implicit conversion of values](http://www.2ality.com/2013/04/quirk-implicit-conversion.html)
- Benjie Gillam, [Quantum JavaScript?](http://www.benjiegillam.com/2013/06/quantum-javascript/)

# 错误处理机制

## Error 实例对象

JavaScript 解析或运行时，一旦发生错误，引擎就会抛出一个错误对象。JavaScript 原生提供`Error`构造函数，所有抛出的错误都是这个构造函数的实例。

```javascript
var err = new Error('出错了');
err.message // "出错了"
```

上面代码中，我们调用`Error()`构造函数，生成一个实例对象`err`。`Error()`构造函数接受一个参数，表示错误提示，可以从实例的`message`属性读到这个参数。抛出`Error`实例对象以后，整个程序就中断在发生错误的地方，不再往下执行。

JavaScript 语言标准只提到，`Error`实例对象必须有`message`属性，表示出错时的提示信息，没有提到其他属性。大多数 JavaScript 引擎，对`Error`实例还提供`name`和`stack`属性，分别表示错误的名称和错误的堆栈，但它们是非标准的，不是每种实现都有。

- **message**：错误提示信息
- **name**：错误名称（非标准属性）
- **stack**：错误的堆栈（非标准属性）

使用`name`和`message`这两个属性，可以对发生什么错误有一个大概的了解。

```javascript
if (error.name) {
  console.log(error.name + ': ' + error.message);
}
```

`stack`属性用来查看错误发生时的堆栈。

```javascript
function throwit() {
  throw new Error('');
}

function catchit() {
  try {
    throwit();
  } catch(e) {
    console.log(e.stack); // print stack trace
  }
}

catchit()
// Error
//    at throwit (~/examples/throwcatch.js:9:11)
//    at catchit (~/examples/throwcatch.js:3:9)
//    at repl:1:5
```

上面代码中，错误堆栈的最内层是`throwit`函数，然后是`catchit`函数，最后是函数的运行环境。

## 原生错误类型

`Error`实例对象是最一般的错误类型，在它的基础上，JavaScript 还定义了其他6种错误对象。也就是说，存在`Error`的6个派生对象。

### SyntaxError 对象

`SyntaxError`对象是解析代码时发生的语法错误。

```javascript
// 变量名错误
var 1a;
// Uncaught SyntaxError: Invalid or unexpected token

// 缺少括号
console.log 'hello');
// Uncaught SyntaxError: Unexpected string
```

上面代码的错误，都是在语法解析阶段就可以发现，所以会抛出`SyntaxError`。第一个错误提示是“token 非法”，第二个错误提示是“字符串不符合要求”。

### ReferenceError 对象

`ReferenceError`对象是引用一个不存在的变量时发生的错误。

```javascript
// 使用一个不存在的变量
unknownVariable
// Uncaught ReferenceError: unknownVariable is not defined
```

另一种触发场景是，将一个值分配给无法分配的对象，比如对函数的运行结果赋值。

```javascript
// 等号左侧不是变量
console.log() = 1
// Uncaught ReferenceError: Invalid left-hand side in assignment
```

上面代码对函数`console.log`的运行结果赋值，结果引发了`ReferenceError`错误。

### RangeError 对象

`RangeError`对象是一个值超出有效范围时发生的错误。主要有几种情况，一是数组长度为负数，二是`Number`对象的方法参数超出范围，以及函数堆栈超过最大值。

```javascript
// 数组长度不得为负数
new Array(-1)
// Uncaught RangeError: Invalid array length
```

### TypeError 对象

`TypeError`对象是变量或参数不是预期类型时发生的错误。比如，对字符串、布尔值、数值等原始类型的值使用`new`命令，就会抛出这种错误，因为`new`命令的参数应该是一个构造函数。

```javascript
new 123
// Uncaught TypeError: 123 is not a constructor

var obj = {};
obj.unknownMethod()
// Uncaught TypeError: obj.unknownMethod is not a function
```

上面代码的第二种情况，调用对象不存在的方法，也会抛出`TypeError`错误，因为`obj.unknownMethod`的值是`undefined`，而不是一个函数。

### URIError 对象

`URIError`对象是 URI 相关函数的参数不正确时抛出的错误，主要涉及`encodeURI()`、`decodeURI()`、`encodeURIComponent()`、`decodeURIComponent()`、`escape()`和`unescape()`这六个函数。

```javascript
decodeURI('%2')
// URIError: URI malformed
```

### EvalError 对象

`eval`函数没有被正确执行时，会抛出`EvalError`错误。该错误类型已经不再使用了，只是为了保证与以前代码兼容，才继续保留。

### 总结

以上这6种派生错误，连同原始的`Error`对象，都是构造函数。开发者可以使用它们，手动生成错误对象的实例。这些构造函数都接受一个参数，代表错误提示信息（message）。

```javascript
var err1 = new Error('出错了！');
var err2 = new RangeError('出错了，变量超出有效范围！');
var err3 = new TypeError('出错了，变量类型无效！');

err1.message // "出错了！"
err2.message // "出错了，变量超出有效范围！"
err3.message // "出错了，变量类型无效！"
```

## 自定义错误

除了 JavaScript 原生提供的七种错误对象，还可以定义自己的错误对象。

```javascript
function UserError(message) {
  this.message = message || '默认信息';
  this.name = 'UserError';
}

UserError.prototype = new Error();
UserError.prototype.constructor = UserError;
```

上面代码自定义一个错误对象`UserError`，让它继承`Error`对象。然后，就可以生成这种自定义类型的错误了。

```javascript
new UserError('这是自定义的错误！');
```

## throw 语句

`throw`语句的作用是手动中断程序执行，抛出一个错误。

```javascript
var x = -1;

if (x <= 0) {
  throw new Error('x 必须为正数');
}
// Uncaught Error: x 必须为正数
```

上面代码中，如果变量`x`小于等于`0`，就手动抛出一个错误，告诉用户`x`的值不正确，整个程序就会在这里中断执行。可以看到，`throw`抛出的错误就是它的参数，这里是一个`Error`对象的实例。

`throw`也可以抛出自定义错误。

```javascript
function UserError(message) {
  this.message = message || '默认信息';
  this.name = 'UserError';
}

throw new UserError('出错了！');
// Uncaught UserError {message: "出错了！", name: "UserError"}
```

上面代码中，`throw`抛出的是一个`UserError`实例。

实际上，`throw`可以抛出任何类型的值。也就是说，它的参数可以是任何值。

```javascript
// 抛出一个字符串
throw 'Error！';
// Uncaught Error！

// 抛出一个数值
throw 42;
// Uncaught 42

// 抛出一个布尔值
throw true;
// Uncaught true

// 抛出一个对象
throw {
  toString: function () {
    return 'Error!';
  }
};
// Uncaught {toString: ƒ}
```

对于 JavaScript 引擎来说，遇到`throw`语句，程序就中止了。引擎会接收到`throw`抛出的信息，可能是一个错误实例，也可能是其他类型的值。

## try...catch 结构

一旦发生错误，程序就中止执行了。JavaScript 提供了`try...catch`结构，允许对错误进行处理，选择是否往下执行。

```javascript
try {
  throw new Error('出错了!');
} catch (e) {
  console.log(e.name + ": " + e.message);
  console.log(e.stack);
}
// Error: 出错了!
//   at <anonymous>:3:9
//   ...
```

上面代码中，`try`代码块抛出错误（上例用的是`throw`语句），JavaScript 引擎就立即把代码的执行，转到`catch`代码块，或者说错误被`catch`代码块捕获了。`catch`接受一个参数，表示`try`代码块抛出的值。

如果你不确定某些代码是否会报错，就可以把它们放在`try...catch`代码块之中，便于进一步对错误进行处理。

```javascript
try {
  f();
} catch(e) {
  // 处理错误
}
```

上面代码中，如果函数`f`执行报错，就会进行`catch`代码块，接着对错误进行处理。

`catch`代码块捕获错误之后，程序不会中断，会按照正常流程继续执行下去。

```javascript
try {
  throw "出错了";
} catch (e) {
  console.log(111);
}
console.log(222);
// 111
// 222
```

上面代码中，`try`代码块抛出的错误，被`catch`代码块捕获后，程序会继续向下执行。

`catch`代码块之中，还可以再抛出错误，甚至使用嵌套的`try...catch`结构。

```javascript
var n = 100;

try {
  throw n;
} catch (e) {
  if (e <= 50) {
    // ...
  } else {
    throw e;
  }
}
// Uncaught 100
```

上面代码中，`catch`代码之中又抛出了一个错误。

为了捕捉不同类型的错误，`catch`代码块之中可以加入判断语句。

```javascript
try {
  foo.bar();
} catch (e) {
  if (e instanceof EvalError) {
    console.log(e.name + ": " + e.message);
  } else if (e instanceof RangeError) {
    console.log(e.name + ": " + e.message);
  }
  // ...
}
```

上面代码中，`catch`捕获错误之后，会判断错误类型（`EvalError`还是`RangeError`），进行不同的处理。

## finally 代码块

`try...catch`结构允许在最后添加一个`finally`代码块，表示不管是否出现错误，都必需在最后运行的语句。

```javascript
function cleansUp() {
  try {
    throw new Error('出错了……');
    console.log('此行不会执行');
  } finally {
    console.log('完成清理工作');
  }
}

cleansUp()
// 完成清理工作
// Uncaught Error: 出错了……
//    at cleansUp (<anonymous>:3:11)
//    at <anonymous>:10:1
```

上面代码中，由于没有`catch`语句块，一旦发生错误，代码就会中断执行。中断执行之前，会先执行`finally`代码块，然后再向用户提示报错信息。

```javascript
function idle(x) {
  try {
    console.log(x);
    return 'result';
  } finally {
    console.log('FINALLY');
  }
}

idle('hello')
// hello
// FINALLY
```

上面代码中，`try`代码块没有发生错误，而且里面还包括`return`语句，但是`finally`代码块依然会执行。而且，这个函数的返回值还是`result`。

下面的例子说明，`return`语句的执行是排在`finally`代码之前，只是等`finally`代码执行完毕后才返回。

```javascript
var count = 0;
function countUp() {
  try {
    return count;
  } finally {
    count++;
  }
}

countUp()
// 0
count
// 1
```

上面代码说明，`return`语句里面的`count`的值，是在`finally`代码块运行之前就获取了。

下面是`finally`代码块用法的典型场景。

```javascript
openFile();

try {
  writeFile(Data);
} catch(e) {
  handleError(e);
} finally {
  closeFile();
}
```

上面代码首先打开一个文件，然后在`try`代码块中写入文件，如果没有发生错误，则运行`finally`代码块关闭文件；一旦发生错误，则先使用`catch`代码块处理错误，再使用`finally`代码块关闭文件。

下面的例子充分反映了`try...catch...finally`这三者之间的执行顺序。

```javascript
function f() {
  try {
    console.log(0);
    throw 'bug';
  } catch(e) {
    console.log(1);
    return true; // 这句原本会延迟到 finally 代码块结束再执行
    console.log(2); // 不会运行
  } finally {
    console.log(3);
    return false; // 这句会覆盖掉前面那句 return
    console.log(4); // 不会运行
  }

  console.log(5); // 不会运行
}

var result = f();
// 0
// 1
// 3

result
// false
```

上面代码中，`catch`代码块结束执行之前，会先执行`finally`代码块。

`catch`代码块之中，触发转入`finally`代码块的标志，不仅有`return`语句，还有`throw`语句。

```javascript
function f() {
  try {
    throw '出错了！';
  } catch(e) {
    console.log('捕捉到内部错误');
    throw e; // 这句原本会等到finally结束再执行
  } finally {
    return false; // 直接返回
  }
}

try {
  f();
} catch(e) {
  // 此处不会执行
  console.log('caught outer "bogus"');
}

//  捕捉到内部错误
```

上面代码中，进入`catch`代码块之后，一遇到`throw`语句，就会去执行`finally`代码块，其中有`return false`语句，因此就直接返回了，不再会回去执行`catch`代码块剩下的部分了。

`try`代码块内部，还可以再使用`try`代码块。

```javascript
try {
  try {
    consle.log('Hello world!'); // 报错
  }
  finally {
    console.log('Finally');
  }
  console.log('Will I run?');
} catch(error) {
  console.error(error.message);
}
// Finally
// consle is not defined
```

上面代码中，`try`里面还有一个`try`。内层的`try`报错（`console`拼错了），这时会执行内层的`finally`代码块，然后抛出错误，被外层的`catch`捕获。

## 参考连接

- Jani Hartikainen, [JavaScript Errors and How to Fix Them](http://davidwalsh.name/fix-javascript-errors)

# 编程风格

## 概述

“编程风格”（programming style）指的是编写代码的样式规则。不同的程序员，往往有不同的编程风格。

有人说，编译器的规范叫做“语法规则”（grammar），这是程序员必须遵守的；而编译器忽略的部分，就叫“编程风格”（programming style），这是程序员可以自由选择的。这种说法不完全正确，程序员固然可以自由选择编程风格，但是好的编程风格有助于写出质量更高、错误更少、更易于维护的程序。

所以，编程风格的选择不应该基于个人爱好、熟悉程度、打字量等因素，而要考虑如何尽量使代码清晰易读、减少出错。你选择的，不是你喜欢的风格，而是一种能够清晰表达你的意图的风格。这一点，对于 JavaScript 这种语法自由度很高的语言尤其重要。

必须牢记的一点是，如果你选定了一种“编程风格”，就应该坚持遵守，切忌多种风格混用。如果你加入他人的项目，就应该遵守现有的风格。

## 缩进

行首的空格和 Tab 键，都可以产生代码缩进效果（indent）。

Tab 键可以节省击键次数，但不同的文本编辑器对 Tab 的显示不尽相同，有的显示四个空格，有的显示两个空格，所以有人觉得，空格键可以使得显示效果更统一。

无论你选择哪一种方法，都是可以接受的，要做的就是始终坚持这一种选择。不要一会使用 Tab 键，一会使用空格键。

## 区块

如果循环和判断的代码体只有一行，JavaScript 允许该区块（block）省略大括号。

```javascript
if (a)
  b();
  c();
```

上面代码的原意可能是下面这样。

```javascript
if (a) {
  b();
  c();
}
```

但是，实际效果却是下面这样。

```javascript
if (a) {
  b();
}
  c();
```

因此，建议总是使用大括号表示区块。

另外，区块起首的大括号的位置，有许多不同的写法。最流行的有两种，一种是起首的大括号另起一行。

```javascript
block
{
  // ...
}
```

另一种是起首的大括号跟在关键字的后面。

```javascript
block {
  // ...
}
```

一般来说，这两种写法都可以接受。但是，JavaScript 要使用后一种，因为 JavaScript 会自动添加句末的分号，导致一些难以察觉的错误。

```javascript
return
{
  key: value
};

// 相当于
return;
{
  key: value
};
```

上面的代码的原意，是要返回一个对象，但实际上返回的是`undefined`，因为 JavaScript 自动在`return`语句后面添加了分号。为了避免这一类错误，需要写成下面这样。

```javascript
return {
  key : value
};
```

因此，表示区块起首的大括号，不要另起一行。

## 圆括号

圆括号（parentheses）在 JavaScript 中有两种作用，一种表示函数的调用，另一种表示表达式的组合（grouping）。

```javascript
// 圆括号表示函数的调用
console.log('abc');

// 圆括号表示表达式的组合
(1 + 2) * 3
```

建议可以用空格，区分这两种不同的括号。

> 1. 表示函数调用时，函数名与左括号之间没有空格。
>
> 2. 表示函数定义时，函数名与左括号之间没有空格。
>
> 3. 其他情况时，前面位置的语法元素与左括号之间，都有一个空格。

按照上面的规则，下面的写法都是不规范的。

```javascript
foo (bar)
return(a+b);
if(a === 0) {...}
function foo (b) {...}
function(x) {...}
```

上面代码的最后一行是一个匿名函数，`function`是语法关键字，不是函数名，所以与左括号之间应该要有一个空格。

## 行尾的分号

分号表示一条语句的结束。JavaScript 允许省略行尾的分号。事实上，确实有一些开发者行尾从来不写分号。但是，由于下面要讨论的原因，建议还是不要省略这个分号。

### 不使用分号的情况

首先，以下三种情况，语法规定本来就不需要在结尾添加分号。

**（1）for 和 while 循环**

```javascript
for ( ; ; ) {
} // 没有分号

while (true) {
} // 没有分号
```

注意，`do...while`循环是有分号的。

```javascript
do {
  a--;
} while(a > 0); // 分号不能省略
```

**（2）分支语句：if，switch，try**

```javascript
if (true) {
} // 没有分号

switch () {
} // 没有分号

try {
} catch {
} // 没有分号
```

**（3）函数的声明语句**

```javascript
function f() {
} // 没有分号
```

注意，函数表达式仍然要使用分号。

```javascript
var f = function f() {
};
```

以上三种情况，如果使用了分号，并不会出错。因为，解释引擎会把这个分号解释为空语句。

### 分号的自动添加

除了上一节的三种情况，所有语句都应该使用分号。但是，如果没有使用分号，大多数情况下，JavaScript 会自动添加。

```javascript
var a = 1
// 等同于
var a = 1;
```

这种语法特性被称为“分号的自动添加”（Automatic Semicolon Insertion，简称 ASI）。

因此，有人提倡省略句尾的分号。麻烦的是，如果下一行的开始可以与本行的结尾连在一起解释，JavaScript 就不会自动添加分号。

```javascript
// 等同于 var a = 3
var
a
=
3

// 等同于 'abc'.length
'abc'
.length

// 等同于 return a + b;
return a +
b;

// 等同于 obj.foo(arg1, arg2);
obj.foo(arg1,
arg2);

// 等同于 3 * 2 + 10 * (27 / 6)
3 * 2
+
10 * (27 / 6)
```

上面代码都会多行放在一起解释，不会每一行自动添加分号。这些例子还是比较容易看出来的，但是下面这个例子就不那么容易看出来了。

```javascript
x = y
(function () {
  // ...
})();

// 等同于
x = y(function () {...})();
```

下面是更多不会自动添加分号的例子。

```javascript
// 引擎解释为 c(d+e)
var a = b + c
(d+e).toString();

// 引擎解释为 a = b/hi/g.exec(c).map(d)
// 正则表达式的斜杠，会当作除法运算符
a = b
/hi/g.exec(c).map(d);

// 解释为'b'['red', 'green']，
// 即把字符串当作一个数组，按索引取值
var a = 'b'
['red', 'green'].forEach(function (c) {
  console.log(c);
})

// 解释为 function (x) { return x }(a++)
// 即调用匿名函数，结果f等于0
var a = 0;
var f = function (x) { return x }
(a++)
```

只有下一行的开始与本行的结尾，无法放在一起解释，JavaScript 引擎才会自动添加分号。

```javascript
if (a < 0) a = 0
console.log(a)

// 等同于下面的代码，
// 因为 0console 没有意义
if (a < 0) a = 0;
console.log(a)
```

另外，如果一行的起首是“自增”（`++`）或“自减”（`--`）运算符，则它们的前面会自动添加分号。

```javascript
a = b = c = 1

a
++
b
--
c

console.log(a, b, c)
// 1 2 0
```

上面代码之所以会得到`1 2 0`的结果，原因是自增和自减运算符前，自动加上了分号。上面的代码实际上等同于下面的形式。

```javascript
a = b = c = 1;
a;
++b;
--c;
```

如果`continue`、`break`、`return`和`throw`这四个语句后面，直接跟换行符，则会自动添加分号。这意味着，如果`return`语句返回的是一个对象的字面量，起首的大括号一定要写在同一行，否则得不到预期结果。

```javascript
return
{ first: 'Jane' };

// 解释成
return;
{ first: 'Jane' };
```

由于解释引擎自动添加分号的行为难以预测，因此编写代码的时候不应该省略行尾的分号。

不应该省略结尾的分号，还有一个原因。有些 JavaScript 代码压缩器（uglifier）不会自动添加分号，因此遇到没有分号的结尾，就会让代码保持原状，而不是压缩成一行，使得压缩无法得到最优的结果。

另外，不写结尾的分号，可能会导致脚本合并出错。所以，有的代码库在第一行语句开始前，会加上一个分号。

```javascript
;var a = 1;
// ...
```

上面这种写法就可以避免与其他脚本合并时，排在前面的脚本最后一行语句没有分号，导致运行出错的问题。

## 全局变量

JavaScript 最大的语法缺点，可能就是全局变量对于任何一个代码块，都是可读可写。这对代码的模块化和重复使用，非常不利。

因此，建议避免使用全局变量。如果不得不使用，可以考虑用大写字母表示变量名，这样更容易看出这是全局变量，比如`UPPER_CASE`。

## 变量声明

JavaScript 会自动将变量声明“提升”（hoist）到代码块（block）的头部。

```javascript
if (!x) {
  var x = {};
}

// 等同于
var x;
if (!x) {
  x = {};
}
```

这意味着，变量`x`是`if`代码块之前就存在了。为了避免可能出现的问题，最好把变量声明都放在代码块的头部。

```javascript
for (var i = 0; i < 10; i++) {
  // ...
}

// 写成
var i;
for (i = 0; i < 10; i++) {
  // ...
}
```

上面这样的写法，就容易看出存在一个全局的循环变量`i`。

另外，所有函数都应该在使用之前定义。函数内部的变量声明，都应该放在函数的头部。

## with 语句

`with`可以减少代码的书写，但是会造成混淆。

```javascript
with (o) {
　foo = bar;
}
```

上面的代码，可以有四种运行结果：

```javascript
o.foo = bar;
o.foo = o.bar;
foo = bar;
foo = o.bar;
```

这四种结果都可能发生，取决于不同的变量是否有定义。因此，不要使用`with`语句。

## 相等和严格相等

JavaScript 有两个表示相等的运算符：“相等”（`==`）和“严格相等”（`===`）。

相等运算符会自动转换变量类型，造成很多意想不到的情况。

```javascript
0 == ''// true
1 == true // true
2 == true // false
0 == '0' // true
false == 'false' // false
false == '0' // true
' \t\r\n ' == 0 // true
```

因此，建议不要使用相等运算符（`==`），只使用严格相等运算符（`===`）。

## 语句的合并

有些程序员追求简洁，喜欢合并不同目的的语句。比如，原来的语句是

```javascript
a = b;
if (a) {
  // ...
}
```

他喜欢写成下面这样。

```javascript
if (a = b) {
  // ...
}
```

虽然语句少了一行，但是可读性大打折扣，而且会造成误读，让别人误解这行代码的意思是下面这样。

```javascript
if （a === b）{
  // ...
}
```

建议不要将不同目的的语句，合并成一行。

## 自增和自减运算符

自增（`++`）和自减（`--`）运算符，放在变量的前面或后面，返回的值不一样，很容易发生错误。事实上，所有的`++`运算符都可以用`+= 1`代替。

```javascript
++x
// 等同于
x += 1;
```

改用`+= 1`，代码变得更清晰了。

建议自增（`++`）和自减（`--`）运算符尽量使用`+=`和`-=`代替。

## switch...case 结构

`switch...case`结构要求，在每一个`case`的最后一行必须是`break`语句，否则会接着运行下一个`case`。这样不仅容易忘记，还会造成代码的冗长。

而且，`switch...case`不使用大括号，不利于代码形式的统一。此外，这种结构类似于`goto`语句，容易造成程序流程的混乱，使得代码结构混乱不堪，不符合面向对象编程的原则。

```javascript
function doAction(action) {
  switch (action) {
    case 'hack':
      return 'hack';
    case 'slash':
      return 'slash';
    case 'run':
      return 'run';
    default:
      throw new Error('Invalid action.');
  }
}
```

上面的代码建议改写成对象结构。

```javascript
function doAction(action) {
  var actions = {
    'hack': function () {
      return 'hack';
    },
    'slash': function () {
      return 'slash';
    },
    'run': function () {
      return 'run';
    }
  };

  if (typeof actions[action] !== 'function') {
    throw new Error('Invalid action.');
  }

  return actions[action]();
}
```

因此，建议`switch...case`结构可以用对象结构代替。

## 参考链接

- Eric Elliott, Programming JavaScript Applications, [Chapter 2. JavaScript Style Guide](http://chimera.labs.oreilly.com/books/1234000000262/ch02.html), O'Reilly, 2013
- Axel Rauschmayer, [A meta style guide for JavaScript](http://www.2ality.com/2013/07/meta-style-guide.html)
- Axel Rauschmayer, [Automatic semicolon insertion in JavaScript](http://www.2ality.com/2011/05/semicolon-insertion.html)
- Rod Vagg, [JavaScript and Semicolons](http://dailyjs.com/2012/04/19/semicolons/)

# console 对象与控制台

## console 对象

`console`对象是 JavaScript 的原生对象，它有点像 Unix 系统的标准输出`stdout`和标准错误`stderr`，可以输出各种信息到控制台，并且还提供了很多有用的辅助方法。

`console`的常见用途有两个。

- 调试程序，显示网页代码运行时的错误信息。
- 提供了一个命令行接口，用来与网页代码互动。

`console`对象的浏览器实现，包含在浏览器自带的开发工具之中。以 Chrome 浏览器的“开发者工具”（Developer Tools）为例，可以使用下面三种方法的打开它。

1. 按 F12 或者`Control + Shift + i`（PC）/ `Command + Option + i`（Mac）。
2. 浏览器菜单选择“工具/开发者工具”。
3. 在一个页面元素上，打开右键菜单，选择其中的“Inspect Element”。

打开开发者工具以后，顶端有多个面板。

- **Elements**：查看网页的 HTML 源码和 CSS 代码。
- **Resources**：查看网页加载的各种资源文件（比如代码文件、字体文件 CSS 文件等），以及在硬盘上创建的各种内容（比如本地缓存、Cookie、Local Storage等）。
- **Network**：查看网页的 HTTP 通信情况。
- **Sources**：查看网页加载的脚本源码。
- **Timeline**：查看各种网页行为随时间变化的情况。
- **Performance**：查看网页的性能情况，比如 CPU 和内存消耗。
- **Console**：用来运行 JavaScript 命令。

这些面板都有各自的用途，以下只介绍`Console`面板（又称为控制台）。

`Console`面板基本上就是一个命令行窗口，你可以在提示符下，键入各种命令。

## console 对象的静态方法

`console`对象提供的各种静态方法，用来与控制台窗口互动。

### console.log()，console.info()，console.debug()

`console.log`方法用于在控制台输出信息。它可以接受一个或多个参数，将它们连接起来输出。

```javascript
console.log('Hello World')
// Hello World
console.log('a', 'b', 'c')
// a b c
```

`console.log`方法会自动在每次输出的结尾，添加换行符。

```javascript
console.log(1);
console.log(2);
console.log(3);
// 1
// 2
// 3
```

如果第一个参数是格式字符串（使用了格式占位符），`console.log`方法将依次用后面的参数替换占位符，然后再进行输出。

```javascript
console.log(' %s + %s = %s', 1, 1, 2)
//  1 + 1 = 2
```

上面代码中，`console.log`方法的第一个参数有三个占位符（`%s`），第二、三、四个参数会在显示时，依次替换掉这个三个占位符。

`console.log`方法支持以下占位符，不同类型的数据必须使用对应的占位符。

- `%s` 字符串
- `%d` 整数
- `%i` 整数
- `%f` 浮点数
- `%o` 对象的链接
- `%c` CSS 格式字符串

```javascript
var number = 11 * 9;
var color = 'red';

console.log('%d %s balloons', number, color);
// 99 red balloons
```

上面代码中，第二个参数是数值，对应的占位符是`%d`，第三个参数是字符串，对应的占位符是`%s`。

使用`%c`占位符时，对应的参数必须是 CSS 代码，用来对输出内容进行 CSS 渲染。

```javascript
console.log(
  '%cThis text is styled!',
  'color: red; background: yellow; font-size: 24px;'
)
```

上面代码运行后，输出的内容将显示为黄底红字。

`console.log`方法的两种参数格式，可以结合在一起使用。

```javascript
console.log(' %s + %s ', 1, 1, '= 2')
// 1 + 1  = 2
```

如果参数是一个对象，`console.log`会显示该对象的值。

```javascript
console.log({foo: 'bar'})
// Object {foo: "bar"}
console.log(Date)
// function Date() { [native code] }
```

上面代码输出`Date`对象的值，结果为一个构造函数。

`console.info`是`console.log`方法的别名，用法完全一样。只不过`console.info`方法会在输出信息的前面，加上一个蓝色图标。

`console.debug`方法与`console.log`方法类似，会在控制台输出调试信息。但是，默认情况下，`console.debug`输出的信息不会显示，只有在打开显示级别在`verbose`的情况下，才会显示。

`console`对象的所有方法，都可以被覆盖。因此，可以按照自己的需要，定义`console.log`方法。

```javascript
['log', 'info', 'warn', 'error'].forEach(function(method) {
  console[method] = console[method].bind(
    console,
    new Date().toISOString()
  );
});

console.log("出错了！");
// 2014-05-18T09:00.000Z 出错了！
```

上面代码表示，使用自定义的`console.log`方法，可以在显示结果添加当前时间。

### console.warn()，console.error()

`warn`方法和`error`方法也是在控制台输出信息，它们与`log`方法的不同之处在于，`warn`方法输出信息时，在最前面加一个黄色三角，表示警告；`error`方法输出信息时，在最前面加一个红色的叉，表示出错。同时，还会高亮显示输出文字和错误发生的堆栈。其他方面都一样。

```javascript
console.error('Error: %s (%i)', 'Server is not responding', 500)
// Error: Server is not responding (500)
console.warn('Warning! Too few nodes (%d)', document.childNodes.length)
// Warning! Too few nodes (1)
```

可以这样理解，`log`方法是写入标准输出（`stdout`），`warn`方法和`error`方法是写入标准错误（`stderr`）。

### console.table()

对于某些复合类型的数据，`console.table`方法可以将其转为表格显示。

```javascript
var languages = [
  { name: "JavaScript", fileExtension: ".js" },
  { name: "TypeScript", fileExtension: ".ts" },
  { name: "CoffeeScript", fileExtension: ".coffee" }
];

console.table(languages);
```

上面代码的`language`变量，转为表格显示如下。

(index)|name|fileExtension
-------|----|-------------
0|"JavaScript"|".js"
1|"TypeScript"|".ts"
2|"CoffeeScript"|".coffee"

下面是显示表格内容的例子。

```javascript
var languages = {
  csharp: { name: "C#", paradigm: "object-oriented" },
  fsharp: { name: "F#", paradigm: "functional" }
};

console.table(languages);
```

上面代码的`language`，转为表格显示如下。

(index)|name|paradigm
-------|----|--------
csharp|"C#"|"object-oriented"
fsharp|"F#"|"functional"

### console.count()

`count`方法用于计数，输出它被调用了多少次。

```javascript
function greet(user) {
  console.count();
  return 'hi ' + user;
}

greet('bob')
//  : 1
// "hi bob"

greet('alice')
//  : 2
// "hi alice"

greet('bob')
//  : 3
// "hi bob"
```

上面代码每次调用`greet`函数，内部的`console.count`方法就输出执行次数。

该方法可以接受一个字符串作为参数，作为标签，对执行次数进行分类。

```javascript
function greet(user) {
  console.count(user);
  return "hi " + user;
}

greet('bob')
// bob: 1
// "hi bob"

greet('alice')
// alice: 1
// "hi alice"

greet('bob')
// bob: 2
// "hi bob"
```

上面代码根据参数的不同，显示`bob`执行了两次，`alice`执行了一次。

### console.dir()，console.dirxml()

`dir`方法用来对一个对象进行检查（inspect），并以易于阅读和打印的格式显示。

```javascript
console.log({f1: 'foo', f2: 'bar'})
// Object {f1: "foo", f2: "bar"}

console.dir({f1: 'foo', f2: 'bar'})
// Object
//   f1: "foo"
//   f2: "bar"
//   __proto__: Object
```

上面代码显示`dir`方法的输出结果，比`log`方法更易读，信息也更丰富。

该方法对于输出 DOM 对象非常有用，因为会显示 DOM 对象的所有属性。

```javascript
console.dir(document.body)
```

Node 环境之中，还可以指定以代码高亮的形式输出。

```javascript
console.dir(obj, {colors: true})
```

`dirxml`方法主要用于以目录树的形式，显示 DOM 节点。

```javascript
console.dirxml(document.body)
```

如果参数不是 DOM 节点，而是普通的 JavaScript 对象，`console.dirxml`等同于`console.dir`。

```javascript
console.dirxml([1, 2, 3])
// 等同于
console.dir([1, 2, 3])
```

### console.assert()

`console.assert`方法主要用于程序运行过程中，进行条件判断，如果不满足条件，就显示一个错误，但不会中断程序执行。这样就相当于提示用户，内部状态不正确。

它接受两个参数，第一个参数是表达式，第二个参数是字符串。只有当第一个参数为`false`，才会提示有错误，在控制台输出第二个参数，否则不会有任何结果。

```javascript
console.assert(false, '判断条件不成立')
// Assertion failed: 判断条件不成立

// 相当于
try {
  if (!false) {
    throw new Error('判断条件不成立');
  }
} catch(e) {
  console.error(e);
}
```

下面是一个例子，判断子节点的个数是否大于等于500。

```javascript
console.assert(list.childNodes.length < 500, '节点个数大于等于500')
```

上面代码中，如果符合条件的节点小于500个，不会有任何输出；只有大于等于500时，才会在控制台提示错误，并且显示指定文本。

### console.time()，console.timeEnd()

这两个方法用于计时，可以算出一个操作所花费的准确时间。

```javascript
console.time('Array initialize');

var array= new Array(1000000);
for (var i = array.length - 1; i >= 0; i--) {
  array[i] = new Object();
};

console.timeEnd('Array initialize');
// Array initialize: 1914.481ms
```

`time`方法表示计时开始，`timeEnd`方法表示计时结束。它们的参数是计时器的名称。调用`timeEnd`方法之后，控制台会显示“计时器名称: 所耗费的时间”。

### console.group()，console.groupEnd()，console.groupCollapsed()

`console.group`和`console.groupEnd`这两个方法用于将显示的信息分组。它只在输出大量信息时有用，分在一组的信息，可以用鼠标折叠/展开。

```javascript
console.group('一级分组');
console.log('一级分组的内容');

console.group('二级分组');
console.log('二级分组的内容');

console.groupEnd(); // 二级分组结束
console.groupEnd(); // 一级分组结束
```

上面代码会将“二级分组”显示在“一级分组”内部，并且“一级分组”和“二级分组”前面都有一个折叠符号，可以用来折叠本级的内容。

`console.groupCollapsed`方法与`console.group`方法很类似，唯一的区别是该组的内容，在第一次显示时是收起的（collapsed），而不是展开的。

```javascript
console.groupCollapsed('Fetching Data');

console.log('Request Sent');
console.error('Error: Server not responding (500)');

console.groupEnd();
```

上面代码只显示一行”Fetching Data“，点击后才会展开，显示其中包含的两行。

### console.trace()，console.clear()

`console.trace`方法显示当前执行的代码在堆栈中的调用路径。

```javascript
console.trace()
// console.trace()
//   (anonymous function)
//   InjectedScript._evaluateOn
//   InjectedScript._evaluateAndWrap
//   InjectedScript.evaluate
```

`console.clear`方法用于清除当前控制台的所有输出，将光标回置到第一行。如果用户选中了控制台的“Preserve log”选项，`console.clear`方法将不起作用。

## 控制台命令行 API

浏览器控制台中，除了使用`console`对象，还可以使用一些控制台自带的命令行方法。

（1）`$_`

`$_`属性返回上一个表达式的值。

```javascript
2 + 2
// 4
$_
// 4
```

（2）`$0` - `$4`

控制台保存了最近5个在 Elements 面板选中的 DOM 元素，`$0`代表倒数第一个（最近一个），`$1`代表倒数第二个，以此类推直到`$4`。

（3）`$(selector)`

`$(selector)`返回第一个匹配的元素，等同于`document.querySelector()`。注意，如果页面脚本对`$`有定义，则会覆盖原始的定义。比如，页面里面有 jQuery，控制台执行`$(selector)`就会采用 jQuery 的实现，返回一个数组。

（4）`$$(selector)`

`$$(selector)`返回选中的 DOM 对象，等同于`document.querySelectorAll`。

（5）`$x(path)`

`$x(path)`方法返回一个数组，包含匹配特定 XPath 表达式的所有 DOM 元素。

```javascript
$x("//p[a]")
```

上面代码返回所有包含`a`元素的`p`元素。

（6）`inspect(object)`

`inspect(object)`方法打开相关面板，并选中相应的元素，显示它的细节。DOM 元素在`Elements`面板中显示，比如`inspect(document)`会在 Elements 面板显示`document`元素。JavaScript 对象在控制台面板`Profiles`面板中显示，比如`inspect(window)`。

（7）`getEventListeners(object)`

`getEventListeners(object)`方法返回一个对象，该对象的成员为`object`登记了回调函数的各种事件（比如`click`或`keydown`），每个事件对应一个数组，数组的成员为该事件的回调函数。

（8）`keys(object)`，`values(object)`

`keys(object)`方法返回一个数组，包含`object`的所有键名。

`values(object)`方法返回一个数组，包含`object`的所有键值。

```javascript
var o = {'p1': 'a', 'p2': 'b'};

keys(o)
// ["p1", "p2"]
values(o)
// ["a", "b"]
```

（9）`monitorEvents(object[, events]) ，unmonitorEvents(object[, events])`

`monitorEvents(object[, events])`方法监听特定对象上发生的特定事件。事件发生时，会返回一个`Event`对象，包含该事件的相关信息。`unmonitorEvents`方法用于停止监听。

```javascript
monitorEvents(window, "resize");
monitorEvents(window, ["resize", "scroll"])
```

上面代码分别表示单个事件和多个事件的监听方法。

```javascript
monitorEvents($0, 'mouse');
unmonitorEvents($0, 'mousemove');
```

上面代码表示如何停止监听。

`monitorEvents`允许监听同一大类的事件。所有事件可以分成四个大类。

- mouse："mousedown", "mouseup", "click", "dblclick", "mousemove", "mouseover", "mouseout", "mousewheel"
- key："keydown", "keyup", "keypress", "textInput"
- touch："touchstart", "touchmove", "touchend", "touchcancel"
- control："resize", "scroll", "zoom", "focus", "blur", "select", "change", "submit", "reset"

```javascript
monitorEvents($("#msg"), "key");
```

上面代码表示监听所有`key`大类的事件。

（10）其他方法

命令行 API 还提供以下方法。

- `clear()`：清除控制台的历史。
- `copy(object)`：复制特定 DOM 元素到剪贴板。
- `dir(object)`：显示特定对象的所有属性，是`console.dir`方法的别名。
- `dirxml(object)`：显示特定对象的 XML 形式，是`console.dirxml`方法的别名。

## debugger 语句

`debugger`语句主要用于除错，作用是设置断点。如果有正在运行的除错工具，程序运行到`debugger`语句时会自动停下。如果没有除错工具，`debugger`语句不会产生任何结果，JavaScript 引擎自动跳过这一句。

Chrome 浏览器中，当代码运行到`debugger`语句时，就会暂停运行，自动打开脚本源码界面。

```javascript
for(var i = 0; i < 5; i++){
  console.log(i);
  if (i === 2) debugger;
}
```

上面代码打印出0，1，2以后，就会暂停，自动打开源码界面，等待进一步处理。

## 参考链接

- Chrome Developer Tools, [Using the Console](https://developers.google.com/chrome-developer-tools/docs/console)
- Matt West, [Mastering The Developer Tools Console](http://blog.teamtreehouse.com/mastering-developer-tools-console)
- Firebug Wiki, [Console API](https://getfirebug.com/wiki/index.php/Console_API)
- Axel Rauschmayer, [The JavaScript console API](http://www.2ality.com/2013/10/console-api.html)
- Marius Schulz, [Advanced JavaScript Debugging with console.table()](http://blog.mariusschulz.com/2013/11/13/advanced-javascript-debugging-with-consoletable)
- Google Developer, [Command Line API Reference](https://developers.google.com/chrome-developer-tools/docs/commandline-api)