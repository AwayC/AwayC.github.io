# js先决条件

#### 前言

这里是记录和总结three.js提供的[引导教程](https://threejs.org/manual/#eh/prerequisites)，真的比较良心，可以规范编码，生怕你不会一些js的机制而不会使用three.js😂

### 1. HTML使用

```html
<html>
  <head>
    ...
  </head>
  <body>
     ...
  </body>
  <script>
    // inline javascript
  </script>
</html>
```



### 2. `document.querySelector`和`document.querySelecorAll`

这两个让你用js选择与css匹配的元素。



### 3. 闭包工作

这里会a是一个创建函数的函数。

```js
function a(v) {
  const foo = v;
  return function() {
     return foo;
  };
}
 
const f = a(123);
const g = a(456);
console.log(f());  // prints 123
console.log(g());  // prints 456
```

### 4.this的使用

注意this被设为的对象，函数被`.`调用是this是调用的对象，否则是`null`

```js
 const callback = someobject.somefunction;
 loader.load(callback); //这里this就会是null
```



### 5.ES5/ES6/ES7 特性

`var`被弃用，使用`const`和`let`

使用`for(elem of collection)`替代`for(elem in collection)`

```js
for (const [key, value] of Object.entries(someObject)) {
  console.log(key, value);
}
```

使用`forEash`,`map`和`filter`

使用解构赋值

```js
const dim = {width: 300, height: 150}; 
const {width, height} = dims; 
```

简写对象声明

```js
 const width = 300;
 const height = 150;
 const obj = {
   width,
   height,
   area() {
     return this.width * this.height;
   },
 };
```

使用扩展运算符`...`

```js
function log(className, ...args) {
    const elem = document.createElement('div');
    elem.className = className;
    elem.textContent = [...args].join(' ');
    document.body.appendChild(elem);
}

const position = [1, 2, 3];
somemesh.position.set(...position);

```

使用`class`来生成类

使用箭头函数`(var) => { } `，且this会自动绑定到调用对象上

Promises改善异步代码

使用反引号的字符模板

### 最后

命名规则： 

- **小驼峰**：变量，函数，方法
- **大驼峰**：构造函数，类



