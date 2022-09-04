---
layout: post
title:  "全栈开发从入门到放弃:1.b-JavaScript基础"
date:   2021-06-18 21:00:00
categories: FullStack
tags:   [React, 全栈开发, 学习笔记]
---

>本系列介绍：《全栈开发从入门到放弃》是本人学习如何使用JavaScript开发现代Web应用程序的过程记录。课程的重点是使用ReactJS构建单页面应用程序（SPA）。

>本系列记录了本人学习过程中的要点，不成体系。如果你想深入学习，建议寻找官方教程 😃

---

JavaScript标准的正式名称是`ECMAScript`。由于浏览器不能支持所有JavaScript的最新特性，所以我们需要转译到一个更旧更兼容的版本。使用`create-react-app`创建的应用已配置为使用`Babel`自动转译。

`Node.js`是基于谷歌的 chrome V8 引擎的`JavaScript`运行时环境。代码文件以 *.js*结尾，通过 `node filename.js` 命令运行文件。

## 变量(Variables)

在JavaScript中有以下几种定义变量的方法:

```javascript
const x = 1
let y = 5

console.log(x, y)
y += 10
console.log(x, y)
y = 'sometext'
console.log(x, y)
x = 4  // cause an error
```

`const` 实际是定义了一个常量， `let`定义了一个普通变量。推荐使用`let`而不是`var`。

## 数组(Arrays)

使用数组的示例如下：

```javascript
const t = [1, -1, 3]
t.push(5)
console.log(t.length)
console.log(t[1])
t.forEach(value => {
  console.log(value)
})
```

即使使用const定义，也可以修改数组中的内容。因为数组是一个对象，数组变量指向这同一个对象。

遍历元素的一种方法是使用 `forEach` ，如示例所示， `forEach` 接收一个函数作为入参。forEach 为数组中的每个元素调用这个函数，并将单个项作为参数传递。

使用`push`方法可以添加元素。但推荐使用函数式编程的技巧。函数编程范型的一个特点就是使用不可变的数据结构。在React代码中，最好使用 `concat` 方法。它不向数组中添加元素，而是创建一个新数组，新数组中包含旧数组和新元素。

```javascript
const t = [1, -1, 3]
const t2 = t.concat(5)
console.log(t2)
```

`map`方法可创建新数组。

```javascript
const t = [1, 2, 3]
const m1 = t.map(value => value *2)
console.log(m1)
```

此外，还可转换为不同的内容。

```javascript
const m2 = t.map(value => '<li>' + value + '</li>')
console.log(m2)
```

还可使用解构赋值方式将数组中的单个元素赋值给变量。

```javascript
const t = [1, 2, 3, 4, 5]
const [first, second, ...rest] = t
console.log(first, second)
console.log(rest)
```

例子中剩余的整数被“收集”到另一个数组中，分配给rest。

## 对象(Objects)

一个使用对象的例子：

```javascript
const object1 = {
  name: 'Arto Hellas',
  age: 35,
  education: 'PhD',
}

const object2 = {
  name: 'Full Stack web application development',
  level: 'intermediate studies',
  size: 5,
}

const object3 = {
  name: {
    first: 'Dan',
    last: 'Abramov',
  },
  grades: [2, 3, 5, 3],
  department: 'Stanford University',
}

// 对象元素的访问可通过.或[]进行
console.log(object1.name)         // Arto Hellas is printed
const fieldName = 'age' 
console.log(object1[fieldName])    // 35 is printed

// 可以动态添加对象属性
object1.address = 'Helsinki'
object1['secret number'] = 12341
```

## 函数（Functions）

函数定义的几种方式：

```javascript
const sum = (p1, p2) => {
  console.log(p1)
  console.log(p2)
  return p1 + p2
}
const result = sum(1, 5)
console.log(result)

// 只有一个参数时可省略括号
const square = p => {
  console.log(p)
  return p * p
}

// 只有一个表达式时可省略大括号
const square = p => p * p
// 该方法适合操作数组
const t = [1, 2, 3]
const tSquared = t.map(p => p * p)

// 或可使用function关键字定义
function product(a, b) {
  return a * b
}

const result = product(2, 6)

// 或使用函数表达式
const average = function(a, b) {
  return (a + b) / 2
}

const result = average(2, 5)
```

本课程推荐使用箭头语法。

## 对象方法和"this"关键字

尽量避免使用this关键字以防止问题。
使用 `setTimeOut`方法，避免问题的方法：

```javascript
const arto = {
  name: 'Arto Hellas',
  greet: function() {
    console.log('hello, my name is ' + this.name)
  },
}

setTimeout(arto.greet, 1000)
// 使用bind来保留原始的this
setTimeout(arto.greet.bind(arto), 1000)
```

## 类(Classes)

Javascript中的类非常像Java中的类，但其实质上仍然是Object。课程建议使用`hooks`特性，所以不具体使用类语法。