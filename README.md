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

## 字符串的扩展
ES5中只限于\u0000~\uFFFF之间的字符。超过这个范围的字符，必须用两个双字节表示。

如果直接在\u后面跟上超过0xFFFF的数值（比如\u20BB7），JavaScript会理解成\u20BB+7。由于\u20BB是一个不可打印字符，所以只会显示一个空格，后面跟着一个7。

ES6对这一点进行了改进，只要将码点放入大括号，就能正确解读字符。

``` javascript
"\u{20BB7}"
// "𠮷"

"\u{41}\u{42}\u{43}"
// "ABC"

let hello = 123;
hell\u{6F} // 123

'\u{1F680}' === '\uD83D\uDE80'
// true
```

同时ES6给出了一系列方法来处理超过0xFFFF的字符，解决ES5的遗留问题。

### codePointAt()方法
ES5没办法处理需要4个字节存储的字符，charAt方法无法读取整个字符，charCodeAt方法只能分别返回前两个字节和后两个字节的值，ES6提供了codePointAt方法，能够正确处理4个字节储存的字符，返回一个字符的码点。codePointAt方法会正确返回32位的UTF-16字符的码点。对于那些两个字节储存的常规字符，它的返回结果与charCodeAt方法相同。

codePointAt方法返回的是码点的十进制值，如果想要十六进制的值，可以使用toString方法转换一下。

``` javascript
var s = '𠮷a';

s.codePointAt(0).toString(16) // "20bb7"
s.codePointAt(2).toString(16) // "61"
```
同时，该方法还是测试一个字符由两个字节还是四个字节组成的简单方法。

``` javascript
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

is32Bit("𠮷") // true
is32Bit("a") // false
```

### String.fromCodePoint()
跟codePointAt()相似，这个函数也是用来解决ES5中无法处理大于FFFF的字符的。这个方法可以直接从码点返回字符，在ES5中会将高位抛弃从而得不到结果。

**注意，fromCodePoint方法定义在String对象上，而codePointAt方法定义在字符串的实例对象上。**

### at()
at()方法同样是用来解决超过0xFFFF的字符问题的。ES5中的charAt方法在对付双字节存储的字符时只会得到一部分。

ES6中的一个提案是用一个at函数去返回正确的字符。

### normalize()
normalize()方法同样是用来处理疑难字符的，比如欧洲的一些带有音调的字符。为了表示它们，Unicode 提供了两种方法。一种是直接提供带重音符号的字符，比如Ǒ（\u01D1）。另一种是提供合成符号（combining character），即原字符与重音符号的合成，两个字符合成一个字符，比如O（\u004F）和ˇ（\u030C）合成Ǒ（\u004F\u030C）。

这两种表示方法，在视觉和语义上都等价，但是 JavaScript 不能识别。

``` javascript
'\u01D1'==='\u004F\u030C' //false

'\u01D1'.length // 1
'\u004F\u030C'.length // 2
```
上面代码表示，JavaScript 将合成字符视为两个字符，导致两种表示方法不相等。

ES6 提供字符串实例的normalize()方法，用来将字符的不同表示方法统一为同样的形式，这称为 Unicode 正规化。

``` javascript
'\u01D1'.normalize() === '\u004F\u030C'.normalize()
// true
```

**不过，normalize方法目前不能识别三个或三个以上字符的合成。这种情况下，还是只能使用正则表达式，通过Unicode编号区间判断。**

### includes(), startsWith(), endsWith()
原来的ES5中提供了一个indexOf方法，可以用来确定一个字符是否包含在另一个字符串中，ES6中又加了3个好用的方法。

- includes(): 返回布尔值，表示是否找到了参数字符串
- startsWith(): 返回布尔值，表示参数字符串是否在原字符串的头部
- endsWith(): 返回布尔值，表示参数字符串是否在元字符串的尾部

这三个方法都支持第二个参数，表示开始搜索的位置。

``` javascript
var s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true

s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```

上面代码表示，使用第二个参数n时，endsWith的行为与其他两个方法有所不同。它针对前n个字符，而其他两个方法针对从第n个位置直到字符串结束。

### repeat()
这个方法就是单纯地把某个字符串重复几遍，没啥好说的。上代码。

