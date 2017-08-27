# ES6学习笔记
## 什么是ES6
### ES6和ECMAScript 2015之间的关系
ES6 既是一个历史名词，也是一个泛指，含义是5.1版以后的 JavaScript 的下一代标准，涵盖了ES2015、ES2016、ES2017等等，而ES2015 则是正式名称，特指该年发布的正式版本的语言标准。本书中提到 ES6 的地方，一般是指 ES2015 标准，但有时也是泛指“下一代 JavaScript 语言”。
### 部署进度
Node是js的服务器运行环境。它对ES6的支持度更高。除了那些默认打开的功能，还有一些语法功能已经实现了，但是默认没有打开。
### Babel转码器
Babel是一个广泛的ES6转码器，可以将ES6代码转为ES5代码，从而实现在现有环境下运行。你可以放心大胆地用ES6写代码，然后不用担心环境是否支持。

``` javascript
// 转码前
input.map(item => item + 1);

// 转码后
input.map(function (item) {
  return item + 1;
});
```
#### .babelrc配置文件
一般你要使用babel进行转码，你必须在项目里配置一个.babelrc文件来规定一下转码的规则。官方提供的规则如下：

```
# 最新转码规则
$ npm install --save-dev babel-preset-latest

# react 转码规则
$ npm install --save-dev babel-preset-react

# 不同阶段语法提案的转码规则（共有4个阶段），选装一个
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3 
```
然后在，在配置文件里加入你要的规则

``` javascript
{
    "presets": [
     "latest",
     "react",
     "stage-2"
    ],
    "plugins": []
}
```
#### 在项目中配置babel-cli
将babel-cli安装在项目之中可以解决项目对环境有依赖的问题。
> npm install --save-dev babel-cli

然后，改写package.json

``` javascript
{
  // ...
  "devDependencies": {
    "babel-cli": "^6.0.0"
  },
  "scripts": {
    "build": "babel src -d lib"
  },
}
# 执行
npm run build
```
#### 使用babel-node替换node
babel-node是babel-cli自带的一个命令，提供支持ES6的REPL环境，可以直接运行ES6代码。
将它安装在项目中并替换node，script.js本身就不用做任何转码处理。

``` javascript
$ npm install --save-dev babel-cli
# package.json
{
  "scripts": {
    "script-name": "babel-node script.js"
  }
}
```
#### babel-register在require时进行转码
babel-register模块改写require命令，为它加上一个钩子。此后，每当使用require加载.js、.jsx、.es和.es6后缀名的文件，就会先用Babel进行转码。

``` javascript
$ npm install --save-dev babel-register
require("babel-register");
require("./index.js");
```
babel-register只会对require命令加载的文件转码，而不会对当前文件转码。另外，由于它是实时转码，所以只适合在开发环境使用。

#### babel和其他工具一起使用
许多工具都需要Babel进行前置转码，入ESLint和Mocha。
比如配置esLint，首先安装esLint。

```
npm install --save-dev eslint babel-eslint
# 新建.eslintrc
{
  "parser": "babel-eslint",
  "rules": {
    ...
  }
}
# 在package.json中，加入相应的scripts脚本
{
    "name": "my-module",
    "scripts": {
      "lint": "eslint my-files.js"
    },
    "devDependencies": {
      "babel-eslint": "...",
      "eslint": "..."
    }
  }
```

## let和const命令
### let
ES6新增了变量let来解决之前块级作用域的问题。let所声明的变量只会在代码块内有效。
可以看一下下面这个例子：

``` javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10

var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```
第一组代码中的var是全局的，所有的i都指向一个i，所以会是10，但是在第二组代码中。i由let声明，只在块级作用域里有效，所以输出了6。

> 对于for循环，有特别之处，设置循环的那部分是父作用域，而循环体内部是单独的子作用域。

``` javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```
由代码可知，两个i不一样

#### 不存在变量提升
使用var会导致变量提升，变量在声明前被使用，不会报错，只会输出undefined。但是使用let之后再声明之前使用变量会直接报错。

``` javascript
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

#### 暂时性死区（TDZ）
如果你在一个块级作用域中使用了let，它所声明的变量就在这个区域内不受外部的影响。

``` javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```
在该块级作用域中，tmp使用了let就被绑死了，在该区域未声明tmp之前使用tmp就会报错，而跟全局的tmp没有什么关系。

ES6明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

这个特性的出现，导致typeof操作不是百分之百安全的了。

``` javascript
# 现在
typeof x; // ReferenceError
let x;

# 之前
typeof x; // "undefined"
var x;
```
上面代码中，变量x使用let命令声明，所以在声明之前，都属于x的“死区”，只要用到该变量就会报错。因此，typeof运行时就会抛出一个ReferenceError。

作为比较，如果一个变量根本没有被声明，使用typeof反而不会报错。

ES6 规定暂时性死区和let、const语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在 ES5 是很常见的，现在有了这种规定，避免此类错误就很容易了。

总之，暂时性死区的本质就是，**只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量**。

#### 不允许重复声明
let不允许在相同作用域内，重复声明同一个变量。

### 块级作用域
let实际是位js新增了块级作用域。

- 允许块级作用域的任意嵌套
- 外层作用域无法读取内层作用域的变量
- 内层作用域可以定义外层作用域的同名变量
- 块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了

#### 块级作用域和函数声明
ES5中，以下情况是非法的：

``` javascript
// 情况一
if (true) {
  function f() {}
}

