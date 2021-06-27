---
layout: post
title:  "全栈开发从入门到放弃:1.a-React基础"
date:   2021-06-17 22:25:00
categories: FullStack
tags:   [React, 全栈开发, 学习笔记]
---

>本系列介绍：《全栈开发从入门到放弃》是本人学习如何使用JavaScript开发现代Web应用程序的过程记录。课程的重点是使用ReactJS构建单页面应用程序（SPA）。

>本系列记录了本人学习过程中的要点，不成体系。如果你想深入学习，建议寻找官方教程 😃

---

## 新建React应用

创建一个React应用可用`create-react-app`，如：

```shell
npx create-react-app part1
cd part1
npm start
```

## 组件(component)

在使用React开发时，通常将需要渲染的内容定义为React组件。一个定义组件的示例如下：

```javascript
const App = () => {
  const a = 10
  const b = 20
  return (
    <div>
      <p>Hello world</p>
      <p>
        {a} plus {b} is {a + b}
      </p>
    </div>
  )
}
```

JavaScript中大括号中的代码会被计算，计算结果将嵌入到组件生成的HTML中。可渲染动态内容。

## JSX

JSX 是 JavaScript 的语法扩展。在底层，React 组件返回的 JSX 会被`Babel`编译成 JavaScript 。

JSX 是类XML语言，每个标签都需要关闭。如HTML中的`<br>`在JSX中，写作`<br />`。

## 多组件(Multiple component)

使用React编写组件很容易，定义的组件可以在其他组件中使用。通常应用的组件树顶部有个root组件叫App，但这也可修改。

## props: 向组件传递数据

使用`props`，可将数据传递给组件。可以有多个props值传递，如果props是JavaScript表达式，则需要使用花括号。

## 一些注意事项

1. **React组件名称首字母必须大写**
2. React组件内容通常需要包含**一个根元素**
3. 如果不想在Dom树中看到多余的`div`元素，可使用空元素包装组件的返回内容。