``` javascript
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""
```

### padStart(), padEnd()
如果某个字符串不够指定长度，可以用padStart(), padEnd()方法在头部或尾部补全。看一下代码也很快就能看懂。如果你省略第二个参数，默认会用空格补全。

``` javascript
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'

'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'
```

### 模板字符串
在ES6可以使用${}来编写模板输出字符串，这跟很多模板语言很像。然后引入了``来标识这种增强版的字符串。也可以用来定义多行字符串，或者在字符中嵌入变量。
代码的话，大概会这样：

``` javascript
$('#result').append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`);

// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```
**如果在模板字符串中需要使用反引号，则前面要用反斜杠转义。**

如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。

``` javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`);
```

在大括号内可以使用表达式，甚至可以调方法，还可以嵌套，很是酷炫。

``` javascript
var x = 1;
var y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

var obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"

function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar

const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;
```

如果需要引用模板字符串本身，在需要时执行，可以像下面这样写。

``` javascript
// 写法一
let str = 'return ' + '`Hello ${name}!`';
let func = new Function('name', str);
func('Jack') // "Hello Jack!"

// 写法二
let str = '(name) => `Hello ${name}!`';
let func = eval.call(null, str);
func('Jack') // "Hello Jack!"
```

### 标签模板
模板字符串可以直接跟在某个函数名后面，然后该函数可以用来处理这个模板字符串。如果模板字符里面有变量，就不是简单的调用，而是会把模板字符串先处理成多个参数，然后再调用函数。

``` javascript
var a = 5;
var b = 10;

tag`Hello ${ a + b } world ${ a * b }`;
// 等同于
tag(['Hello ', ' world ', ''], 15, 50);
```

函数调用的参数如下：

- 第一个参数是一个数组，该数组的成员是模板字符串中那些没有变量替换的部分，也就是说，变量替换只发生在数组的第一个成员与第二个成员之间、第二个成员与第三个成员之间，以此类推
- tag函数的其他参数，都是模板字符串各个变量被替换后的值。由于本例中，模板字符串含有两个变量，因此tag会接受到value1和value2两个参数

另外一个例子：

``` javascript
var a = 5;
var b = 10;

function tag(s, v1, v2) {
  console.log(s[0]);
  console.log(s[1]);
  console.log(s[2]);
  console.log(v1);
  console.log(v2);

  return "OK";
}

tag`Hello ${ a + b } world ${ a * b}`;
// "Hello "
// " world "
// ""
// 15
// 50
// "OK"
```

标签模板有主要这么几种功能：

- 过滤HTML字符串
- 多语言转换（国际化处理）
- 在js中嵌入其他语言

### String.raw()
String.raw()方法，往往用来充当模板字符串的处理函数，返回一个斜杠都被转义的字符串，对应于替换变量后的模板字符串。

``` javascript
String.raw`Hi\n${2+3}!`;
// "Hi\\n5!"

String.raw`Hi\u000A!`;
// 'Hi\\u000A!'
```

### 模板字符串的限制
当然啦，模板字符串也有它的局限性，归纳如下

- 虽然可以用模板字符串嵌入其他语言，但是由于存在字符串的转义，导致有些语言无法使用，比如嵌入Latex

## 正则的拓展
在ES5中，我们声明正则表达式一般有两种方法

- 参数是字符串，这时第二个参数表示正则表达式的修饰符
- 参数是一个正则表达式，这时会返回一个原有正则表达式的拷贝

但是，不允许这种情况

``` javascript
var regex = new RegExp(/xyz/, 'i');
```

ES6改变了这个状况，如果RegExp构造函数第一个参数是一个正则对象，那么可以使用第二个参数指定修饰符。而且，返回的正则表达式会忽略原有的正则表达式的修饰符，只使用新指定的修饰符。

``` javascript
new RegExp(/abc/ig, 'i').flags
// "i"
// 原有正则对象的修饰符是ig，它会被第二个参数i覆盖
```

### 字符串的正则方法
字符串自身就可以调用正则方法，比如match(), replace(), search(), split()。

ES6干脆把这几个方法都封装在了RexExp上。

- String.prototype.match 
- String.prototype.replace
- String.prototype.search 
- String.prototype.split 

### u
ES6中添加了u修饰符，来处理大于\uFFFF的unicode字符。

``` javascript
/^\uD83D/u.test('\uD83D\uDC2A') // false
/^\uD83D/.test('\uD83D\uDC2A') // true
```

上面代码中，\uD83D\uDC2A是一个四个字节的 UTF-16 编码，代表一个字符。但是，ES5 不支持四个字节的 UTF-16 编码，会将其识别为两个字符，导致第二行代码结果为true。加了u修饰符以后，ES6 就会识别其为一个字符，所以第一行代码结果为false。

u修饰符会影响以下行为：

#### 点字符
``` javascript
var s = '𠮷';

