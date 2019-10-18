---
title: intersection原理
date: 2019-01-30 15:32:13
tags:
---
假如 Intersections 可以满足 兼容性那么 实现 元素曝光监听是开心美好的
```
function visibility(param){   
    //获取dom节点  
    this.ele = document.querySelectorAll(param.ele);    
    this.direction = param.direction;
    this.viewport = param.viewport;
    this.isclose = param.isclose;
    //事件
    this.io = null;
    this.callback = param.callback;    
}
//添加监听
visibility.prototype.addObserve = function(){
    let that = this;  
    that.io = new IntersectionObserver((res)=>{      
        that.Intersections(res)
    },{         
        root: that.viewport,      
        rootMargin:'0px',       
        threshold: [ 0, Number.MIN_VALUE, 0.01]        
    });    
    that.ele.forEach(function(item){
             that.io.observe(item);
    })   
}
//判断是否出现
visibility.prototype.Intersections = function(entries){               
    if(// 正在交叉
        entries[0].isIntersecting ||
        // 交叉率大于0
        entries[0].intersectionRatio){
        this.callback&&this.callback(entries[0]);
        if(this.isclose){          
           this.io.unobserve(entries[0].target);
        }      
    }
} 

/**
 * 判断元素何时可见
 * @param {element} ele 被监听的元素
 * @param {element} root 默认是body
 * @param {string}  direction 默认是vertical
 * @param {array} threshold 可视区域出现比例比例如果符合就会回调
 * @param {boolen} isclose 是否关闭监视器
 * @param {fn} callback 回调函数
 */
function isVisibility(param){   
    //根视口
    let defaults = {
        ele:'',
        root:typeof window !== 'undefined' ? window.HTMLElement : Object,
        direction:'vertical',  
        isclose:false,     
        threshold:[ 0, Number.MIN_VALUE, 0.01]      
    }
    defaults = Object.assign(defaults,param); 
    // 初始化
    let visibilityObj = new visibility(defaults);
    //添加监听
    visibilityObj.addObserve();
    return visibilityObj;
}

module.exports = isVisibility;
```
然而显示是残酷的，很多浏览器都不兼容这个属性，在网上看了一位大神写的polyfill 解决了这个问题
[https://github.com/w3c/IntersectionObserver/blob/master/polyfill/intersection-observer.js]这是那个链接，可是如果我仅仅只要实现最基本的功能就可以了，于是在这个js基础上进行缩减.