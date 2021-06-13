---
title: JavaScript笔记（一）
date: 2019-10-31 00:01:14
categories: 前端
tags: JavaScript
---


## 1.浅拷贝深拷贝
- 如图：
![示例](/assets/8393845323.png)

<!-- more -->

## 2.创建对象的几种方法
### 2.1 工厂模式
> 缺点：无法识别对象类型
### 2.2 构造函数模式
> 解决了类型识别的问题，但是创建实例时构造函数中的方法都得重新创建一遍
### 2.3 原型模式
> 属性共享，实例中的某个属性的更改会反映到其他实例中，而实际中往往实例是需要有属于自己的属性的
### 2.4 构造函数 + 原型模式
> 核心就是构造函数定义私有的属性，公有的方法写在原型对象中
### 2.5 动态原型模式
> 即在构造函数中动态的给原型定义方法，通过条件判断语句判断某个原型方法是否被创建过，这样一来无论创建多少个实例，方法都只会在第一次构造函数调用时创建
```javascript
function Person(name, age, job) {
  //属性
  this.name = name;
  this.age = age;
  this.job = job;
  //方法
  if (typeof this.sayName != "function") {
    Person.prototype.sayName = function () {
      alert(this.name);
    };
  }
}
```
### 2.6 寄生构造函数模式
> 这种模式的基本思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象；但从表面上看，这个函数又很像是典型的构造函数
```javascript
function Person(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function () {
    alert(this.name);
  };
  return o;
}
var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName(); //"Nicholas"
```
### 2.7 稳妥构造函数模式
> 所谓稳妥对象，指的是没有公共属性，而且其方法也不引用 this 的对象。稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用 this 和 new），或者在防止数据被其他应用程序（如 Mashup
程序）改动时使用。稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：一是新创建对象的
实例方法不引用 this；二是不使用 new 操作符调用构造函数
```javascript
function Person(name, age, job) {
  //创建要返回的对象
  var o = new Object();
  
  //可以在这里定义私有变量和函数
  
  //添加方法
  
  o.sayName = function () {
    alert(name);
  };
  //返回对象
  return o;
}
```
## 3.继承
### 3.1 原型链

> 实现原型链有一种基本模式, 让子类构造函数的prototype属性指向父类的实例：
```javascript
function SuperType() {
  this.property = true;
}
SuperType.prototype.getSuperValue = function () {
  return this.property;
};
function SubType() {
  this.subproperty = false;
}
//继承了 SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function () {
  return this.subproperty;
};
var instance = new SubType();
alert(instance.getSuperValue()); //true
```
### 3.2 借用构造函数
> 这种技术的基本思想相当简单，即在子类型构造函数的内部调用父类构造函数。
```javascript
function SuperType() {
  this.colors = ["red", "blue", "green"];
}
function SubType() {
  //继承了 SuperType
  SuperType.call(this);
}
```
相对于原型链而言，借用构造函数有一个很大的优势，即可以在子类型构造函数中向超类型构造函数传递参数。
### 3.3 组合继承
> 这种方式使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。

> 缺点：每次都要调用两次父类的构造函数
```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function () {
  alert(this.name);
};

function SubType(name, age) {
//继承属性
  SuperType.call(this, name);
  this.age = age;
}

//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function () {
  alert(this.age);
};
var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"
instance1.sayName(); //"Nicholas";
instance1.sayAge(); //29
var instance2 = new SubType("Greg", 27);
alert(instance2.colors); //"red,blue,green"
instance2.sayName(); //"Greg";
instance2.sayAge(); //27
```

### 3.4 原型式继承
> 借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。
```javascript
function object(o){
  function F(){}
  F.prototype = o;
  return new F();
}
```
### 3.5 寄生式继承
> 寄生式（ parasitic）继承是与原型式继承紧密相关的一种思路，并且同样也是由克罗克福德推而广
之的。寄生式继承的思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数，该
函数在内部以某种方式来增强对象，最后再像真地是它做了所有工作一样返回对象。