/^.$/.test(s) // false
/^.$/u.test(s) // true
```

#### unicode字符表示法
``` javascript
/\u{61}/.test('a') // false
/\u{61}/u.test('a') // true
/\u{20BB7}/u.test('𠮷') // true
```

#### 量词
``` javascript
/a{2}/.test('aa') // true
/a{2}/u.test('aa') // true
/𠮷{2}/.test('𠮷𠮷') // false
/𠮷{2}/u.test('𠮷𠮷') // true
```

#### 预定义模式
``` javascript
/^\S$/.test('𠮷') // false
/^\S$/u.test('𠮷') // true
```

#### i修饰符
``` javascript
//有些 Unicode 字符的编码不同，但是字型很相近，比如，\u004B与\u212A都是大写的K。

/[a-z]/i.test('\u212A') // false
/[a-z]/iu.test('\u212A') // true
//上面代码中，不加u修饰符，就无法识别非规范的K字符。
```

### y修饰符
ES6添加了一个叫做“粘连”的y修饰符。y和g的作用其实是差不多的，但是g是在上次匹配成功的后面的串种只有有匹配的串就行了，但是y必须是从上次匹配成功的地方开始。

``` javascript
var s = 'aaa_aa_a';
var r1 = /a+/g;
var r2 = /a+/y;

r1.exec(s) // ["aaa"]
r2.exec(s) // ["aaa"]

r1.exec(s) // ["aa"]
r2.exec(s) // null
```

**同时，ES6新增了一个sticky属性看是不是设置了y，并设置了一个flags属性来返回修饰符**

### s修饰符：dotAll
ES6引入s修饰符使得.可以通配所有的字符。并且设置了一个dotAll属性看是不是使用了这个属性。

### 后行断言
ES本来只有先行断言和先行否定断言，但是ES6引入了后行断言和后行否定断言。

- 先行断言：x只有在y前面才匹配，必须写成/x(?=y)/。比如，只匹配百分号之前的数字，要写成/\d+(?=%)/
- 先行否定断言：x只有不在y前面才匹配，必须写成/x(?!y)/。比如，只匹配不在百分号之前的数字，要写成/\d+(?!%)/
- 后行断言：x只有在y后面才匹配，必须写成/(?<=y)x/。比如，只匹配美元符号之后的数字，要写成/(?<=\$)\d+/
- 后行否定断言：x只有不在y后面才匹配，必须写成/(?<!y)x/。比如，只匹配不在美元符号后面的数字，要写成/(?<!\$)\d+/

### Unicode属性类
这个新加的特性非常猛，可以通过\p来匹配unicode属性相关的东西。比如：

``` javascript
const regexGreekSymbol = /\p{Script=Greek}/u;
regexGreekSymbol.test('π') // true
```
上面代码匹配了一个希腊字母。

规则：
> \p{UnicodePropertyName=UnicodePropertyValue}

有些属性可以致谢属性名。

**注意，这两种类只对 Unicode 有效，所以使用的时候一定要加上u修饰符。如果不加u修饰符，正则表达式使用\p和\P会报错，ECMAScript 预留了这两个类。**

功能非常强：

``` javascript
const regex = /^\p{Decimal_Number}+$/u;
regex.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼') // true

// 匹配所有数字
const regex = /^\p{Number}+$/u;
regex.test('²³¹¼½¾') // true
regex.test('㉛㉜㉝') // true
regex.test('ⅠⅡⅢⅣⅤⅥⅦⅧⅨⅩⅪⅫ') // true

// 匹配各种文字的所有字母，等同于 Unicode 版的 \w
[\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]

