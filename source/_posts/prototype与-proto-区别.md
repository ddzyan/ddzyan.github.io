---
title: prototype与__proto__区别
date: 2018-08-24 01:59:10
categories: javascript
tags:
- javascript
---
### prototype（原型属性）
只有函数才具有 prototype 属性。prototype 是一个对象，默认有一个属性 constructor 指向原型对象的构造函数。

### __ proto __
所有对象都具有的 __ proto __ 属性，可称为隐式原型对象。一个对象的隐式原型对象指向原型对象的 prototype 属性，这也保证了实例化对象能共享原型对象中 prototype 属性的方法和属性。

<!--more-->

### 例子
```js
function Demo() {
    this.name = 'a';
    this.apple = 'd';
}

Demo.age = 'b';

Demo.prototype.writeTest = function () {
    console.log(this.test);
}

const demo = new Demo();

demo.apple = 'c'

//判断对象自身是否具有该属性
console.log(demo.hasOwnProperty('writeTest'));//false

//判断对象和原型对象是否具有该属性
console.log('apple' in demo);//true

//判断原型对象是否具有该属性
console.log(hasPrototypeProperty(demo,'writeTest'));//true

console.log(demo.apple);// c
console.log(demo.__proto__ == Demo.prototype); //true
console.log(Demo.__proto__.__proto__ == Object.prototype); // true

//是否为原型对象属性
function hasPrototypeProperty(obj, name){
    return name in obj && !obj.hasOwnProperty(name);
}
```
Demo 是原型对象，demo 是Demo实例化的对象。在实例化的过程中，值传递如下：
1. demo 对象传入 Demo 的构造函数中,构造函数中的赋值将添加到 demo 属性上。
2. demo 的 __ proto __ 等于 Demo.prototype 

由于 __ proto __ 对象为引用类型，所以修改 Demo.prototype 上的属性或者方法，将影响所有实例化对象。

#### 注意
如果要取 demo 上的属性，解析器的查找顺序如下：
1. 检查 demo 对象是否具有该属性
2. 检查 demo 的 __ proto __ 是否具有该属性

如果实例化对象和原型对象的 prototype 具有相同属性或者方法，则只会取到实例化对象的值。

#### 检查方法
hasOwnProperty() 判断指定属性是否为自有属性；in操作符对原型属性和自有属性都返回true。 结合两者可以判断属性实在实例化对象上，还是在原型对象上。