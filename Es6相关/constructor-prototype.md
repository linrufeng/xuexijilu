---
title: 'this-constructor-prototype'
date: 2018-11-30 16:50:48
tags:
---

说起这三个属性，肯定有一些同学和我一样，初学js时非常困惑，头大，一脸的迷茫。今天就来给大家彻底解决这些担心受怕的问题。

this就是函数赖以执行的对象。
this是对象。  
this依赖函数执行的上下文环境。 
this存在函数中。

prototype

prototype是函数的一个属性，并且是函数的原型对象。引用它的必然是函数，

```function func(){
    var args = Array.prototype.slice.call(arguments, 0);
    return arguments;
 }
 func(1,2,3,4,5)
```
call是函数的一个方法，关于这个方法，它也是只有····函数····才能够调用的，它的作用是：调用引用它的函数。

就拿这个Array.prototype.slice.call(arguments,1)来讲，这里面包含太多信息了，我一个个分析一下。

slice(start[,end])是js的一个原生数组函数，作用是获取数组中从start下标开始到end下标结束的元素。

arguments是js函数对象的一个属性，作用是获取函数的实参，返回的是一个以函数实参为属性元素的对象。

function args(){
    return this.arguments;//这里的this可以省略，你自己可以试一下
    //我加上是为了说明，调用arguments的只能是对象
}
alert(JSON.stringify(args(1,3,5,6,8)));

　而Array是js中生成数组的关键字。

用Array.prototype来调用slice函数呢？而不用Array.slice，原因很简单，因为数组的关键字Array不能这样子Array.xx直接调用js的数组函数。

```alert(Array.slice());
//Uncaught TypeError: Array.slice is not a function```

Array关键字确实不能直接调用数组的函数。

alert(JSON.stringify(Array.prototype));
alert(JSON.stringify(Array.prototype.slice()));

　这里返回都是空数组···[]···，说明了什么？说明了Array关键字确实是可以调用prototype函数的属性的，同时也说明js是可以这样子Array.prototype调用js的数组函数的。

　转个弯说，看官是否还记得js生成数组的几种方式？应该有多种，但，我这里就不介绍了。

　不过，你是否看过这样生成数组的方式？我们先来看下面的代码：

var arr = new Array();

那么，我们js的function是不是也可以像下面这样子生成对象呢？

function func(){

}
var obj = new func();

Array()这个东西其实是js一个·····构造数组的内置函数····

我们再返回来说js可以这样子Array.prototype调用prototype就很明白不过了是吧？Array()是js的一个内置函数，既然Array是一个函数，那么Array肯定拥有prototype这个属性对吧？所以说，Array关键字调用prototype并没有违反prototype是函数的一个属性这个原则，prototype是函数的一个属性依然是一个不变的结论。

Array.prototype是一个空数组，

　  Array.prototype.slice()的意思是一个空数组调用数组的函数slice()，

　  Array.prototype.slice.call()的意思是call函数调用数组的函数slice()，

　  这里call()怎么调用slice()呢？

　  是这样子的，Arguments获取func函数实参列表，生成一个对象传递给call()函数，call函数又把Arguments生成的对象传递给Array.prototype这个空数组，把第二个参数···1···传递给slice函数，然后，call就让引用它的函数slice执行slice(1)，所以slice就从1下标开始获取数组的元素了，而这个数组的元素其实就是Arguments的元素，因此，这个函数func的作用就是获取下标为1开始的所有实参。不相信，你自己可以执行一下上面的函数。

下面讲讲prototype的应用：

应用1：

给原型对象增加函数，就是让对象拥有公用的函数。

应用2：

给原型对象增加属性，也就是给对象增加公用的属性

应用3：

实现原型继承；
```
    function P1(){
    
    }
    function P2(){
        
    }
    //原型对象增加属性和方法
    P2.prototype.name = 'P2"s name';
    P2.prototype.get=function(value){
            return value;
    }
    //实例化P2构造函数的一个对象
    var obp2 = new P2();//这个对象应该包含所有原型对象的属性和方法
    //给P1的原型对象赋值一个对象，相当于P1继承了obp2的所有属性和方法
    P1.prototype = obp2;//这个式子，简单来讲就类似于a = b, b赋值给a这个总该明白吧？
    //调用P1从obp2继承过来的get函数
    alert(P1.prototype.get('out"s name'));
    //展示P1从obp2继承过来的name属性
    alert(P1.prototype.name);
    //用构造函数P1实例化一个obp1对象
    var obp1 = new P1();
    //P1的原型对象prototype既然已经继承了obp2的所有属性和函数，那么依据P1所实例化出来的对象也都有obp2的属性和函数了
    alert(obp1.get('obp1"s name'));```

    　 特别指出：

　　Array.prototype是一个数组

　　String.prototype是一个字符串

　　Object.prototype是一个对象