// 匹配各种文字的所有非字母的字符，等同于 Unicode 版的 \W
[^\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]

// 匹配所有的箭头字符
const regexArrows = /^\p{Block=Arrows}+$/u;
regexArrows.test('←↑→↓↔↕↖↗↘↙⇏⇐⇑⇒⇓⇔⇕⇖⇗⇘⇙⇧⇩') // true
```

### 具名组匹配
原本的组匹配就是加一个圆括号，然后就可以用序号访问到。

``` javascript
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj[1]; // 1999
const month = matchObj[2]; // 12
const day = matchObj[3]; // 31
```

现在引入了具体的名称，便于引用。

``` javascript
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj.groups.year; // 1999
const month = matchObj.groups.month; // 12
const day = matchObj.groups.day; // 31
```

在圆括号内部，模式的头部添加“问号 + 尖括号 + 组名”（?<year>），然后就可以在exec方法返回结果的groups属性上引用该组名。同时，数字序号（matchObj[1]）依然有效。

用法非常骚气：

``` javascript
// 可以直接赋值
let {groups: {one, two}} = /^(?<one>.*):(?<two>.*)$/u.exec('foo:bar');
one  // foo
two  // bar

// 直接replace
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;

'2015-01-02'.replace(re, '$<day>/$<month>/$<year>')
// '02/01/2015'

// 内部自己引用
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
RE_TWICE.test('abc!abc') // true
RE_TWICE.test('abc!ab') // false
```

## 数值的扩展
### 二进制和八进制表示法
ES6 提供了二进制和八进制数值的新的写法，分别用前缀0b（或0B）和0o（或0O）表示。

``` javascript
0b111110111 === 503 // true
0o767 === 503 // true
```

从 ES5 开始，在严格模式之中，八进制就不再允许使用前缀0表示，ES6 进一步明确，要使用前缀0o表示。

``` javascript
/ 非严格模式
(function(){
  console.log(0o11 === 011);
})() // true

// 严格模式
(function(){
  'use strict';
  console.log(0o11 === 011);
})() // Uncaught SyntaxError: Octal literals are not allowed in strict mode.
```

如果要将0b和0o前缀的字符串数值转为十进制，要使用Number方法。

``` javascript
Number('0b111')  // 7
Number('0o10')  // 8
```

### Number.isFinite(), Number.isNaN()
ES6 在Number对象上，新提供了Number.isFinite()和Number.isNaN()两个方法。
Number.isFinite()用来检查一个数值是否为有限的（finite）。
Number.isNaN()用来检查一个数值是否为NaN。

### Number.parseInt(), Number.parseFloat()
ES6将parseInt()和parseFloat()两个方法放进了Number里，行为保持不变。

### Number.isInteger()
ES6新加了isInteger()方法来判断一个数是不是整数，在js中，整数和浮点数是一样的存储方式。所以1和1.0会得到一样的结果。

### Number.EPSILON
这个东西纯粹是为了方便。我们都知道浮点数的比较是比较麻烦的，浮点数的计算也是不精确的，这个常量的添加就是为了来进行误差检查的。

``` javascript
Number.EPSILON
// 2.220446049250313e-16
Number.EPSILON.toFixed(20)
// '0.00000000000000022204'
```

如果说计算的误差小于这个值，我们就可以说我们的计算是精确的

### 安全整数和Number.isSafeIntger()
js能够精确表示的数值在负2的53次和2的53次之间，超过了就不精确了。ES6引入Number.MAX_SAFE_INTEGER和Number.MIN_SAFE_INTEGER这两个常量，用来表示这个范围的上下限。Number.isSafeInteger()则是用来判断一个整数是否落在这个范围之内。

``` javascript
Number.isSafeInteger('a') // false
Number.isSafeInteger(null) // false
Number.isSafeInteger(NaN) // false
Number.isSafeInteger(Infinity) // false
Number.isSafeInteger(-Infinity) // false

