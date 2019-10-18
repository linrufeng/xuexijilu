---
title: new和对象
date: 2019-01-29 09:53:02
tags:
---
在说 new 这个操作符之前，我先通过 一个普通的函数 和 它的实例化来分析

```
 function ab(name){
     var that = this;
     console.log(this,name)
 }
 ab('里')
```

#### 我们就会发现 this 指向的是 window, 
#### 当我们使用 new 来初始化这个函数的时候 this 指向的是这个函数对象。

接下来看下函数对象终的 prototpye，我们可以这样用
```
 ab.prototype.add = () => {
    console.log(this)
    return this;
 }

```
在函数被创建时候 这个属性 prototype = null ，prototype对象是实现面向对象的一个重要机制。每个函数也是一个对象，它们对应的类就是function，每个函数对象都具有一个子对象prototype。Prototype 表示了该函数的原型，
prototype表示了一个类的属性的集合。当通过new来生成一个类的对象时，prototype对象的属性就会成为实例化对象的属性。

```
let abs = new ab()
```
#### 这时 abs 的属性就有了 add，也就是说 prototype对象的属性 自动附加给了 new的对象 也就是 abs

那么这样有什么用呢？w3school里面说到 prototype 属性使您有能力向对象添加属性和方法。那么我添加了这些然后呢？我在什么场景下应用呢?

可以利用prototype 实现继承（就是为了可以做函数的功能的逻辑的复用）

```
function a(x){
    console.log(x+1)
}
a.prototype.afn = function(){
    console.log('a的属性')
}
function b (y){
   
}
b.prototype = a.prototype; //把 a 的 prototype 附给了 b的prototyp 
b.prototype.Name="新属性name";
b.prototype.sub = ()=>{
    return this.Name;
}
b.prototype.sub2 = function(){
    return this.Name
}
var test = new b()
test.afn()
test.Name = 's';
test.id = 'test'
test.sub()
console.log(test.Name)
//这样 b 就继承 a 中 prototype 的方法 ，在 b  通过 new 之后 把 prototype对象的属性 附加给力 实例化后的对象 
```

在以上的代码中，

* 首先是 b 具有了和 a 一样的prototype，
* 如果不考虑构造方法，则两个类是等价的。
* 随后，又通过 prototype 给 b 赋予了额外的属性和方法
* 所以 b 是在 a 的 基础上 增加了 新的属性和方法 ，从而实现了类的继承。
* 在上面的例子中 我还发现了一个 问题 
**使用 Es6 语法 声明的函数 sub 中的 this 指向 window ，而使用 function 声明的函数 只向这个对象** 
在上面的例子中使用箭头函数 .prototype 会报错
```
 let a = ()=>{

 }
 a.prototype.test = function(){

 }
```
这是因为 **箭头函数内部没有 constructor与prototype属性 不能被new,且 箭头函数内部的this被绑定为函数时候被认为函数定义时候的this,并且无法改变**

```
var my = {
    firstName:"John",
    lastName: "Doe",
    fullName: function () {
        return this.firstName + " " + this.lastName; // this my
    },
    testName(){
        return this // this my
    },
    testName2:()=>{
        return this // this window
    }
}
```

--- 

由上面的例子可以得出 new 的 作用

1. 创建一个新对象；

2. 将构造函数的作用域赋给新对象（因此 this 就指向了这个新对象） ；

3. 执行构造函数中的代码（为这个新对象添加属性） ；

4. 返回新对象

### 面向对象： class + new 的方式创建对象

### 把let aObj = new a() 拆分来看

```
let aObj = {}
aObj.proto = a.prototype;
a.call(aObj);
```
1. 第一行，我们创建了一个空对象obj
2. 我们将这个空对象的proto成员指向了Base函数对象prototype成员对象
3. 我们将Base函数对象的this指针替换成obj，然后再调用Base函数，于是我们就给obj对象赋值了一个id成员变量，这个成员变量的值是”base”，关于call函数的用法。

---
### 小结

1. new 可以把一个函数对象，实例化一个新的对象，新的对象proto 会复制 原函数的prototype。
2. new 可以改变 function 声明的函数的 this 指向
3. new 无法改变 ()=>{}函数里面的this 指向
4. this 的只向问题
5. 简单聊了下继承
6. 箭头函数和 function 的区别


