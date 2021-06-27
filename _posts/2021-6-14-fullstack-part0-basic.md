---
layout: post
title:  "全栈开发从入门到放弃:0-Web应用的基本介绍"
date:   2021-06-14 22:25:00
categories: FullStack
tags:   [React, 全栈开发, 学习笔记]
---

>本系列介绍：《全栈开发从入门到放弃》是本人学习如何使用JavaScript开发现代Web应用程序的过程记录。课程的重点是使用ReactJS构建单页面应用程序（SPA）。

>本系列记录了本人学习过程中的要点，不成体系。如果你想深入学习，建议寻找官方教程 😃

---

## 必要软件安装

1. Chrome 或 Firefox 开发版。
2. Git 和 GitHub。
3. Visual Studio Code。
4. Node.js

**Web开发准则**：始终打开开发者控制台。

## HTTP GET

服务器和浏览器使用HTTP协议相互通信， 浏览器可使用 Get 方法发送请求。

## 传统的网络应用

由于含有变量，模板字符串是可运行的字符串。

课程使用 `Node.js` 和 `Express` 创建Web服务器。

使用 `Console` 选项卡和 `console.log` 命令可以方便调试。

## 事件处理和回调函数

调用事件处理程序的机制在 `Javascript` 中非常常见。事件处理函数被称为回调函数。应用代码时不直接调用函数本身，而是运行时浏览器会在事件发生的适当时间调用函数。

## 文档对象模型(Document Object Model, DOM)

`DOM` 是一个 API， 它支持对web页面对应的元素树进行编程修改。

## 从console中操作文档对象

html 文档 DOM 树的最顶层节点称为文档对象(`document`)。可以在控制台的console中操作文档对象。例如:

```javascript
list = document.getElementsByTagName('ul')[0];
newElement = document.createElement('li');
newElement.textContent = 'Page manipulation from console is easy.是吧'
list.appendChild(newElement)
```

注意，上述更改只是更改浏览器页面，并没有将更改推送到服务器。

## 层叠样式表(Cascading Style Sheets, CSS)

`CSS` 是一种用来确定web应用外观的标记语言。

CSS 中的类选择器用于选择页面中的某些部分，并对它们定义样式规则。例如：

```css
.container {
  padding: 10px;
  border: 1px solid;
}
```

类选择器的定义始终以句点 `.` 开头，并包含类的名称。

CSS 属性可以添加到 HTML 元素中，例如：

```html
<body>
  <div class="container">
  <h1>Notes</h1>
  </div>
</body>
```

HTML 元素也可以有 class 以外的属性。例如：

```html
<div id="notes">
```

上述代码中包含`id`属性，JavaScript 代码可使用 id 来查找元素。同样可以在控制台的`Elements`中更改元素样式，这也不是永久性的。

## 表单与HTTP POST

表单 Form 具有属性 action 和 method，例如：

```html
<form action="/new_note" method="POST">
  <input type="text" name="note">
  <br>
  <input type="submit" value="Save">
</form>
```

上述属性定义将表单作为一个 HTTP POST 请求提交到 `/new_note` 地址。数据作为 POST 请求的`body`发送到服务器。

例如，下述服务器端代码可通过访问请求对象req的body来访问发送来的数据。

```javascript
app.post('/new_note', (req, res) => {
  notes.push({
    content: req.body.note,
  });
  return res.redirect('/notes');
});
```

## AJAX

异步的JavaScript和XML(Asynchronous JavaScript and XML, AJAX) 是2005年2月引入的一个术语。它描述了一种新的革命性的方法，这种方法使用包含在 HTML 中的 JavaScript 来获取网页内容，而且不需要重新渲染页面。现在已经普遍应用。

## 单页面应用

单页面应用 Single-page application (SPA) 只从服务器获取一个HTML页面，内容由JavaScript在浏览器中执行操作。

## JavaScript库

JavaScript工具库操作页面更容易。jQuery因为它所谓的跨浏览器兼容性发展开来。

由于SPA的发展，几种更现代的开发方式流行开来。先是[BackboneJS](https://backbonejs.org/)， 然后有[AngularJS](https://angularjs.org/)。现在流行的有[React](https://reactjs.org/)和[Vue.js](https://vuejs.org/)。

此次学习课程中使用 React 和 [Redux](https://github.com/reactjs/redux) 库。

## 全栈web开发

课程中将关注应用的所有部分: 从前端、后端到数据库。将使用 JavaScript 编写后端代码，使用 Node.js 运行时环境。