Number.isSafeInteger(3) // true
Number.isSafeInteger(1.2) // false
Number.isSafeInteger(9007199254740990) // true
Number.isSafeInteger(9007199254740992) // false

Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1) // false
Number.isSafeInteger(Number.MIN_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1) // false
```

### Math对象的拓展
ES6在Math对象上新增了17个数学方法。

#### Math.trunc()
Math.trunc()方法用来去除小数部分，对于非数值，会先转换为数值，对于空值和无法截取整数的值，会返回NaN。

``` javascript
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4
Math.trunc(-0.1234) // -0
```

#### Math.sign()
Math.sign()方法用来判断一个数是整数、负数、0。有5种情况：

- 参数为正数，返回+1
- 参数为负数，返回-1
- 参数为0，返回0
- 参数为-0，返回-0
- 其他值，返回NaN

#### Math.signbit()
Math.signbit()用来解决无法判断正负零的问题。

#### Math.cbrt()
Math.cbrt()方法用来计算一个数的立方根，会先转换为数值。判断一个数的符号位是否设置了。

``` javascript
Math.cbrt(-1) // -1
Math.cbrt(0)  // 0
Math.cbrt(1)  // 1
Math.cbrt(2)  // 1.2599210498948734
```

#### Math.clz32()
Math.clz32()方法返回一个数的32位无符号整数形式有多少个前导。对于小数，只考虑整数部分，对于空值或者其他类型的值，会先转化，再计算。

``` javascript
Math.clz32(0) // 32
Math.clz32(1) // 31
Math.clz32(1000) // 22
```

#### Math.imul()
Math.imul()方法返回两个数以32位带符号整数形式相乘的结果，返回的也是一个32位的带符号整数。这个方法感觉用处不是很大，就是一个核心问题，超过了2的53次js无法保证精度，低位会有问题，在做溢出的计算时，这个方法可以保证低位的精度。一般来说很少会有这种情况，我感觉用处不大。

``` javascript
(0x7fffffff * 0x7fffffff)|0 // 0
Math.imul(0x7fffffff, 0x7fffffff) // 1
```

#### Math.fround()
Math.fround()方法返回一个数的单精度浮点数形式，用来处理没法用64位二进制精确表示的小数。

``` javascript
Math.fround(0)     // 0
Math.fround(1)     // 1
Math.fround(1.337) // 1.3370000123977661
Math.fround(1.5)   // 1.5
Math.fround(NaN)   // NaN
```

#### Math.hypot()
Math.hypot()方法返回所有参数的平方和的平方根。

``` javascript
Math.hypot(3, 4);        // 5
Math.hypot(3, 4, 5);     // 7.0710678118654755
Math.hypot();            // 0
Math.hypot(NaN);         // NaN
```

#### Math.expm1(), Math.log1p(), Math.log10(), Math.log2()
这四个方法都是针对对数的方法。

- Math.expm1(x)会返回e^x -1
- Math.log1p(x)会返回1 + x的自然对数
- Math.log10(x)会返回以10为底的x的对数。如果x小于0，则返回NaN
- Math.log2(x)会返回以2为底的x的对数。如果x小于0，则返回NaN

#### 双曲线方法

- Math.sinh(x) 返回x的双曲正弦（hyperbolic sine）
- Math.cosh(x) 返回x的双曲余弦（hyperbolic cosine）
- Math.tanh(x) 返回x的双曲正切（hyperbolic tangent）
- Math.asinh(x) 返回x的反双曲正弦（inverse hyperbolic sine）
- Math.acosh(x) 返回x的反双曲余弦（inverse hyperbolic cosine）
- Math.atanh(x) 返回x的反双曲正切（inverse hyperbolic tangent）

### 指数运算符
ES6新增了指数运算符，它可以和=结合，形成新的赋值运算符 \**=。

``` javascript
2 ** 2 // 4
2 ** 3 // 8
let a = 1.5;
a **= 2;
// 等同于 a = a * a;

let b = 4;
b **= 3;
// 等同于 b = b * b * b;
```

**这个运算符的实现和Math.pow()是不同的，在对特别大的数据进行计算时，会有一些差异。**

### Integer数据类型
这个特性完全是为了迎合时代潮流了。因为js的所有数字都保存成64位浮点数，所以它的最高精度只能到53个二进制位，没法做科学计算。现在就出来一个Integer，只用来表示整数，无位数的限制。

为了区分，必须使用n后缀。

``` javascript
1n + 2n //3n
0b1101n // 二进制
0o777n // 八进制
0xFFn // 十六进制

