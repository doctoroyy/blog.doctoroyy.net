---
title: '模拟实现JavaScript中的apply, call函数'
date: 2019-12-24 23:53:19
categories: 前端
tags: JavaScript
---

## 1.apply(thisArg, [argsArray])
在实现之前，首先看一下MDN上关于apply函数的说明：
> apply() 方法调用一个具有给定this值的函数，以及作为一个数组（或类似数组对象）提供的参数。

apply函数会将当前调用函数的this绑定在thisArgs上，我们知道，在JavaScript中，常见的函数绑定this的方法除了apply,call之外，还会在函数被当做对象的属性调用时被这个对象绑定，
例如：
```
obj.func() // 执行时func中的this指向obj
```

<!-- more -->

根据这个思路，就容易写出代码了：

```
// 为了让所有的函数都能调用，需要定义在Function的原型对象上
Function.prototype.myApply = function(thisArg) {
  let fn = this;
  let args = arguments[1];
  thisArg.fn = fn; // 将函数添加到thisArgs的属性上，通过属性调用
  thisArg.fn(...args);
}

const obj = {
  name: 'oyy',
  age: 23,
}

const func = function(a, b, c) {
  console.log(this.name);
  console.log(a + b + c)
}

func.myApply(obj, [1, 2, 3])

```

上面的代码还有些问题，如果将obj打印出来，会发现obj多了一个fn属性，为了不污染源对象，需要在fn调用完之后手动将其删除

```
Function.prototype.myApply = function(thisArg) {
  let fn = this;
  let args = arguments[1];
  thisArg.fn = fn; // 将函数添加到thisArgs的属性上，通过属性调用
  thisArg.fn(...args);
  delete thisArg.fn;
}
```

另外，在thisArgs为null或者undefined时，thisArg会自动替换为指向全局对象。

```
Function.prototype.myApply = function(thisArgs) {
  if (!thisArgs) thisArgs = window; // 这里指向window
  let fn = this;
  let args = arguments[1] || [];
  thisArgs.fn = fn;
  thisArgs.fn(...args);
  delete thisArgs.fn;
}
```

如此一来，就基本上实现了apply函数。


## 2.call(thisArg, arg1, arg2, ...)
> call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。

与apply函数类似，call也能做到绑定函数的this，他们的区别在于传参方式有些不一样，call接受的是一个的参数列表

仍然使用上面的思路：

```
Function.prototype.myCall = function(thisArg) {
  if (!thisArg) thisArg = window;
  let fn = this;
  let args = [];
  for (let i = 1; i < arguments.length; i++) {
    args.push(arguments[i]);
    // args.push('arguments[' + i + ']');
  }
  thisArg.fn = fn;
  thisArg.fn(...args);
  // eval('thisArg.fn(' + args + ')');
  delete thisArg.fn;
}
```
上面两个函数都使用了扩展运算符'...'，如果不用这个的话，那么通过eval的方式传入一个构造好的参数字符串列表也可以达到一样的效果（看注释部分）

