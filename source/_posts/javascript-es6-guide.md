---
title: JavaScript ES6+ 新特性完全指南
date: 2025-02-10
tags:
  - JavaScript
  - ES6
  - 前端
categories:
  - 前端开发
cover: https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=800&q=80
excerpt: 全面介绍 ES6+ 的核心特性，包括解构赋值、箭头函数、Promise、async/await 等现代 JavaScript 必备知识。
---

## 前言

ES6（ECMAScript 2015）是 JavaScript 语言的一次重大更新，引入了大量新特性。本文将介绍最常用的 ES6+ 特性。

## 1. let 和 const

```javascript
// let - 块级作用域变量
let count = 0;
count = 1; // 可以重新赋值

// const - 块级作用域常量
const PI = 3.14159;
// PI = 3; // 报错！不能重新赋值

// const 对象的属性可以修改
const user = { name: 'Tom' };
user.name = 'Jerry'; // 这是允许的
```

## 2. 箭头函数

```javascript
// 传统函数
function add(a, b) {
  return a + b;
}

// 箭头函数
const add = (a, b) => a + b;

// 多行箭头函数
const greet = (name) => {
  const message = `Hello, ${name}!`;
  return message;
};
```

## 3. 模板字符串

```javascript
const name = '世界';
const greeting = `你好，${name}！`;

// 多行字符串
const html = `
  <div class="card">
    <h2>${title}</h2>
    <p>${content}</p>
  </div>
`;
```

## 4. 解构赋值

```javascript
// 对象解构
const { name, age, hobby = '编程' } = user;

// 数组解构
const [first, second, ...rest] = [1, 2, 3, 4, 5];

// 函数参数解构
function printUser({ name, age }) {
  console.log(`${name} is ${age} years old`);
}
```

## 5. Promise 和 async/await

```javascript
// Promise
function fetchData(url) {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then(res => res.json())
      .then(data => resolve(data))
      .catch(err => reject(err));
  });
}

// async/await - 更优雅的异步写法
async function getData() {
  try {
    const data = await fetchData('/api/users');
    console.log(data);
  } catch (error) {
    console.error('请求失败:', error);
  }
}
```

## 6. 展开运算符

```javascript
// 数组展开
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

// 对象展开
const defaults = { theme: 'light', lang: 'zh' };
const settings = { ...defaults, theme: 'dark' };
```

## 总结

ES6+ 让 JavaScript 变得更加现代和强大。掌握这些特性是前端开发的基础。建议多写代码练习，逐步在项目中使用这些新特性。

## 参考资料

- [MDN Web Docs](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
- [ES6 入门教程 - 阮一峰](https://es6.ruanyifeng.com/)