typeof 123n
// 'integer'

Integer(123) // 123n
Integer('123') // 123n
Integer(false) // 0n
Integer(true) // 1n
```

在数学运算方面，Integer 类型的+、-、\*和\**这四个二元运算符，与 Number 类型的行为一致。但是有两个除外：不带符号的右移位运算符>>>和一元的求正运算符+，使用时会报错。前者是因为>>>要求最高位补0，但是 Integer 类型没有最高位，导致这个运算符无意义。后者是因为一元运算符+在 asm.js 里面总是返回 Number 类型或者报错。

**Integer 类型不能与 Number 类型进行混合运算。**
**相等运算符（==）会改变数据类型，也是不允许混合使用。**

## 函数的拓展
### 函数默认值
到了ES6，函数终于有默认值了，在ES5中其实是可以用一些变通的方法来完成参数默认的，但是有一些缺点，比如说如果参数赋值了，但是对应的布尔值为false，则该赋值不起作用。ES6就解决了这个问题。

``` javascript
// ES5
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello World 出现问题

// ES6
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello 没毛病
```

**参数变量是默认声明的，所以不能用let或者const再次声明**

``` javascript
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}
```
**使用参数默认值时，函数不能有同名参数**

``` javascript
// 不报错
function foo(x, x, y) {
  // ...
}

// 报错
function foo(x, x, y = 1) {
  // ...
}
// SyntaxError: Duplicate parameter name not allowed in this context
```

**参数默认值不是传值的，而是每次都重新计算默认值表达式的值。也就是说，参数默认值是惰性求值的**

``` javascript
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}

foo() // 100

x = 100;
foo() // 101
```
每次调用都重新计算

#### 与解构赋值默认值结合使用
参数默认值可以和解构赋值的默认值，结合起来使用。
更骚气的是如果参数时一个对象，对象里面做了默认值，比如{x, y=5}，这个时候如果啥也不传就会报错，这个时候可以通过提供函数参数的默认值，避免报错。

``` javascript
function foo({x, y = 5} = {}) {
  console.log(x, y);
}

foo() // undefined 5

function fetch(url, { body = '', method = 'GET', headers = {} }) {
  console.log(method);
}

fetch('http://example.com', {})
// "GET"

fetch('http://example.com')

function fetch(url, { method = 'GET' } = {}) {
  console.log(method);
}

fetch('http://example.com')
// "GET"
```
出现了双重默认值。

#### 参数默认值的位置
通常情况下，定义了默认值的参数，应该是函数的尾参数。因为这样比较容易看出来，到底省略了哪些参数。如果非尾部的参数设置默认值，实际上这个参数是没法省略的。

``` javascript
// 例一
function f(x = 1, y) {
  return [x, y];
}

f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 报错
f(undefined, 1) // [1, 1]

// 例二
function f(x, y = 5, z) {
  return [x, y, z];
}

f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // 报错
f(1, undefined, 2) // [1, 5, 2]
```

上述情况都不能省略参数，只能设置undefined来触发等于默认值。但是null没有这个效果。

#### 函数的长度（length）
函数的属性length返回函数没有指定默认值的参数的个数，如果指定了默认参数，那么将不计入length，length永远返回没有默认参数的传入参数个数。

``` javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
```
有趣的是这个length的计数是根据默认参数的位置来计算的，果设置了默认值的参数不是尾参数，那么length属性也不再计入后面的参数了。

``` javascript
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

#### 作用域
一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域。等到初始化结束，这个作用域也就消失了。

``` javascript
var x = 1;

function f(x, y = x) {
  console.log(y);
}

f(2) // 2

let x = 1;

function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // 1
```
上面两种情况，第一种就是指向第一个x，外面全局的x不会影响y的赋值。但是第二种情况因为x本身没有定义，所以会指向外部全局的x，但是在内部x的操作，并不会影响到y的值。

#### 应用
利用参数默认值，可以指定某一个参数不得省略，如果省略就抛出错误。

``` javascript
function throwIfMissing() {
  throw new Error('Missing parameter');
}

function foo(mustBeProvided = throwIfMissing()) {
  return mustBeProvided;
}

foo()
// Error: Missing parameter
```