> ps: 使用寄生式继承来为对象添加函数，会由于不能做到函数复用而降低效率；这一
点与构造函数模式类似。
```javascript
function createAnother(original){
  var clone = object(original); //通过调用函数创建一个新对象
  clone.sayHi = function(){ //以某种方式来增强这个对象
  alert("hi");
  };
  return clone; //返回这个对象
}
```
### 3.6 寄生组合式继承
> 组合继承是JavaScript最常用的继承模式；不过，它也有自己的不足。组合继承最大的
问题就是无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是
在子类型构造函数内部。没错，子类型最终会包含超类型对象的全部实例属性，但我们不得不在调用子
类型构造函数时重写这些属性。

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function () {
  alert(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name); //第二次调用 SuperType()
  this.age = age;
}

SubType.prototype = new SuperType(); //第一次调用 SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function () {
  alert(this.age);
};
```
> 所谓寄生组合式继承，即通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。其背
后的基本思路是：不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型
原型的一个副本而已。本质上，就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型
的原型。
```javascript
function inheritPrototype(subType, superType){
  var prototype = object(superType.prototype); //创建对象
  prototype.constructor = subType; //增强对象
  subType.prototype = prototype; //指定对象
}
```
## 4. 实现new操作符
首先了解需要new操作符做了哪些事情

```
/**
 * 1. 创建一个空的简单JavaScript对象（即{}）；
 * 2. 链接该对象（即设置该对象的构造函数）到另一个对象 ；
 * 3. 将步骤1新创建的对象作为this的上下文 ；
 * 4. 如果该函数没有返回对象，则返回this。
 */
```
TODO

## 5. 闭包

闭包可以理解成能够访问其外部函数作用域的函数。

如何从函数外部读取内部的局部变量？用闭包可以实现。

## 6. 节流与防抖

### 6.1 函数节流
    函数节流是指在规定的时间内函数只能触发一次。如果在这个时间内多次触发，只有一次生效。
    
```javascript 

function throttle(func, delay) {
  var last;
  var timer;
  return function() {
    var now = +new Date();
    var ctx = this;
    var args = arguments;
    
    if (last && now - last < delay) {
      clearTimeout(timer);
      timer = setTimeout(function(){
        last = now;
        func.apply(ctx, args);
      }, delay)
    } else {
      last = now;
      func.apply(ctx, args);
    }
  }
}

```
    
### 6.2 函数防抖
    函数防抖是指函数在n秒后触发，如果在n秒内再次被触发，那么将重新计时

```javasript
function debounce(func, delay) {
  var timer;
  return function() {
    var this = ctx;
    var args = arguments;
    clearTimeout(timer);
    timer = setTimeout(function(){
      func.apply(ctx, args);
    }, delay);
  }
}
```
关于什么时候使用节流什么时候使用防抖：

防抖：在实现搜索框联想时，需要发送请求与后端交换数据，用户在开始输入到完成之前，这段时间出现的字符都不是用户想要搜索的，所以可以用到防抖函数，只在用户输入完成后的n秒后发送请求，而输入的过程并不发送请求，这样就大大减少了服务器压力

节流：例如一个点击一个按钮发送一次请求，如果在很短的时间内点击了n次按钮就会发送n次请求，为了防重点击，就可以使用到节流，即用户在一定的时候间隔内只能发送一次请求，不管用户以多快的频率点击，反正只会在设定的时间执行一次；又比如玩坦克大战时不管你以多快的速度按按钮射击炮弹，坦克只会以稳定的频率发出炮弹，这就是节流

节流和防抖可以交换使用吗？

首先分析一下第一个搜索联想的例子，对用户而言他想要的只是在输入结束之后发起联想，例如搜索“维基百科”，用户既不是想要搜索“维”，也不是想要搜索“维基”，就是说用户只关心最后完成的输入。这个例子使用节流我觉得就不太好，因为节流函数不管怎么样都会在最开始的第一次被触发，这就与实际想要的效果相悖了

再来看第二个例子，用户在很短的时间内点击了多次按钮，如果使用防抖函数，产生的效果是：n秒内在最后一次触发事件延迟n秒执行。那么假设用户现在正常操作，点一次按钮，发现竟然过了n秒才产生响应，这就会让用户产生卡顿的的感觉。


## 7. ES6 Module

特性：

    1. 编译时加载，早于其他语句
    2. 不允许运行时改变
    3. 动态绑定
    export var foo = 'bar';
    setTimeout(() => foo = 'baz', 500);
    上面代码输出变量foo，值为bar，500 毫秒之后变成baz。
    
## 8. 事件流
    事件流描述的是从页面中接收事件的顺序