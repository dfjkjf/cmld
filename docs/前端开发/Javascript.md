---
sort: 2
---

# Javascript

- 注释

```js
// 注释方式和C很像，这是单行注释
/* 这是多行
   注释 */
```

- 数字

```js
// Javascript 只有一种数字类型(即 64位 IEEE 754 双精度浮点 double)。
// double 有 52 位表示尾数，足以精确存储大到 9✕10¹⁵ 的整数。
3; // = 3
1.5; // = 1.5

// 有三种非数字的数字类型
Infinity; // 1/0 的结果
-Infinity; // -1/0 的结果
NaN; // 0/0 的结果

// 也有布尔值。
true;
false;
```

- 所有基本的算数运算都如你预期。

```js
1 + 1; // = 2
0.1 + 0.2; // = 0.30000000000000004
8 - 1; // = 7
10 * 2; // = 20
35 / 5; // = 7

// 包括无法整除的除法。
5 / 2; // = 2.5

// 位运算也和其他语言一样；当你对浮点数进行位运算时，
// 浮点数会转换为*至多* 32 位的无符号整数。
1 << 2; // = 4

// 括号可以决定优先级。
(1 + 3) * 2; // = 8

// 用！来取非
!true; // = false
!false; // = true

// 相等 ===
1 === 1; // = true
2 === 1; // = false

// 不等 !=
1 !== 1; // = false
2 !== 1; // = true

// 更多的比较操作符 
1 < 10; // = true
1 > 10; // = false
2 <= 2; // = true
2 >= 2; // = true

// 字符串用+连接
"Hello " + "world!"; // = "Hello world!"

// 字符串也可以用 < 、> 来比较
"a" < "b"; // = true

// 使用“==”比较时会进行类型转换...
"5" == 5; // = true
null == undefined; // = true

// ...除非你是用 ===
"5" === 5; // = false
null === undefined; // = false 

// ...但会导致奇怪的行为
13 + !0; // 14
"13" + !0; // '13true'
```

- 字符串

```js
// 可以通过单引号或双引号来构造字符串。
'abc';
"Hello, world";

// 你可以用`charAt`来得到字符串中的字符
"This is a string".charAt(0);  // = 'T'

// ...或使用 `substring` 来获取更大的部分。
"Hello world".substring(0, 5); // = "Hello"

// `length` 是一个属性，所以不要使用 ().
"Hello".length; // = 5
```

- 还有两个特殊的值：`null`和`undefined`

```js
null;      // 用来表示刻意设置的空值
undefined; // 用来表示还没有设置的值(尽管`undefined`自身实际是一个值)

// false, null, undefined, NaN, 0 和 "" 都是假的；其他的都视作逻辑真
// 注意 0 是逻辑假而  "0"是逻辑真，尽管 0 == "0"。
```
- 变量

```js
// 变量需要用`var`关键字声明。Javascript是动态类型语言，
// 所以你无需指定类型。 赋值需要用 `=` 
var someVar = 5;

// 如果你在声明时没有加var关键字，你也不会得到错误，但是此时这个变量就会在全局作用域被创建，而非你定义的当前作用域
someOtherVar = 10;

// 没有被赋值的变量都会被设置为undefined
var someThirdVar; // = undefined

// 对变量进行数学运算有一些简写法：
someVar += 5; // 等价于 someVar = someVar + 5; someVar 现在是 10 
someVar *= 10; // 现在 someVar 是 100

// 自增和自减也有简写
someVar++; // someVar 是 101
someVar--; // 回到 100
```

- 数组

```js
// 数组是任意类型组成的有序列表
var myArray = ["Hello", 45, true];

// 数组的元素可以用方括号下标来访问。
// 数组的索引从0开始。
myArray[1]; // = 45

// 数组是可变的，并拥有变量 length。
myArray.push("World");
myArray.length; // = 4

// 在指定下标添加/修改
myArray[3] = "Hello";
```

- 对象

```js
// javascript中的对象相当于其他语言中的“字典”或“映射”：是键-值对的无序集合。
var myObj = {key1: "Hello", key2: "World"};

// 键是字符串，但如果键本身是合法的js标识符，则引号并非是必须的。
// 值可以是任意类型。
var myObj = {myKey: "myValue", "my other key": 4};

// 对象属性的访问可以通过下标
myObj["my other key"]; // = 4

// ... 或者也可以用 . ，如果属性是合法的标识符
myObj.myKey; // = "myValue"

// 对象是可变的；值也可以被更改或增加新的键
myObj.myThirdKey = true;

// 如果你想要获取一个还没有被定义的值，那么会返回undefined
myObj.myFourthKey; // = undefined
```

- 逻辑与控制结构

```js
// `if`语句和其他语言中一样。
var count = 1;
if (count == 3){
    // count 是 3 时执行
} else if (count == 4){
    // count 是 4 时执行
} else {
    // 其他情况下执行 
}

// while循环
while (true) {
    // 无限循环
}

// Do-while 和 While 循环很像 ，但前者会至少执行一次
var input;
do {
    input = getInput();
} while (!isValid(input))

// `for`循环和C、Java中的一样：
// 初始化; 继续执行的条件; 迭代。
for (var i = 0; i < 5; i++){
    // 遍历5次
}

// && 是逻辑与, || 是逻辑或
if (house.size == "big" && house.colour == "blue"){
    house.contains = "bear";
}
if (colour == "red" || colour == "blue"){
    // colour是red或者blue时执行
}

// && 和 || 是“短路”语句，它在设定初始化值时特别有用 
var name = otherName || "default";

// `switch`语句使用`===`检查相等性。
// 在每一个case结束时使用 'break'
// 否则其后的case语句也将被执行。 
grade = 'B';
switch (grade) {
  case 'A':
    console.log("Great job");
    break;
  case 'B':
    console.log("OK job");
    break;
  case 'C':
    console.log("You can do better");
    break;
  default:
    console.log("Oy vey");
    break;
}
```

- 函数

```js

```