### rest参数
ES5中函数传参可以用arguments，在ES6加入了rest参数来使得传参更加地优雅，用法为...变量名。

``` javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10
```

**注意，rest参数必须是最后一个参数，否则会报错**

### 严格模式
ES5开始函数内部可以设定为严格模式，但是ES6引入默认值、解构赋值、扩展运算符之后，函数内部就不可以使用严格模式了，否则就会报错。

因为函数内部的严格模式，同样适用于函数体和函数参数，但是函数执行的时候，先执行函数参数，然后执行函数体，这就不合理了，只有从函数体之中，才能知道参数是否应该以严格模式执行，但是参数应该先初始化。

当然了，你可以吧“use strict”放在外面或者用一个立即执行的函数包住函数。

``` javascript
'use strict';

function doSomething(a, b = a) {
  // code
}

const doSomething = (function () {
  'use strict';
  return function(value = 42) {
    return value;
  };
}());
```

### name属性
函数的name属性，直接返回函数的名字，有很多种情况，比如匿名函数还有bind的函数等，详见代码。

``` javascript
var f = function () {};

// ES5
f.name // ""

// ES6
f.name // "f"

const bar = function baz() {};

// ES5
bar.name // "baz"

// ES6
bar.name // "baz"

(new Function).name // "anonymous"

function foo() {};
foo.bind({}).name // "bound foo"

(function(){}).bind({}).name // "bound "
```

### 箭头函数
ES6的箭头表达式是非常有用的一个东西。哇，真的方便。

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

``` javascript
var f = () => 5;
// 等同于
var f = function () { return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};
```

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回。

var sum = (num1, num2) => { return num1 + num2; }
由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错。

``` javascript
// 报错
let getTempItem = id => { id: id, name: "Temp" };

// 不报错
let getTempItem = id => ({ id: id, name: "Temp" });
```

如果箭头函数只有一行语句，且不需要返回值，可以采用下面的写法，就不用写大括号了。

``` javascript
let fn = () => void doesNotReturn();
```

箭头函数可以与变量解构结合使用。

``` javascript
const full = ({ first, last }) => first + ' ' + last;

// 等同于
function full(person) {
  return person.first + ' ' + person.last;
}
```

箭头函数使得表达更加简洁。
箭头函数的一个用处是简化回调函数。

``` javascript
// 正常函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);
另一个例子是

// 正常函数写法
var result = values.sort(function (a, b) {
  return a - b;
});

// 箭头函数写法
var result = values.sort((a, b) => a - b);
```

下面是 rest 参数与箭头函数结合的例子。

``` javascript
const numbers = (...nums) => nums;

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]
```

#### 注意点

- 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象
- 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误
- 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替
- 不可以使用yield命令，因此箭头函数不能用作 Generator 函数

**第一点尤其值得注意。this对象的指向是可变的，但是在箭头函数中，它是固定的**

#### 绑定this
箭头函数可以绑定this对象，这就可以减少bind显式的调用。

函数绑定运算符是并排的两个冒号（::），双冒号左边是一个对象，右边是一个函数。该运算符会自动将左边的对象，作为上下文环境（即this对象），绑定到右边的函数上面。

如果双冒号左边为空，右边是一个对象的方法，则等于将该方法绑定在该对象上面。

由于双冒号运算符返回的还是原对象，因此可以采用链式写法。

``` javascript
foo::bar;
// 等同于
bar.bind(foo);

foo::bar(...arguments);
// 等同于
bar.apply(foo, arguments);

const hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return obj::hasOwnProperty(key);
}

var method = obj::obj.foo;
// 等同于
var method = ::obj.foo;

let log = ::console.log;
// 等同于
var log = console.log.bind(console);

// 例一
import { map, takeWhile, forEach } from "iterlib";

getPlayers()
::map(x => x.character())
::takeWhile(x => x.strength > 100)
::forEach(x => console.log(x));

// 例二
let { find, html } = jake;

document.querySelectorAll("div.myClass")
::find("p")
::html("hahaha");
```

#### 尾逗号
ES6支持参数的最后一个后面可以接一个逗号

``` javascript
function clownsEverywhere(
  param1,
  param2,
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar
```