// 情况二
try {
  function f() {}
} catch(e) {
  // ...
}
```

但是，浏览器没有遵守这个规定，为了兼容以前的旧代码，还是支持在块级作用域之中声明函数，因此上面两种情况实际都能运行，不会报错。

ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。ES6 规定，块级作用域之中，函数声明语句的行为类似于let，在块级作用域之外不可引用。

考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。

``` javascript
// 函数声明语句
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```
另外，还有一个需要注意的地方。ES6 的块级作用域允许声明函数的规则，只在使用大括号的情况下成立，如果没有使用大括号，就会报错。

### const
const声明一个只读的常量。一旦声明，常量的值就不能改变了。const声明的变量不得改变值，这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值，如果你只声明而不赋值，就会发生错误。

const声明的常量也存在暂时性死区，只能在声明的位置后面使用；也不能重复声明。

#### 本质
const实际上保证的，并不是变量的值不得改动，而是指向的内存地址不能改动。这边就会涉及到说，把对象变成const的，那么只能保证指向对象的指针是不变的，至于它的数据结构是不是可变是不可以空数字的，所以要很小心。

``` javascript
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only

const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
```

如果你真的想把对象锁起来，可以使用Object.freeze方法。但是只在严格模式下会报错，常规模式下，只是不起作用。

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数：

``` javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```

### 顶层对象的属性
顶层对象的属性与全局变量挂钩，被认为是JavaScript语言最大的设计败笔之一。这样的设计带来了几个很大的问题，首先是没法在编译时就报出变量未声明的错误，只有运行时才能知道（因为全局变量可能是顶层对象的属性创造的，而属性的创造是动态的）；其次，程序员很容易不知不觉地就创建了全局变量（比如打字出错）；最后，顶层对象的属性是到处可以读写的，这非常不利于模块化编程。另一方面，window对象有实体含义，指的是浏览器的窗口对象，顶层对象是一个有实体含义的对象，也是不合适的。

ES6为了改变这一点，一方面规定，为了保持兼容性，var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，**let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性**。也就是说，从ES6开始，全局变量将逐步与顶层对象的属性脱钩。

## 变量的结构赋值
### 数组的解构赋值
ES6允许从数组中提取值，按照对应的位置，对变量赋值。

``` javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```
如果解构不成功，变量的值就等于undefined。

**事实上，只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值**。

可以指定默认值来进行解构。

``` javascript
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

默认值的判定使用的是严格的===，如果一个数组成员是null，就会失效，因为null不严格等于undefined。

如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。

### 对象的解构赋值
对象的解构赋值和数组的解构赋值是相似的。

对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。

``` javascript
let { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
```

**对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。**

``` javascript
var node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

var { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start // Object {line: 1, column: 5}
```

同时也支持默认值，也是null这边要特别注意。

如果要将一个已经声明的变量用于解构赋值，必须非常小心。

``` javascript
// 错误的写法
let x;
{x} = {x: 1};
// SyntaxError: syntax error

// 正确的写法
let x;
({x} = {x: 1});
```
上面代码的写法会报错，因为 JavaScript 引擎会将{x}理解成一个代码块，从而发生语法错误。只有不将大括号写在行首，避免 JavaScript 将其解释为代码块，才能解决这个问题。

### 字符串的解构赋值
字符串也可以结构赋值，因为字符串被转换成了一个类似数组的数据结构。

``` javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
let {length : len} = 'hello';
len // 5
```

### 数值和布尔值的解构赋值
解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。undefined和null没法转为对象，所以对它们进行解构，都会报错。

### 函数参数的解构赋值
函数的参数也可以进行解构赋值，并可以指定默认值，undefined就会触发默认值。

``` javascript
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]

[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```

### 圆括号问题
解构赋值虽然很方便，但是解析起来并不容易。对于编译器来说，一个式子到底是模式，还是表达式，没有办法从一开始就知道，必须解析到（或解析不到）等号才能知道。

由此带来的问题是，如果模式中出现圆括号怎么处理。ES6 的规则是，只要有可能导致解构的歧义，就不得使用圆括号。

但是，这条规则实际上不那么容易辨别，处理起来相当麻烦。因此，建议只要有可能，就不要在模式中放置圆括号。

#### 不能使用的情况

- 变量声明语句
- 函数参数
- 赋值语句的模式

#### 可以使用的情况
赋值语句的非模式部分

``` javascript
[(b)] = [3]; // 正确
({ p: (d) } = {}); // 正确
[(parseInt.prop)] = [3]; // 正确
```

### 解构赋值的用途

#### 交换变量的值

``` javascript
let x = 1;
let y = 2;
    
[x, y] = [y, x];
```
#### 从函数返回多个值

``` javascript
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```
#### 函数参数的定义

``` javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

#### 提取json

``` javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

#### 函数参数的默认值
``` javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
```
#### 遍历MAP结构
``` javascript
var map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world

// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}

```
#### 输入模块的指定方法

``` javascript
const { SourceMapConsumer, SourceNode } = require("source-map");
```

