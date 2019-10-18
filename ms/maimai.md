js:


https和http请求
https对哪些进行了加密
如何实现跨越？除了jsonp还有什么方法

vue如何实现双向数据绑定


如何实现深度克隆，除了 for 循环 还有什么方法

react sitState({a:1})sitState({a:2})sitState({a:3}) rendenr 执行了多少次

vue router 两种方式，差异，好处 

算法:

  查找 [1,2,3,4,5,6,7,8,9] a+b=5 找出 a ，b ? 时间复杂度？
  排序 

##vue里面的 css 怎么实现 作用域的
vue中的scoped属性的效果其实主要是通过PostCSS转译实现。
vue-loader文件中有一个lib文件夹 下面有一个style-compiler文件夹，里面的index.js就是处理css的。
里面加载的就是PostCss，PostCSS给组件中的所有dom添加了一个独一无二的动态属性，然后，给CSS选择器额外添加一个对应的属性选择器来选择该组件中dom，原理是利用css的属性选择器。
## setTimeout和setInterval的区别以及如何写出效率高的倒计时

网络请求部分：

如何实现 js和客户端通信
post请求哪些地方会加密

### 如何解决跨域问题
#### JSONP：

原理是：动态插入script标签，通过script标签引入一个js文件，这个js文件载入成功后会执行我们在url参数中指定的函数，并且会把我们需要的json数据作为参数传入。
由于同源策略的限制，XmlHttpRequest只允许请求当前源（域名、协议、端口）的资源，为了实现跨域请求，可以通过script标签实现跨域请求，然后在服务端输出JSON数据并执行回调函数，从而解决了跨域的数据请求。
优点是兼容性好，简单易用，支持浏览器与服务器双向通信。
缺点是只支持GET请求。

JSONP：json+padding（内填充），顾名思义，就是把JSON填充到一个盒子里

```
<script>

    function createJs(sUrl){

        var oScript = document.createElement('script');
        oScript.type = 'text/javascript';
        oScript.src = sUrl;
        document.getElementsByTagName('head')[0].appendChild(oScript);
    }

    createJs('jsonp.js');

    box({
       'name': 'test'
    });

    function box(json){
        alert(json.name);
    }
</script>
```
### CORS
服务器端对于CORS的支持，主要就是通过设置Access-Control-Allow-Origin来进行的。如果浏览器检测到相应的设置，就可以允许Ajax进行跨域的访问。
### 通过修改document.domain来跨子域
将子域和主域的document.domain设为同一个主域.前提条件：这两个域名必须属于同一个基础域名!而且所用的协议，端口都要一致，否则无法利用document.domain进行跨域
主域相同的使用document.domain
### 使用window.name来进行跨域
window对象有个name属性，该属性有个特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限，window.name是持久存在一个窗口载入过的所有页面中的
### 使用HTML5中新引进的window.postMessage方法来跨域传送数据
还有flash、在服务器上设置代理页面等跨域方式。个人认为window.name的方法既不复杂，也能兼容到几乎所有浏览器，这真是极好的一种跨域方法。
### 说说TCP传输的三次握手四次挥手策略
为了准确无误地把数据送达目标处，TCP协议采用了三次握手策略。用TCP协议把数据包送出去后，TCP不会对传送    后的情况置之不理，它一定会向对方确认是否成功送达。握手过程中使用了TCP的标志：SYN和ACK。
发送端首先发送一个带SYN标志的数据包给对方。接收端收到后，回传一个带有SYN/ACK标志的数据包以示传达确认信息。
最后，发送端再回传一个带ACK标志的数据包，代表“握手”结束。
若在握手过程中某个阶段莫名中断，TCP协议会再次以相同的顺序发送相同的数据包。
>>> 断开一个TCP连接则需要“四次握手”：
第一次挥手：主动关闭方发送一个FIN，用来关闭主动方到被动关闭方的数据传送，也就是主动关闭方告诉被动关闭方：我已经不 会再给你发数据了(当然，在fin包之前发送出去的数据，如果没有收到对应的ack确认报文，主动关闭方依然会重发这些数据)，但是，此时主动关闭方还可 以接受数据。
第二次挥手：被动关闭方收到FIN包后，发送一个ACK给对方，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号）。
第三次挥手：被动关闭方发送一个FIN，用来关闭被动关闭方到主动关闭方的数据传送，也就是告诉主动关闭方，我的数据也发送完了，不会再给你发数据了。
第四次挥手：主动关闭方收到FIN后，发送一个ACK给被动关闭方，确认序号为收到序号+1，至此，完成四次挥手。
###TCP和UDP的区别
TCP（Transmission Control Protocol，传输控制协议）是基于连接的协议，也就是说，在正式收发数据前，必须和对方建立可靠的连接。一个TCP连接必须要经过三次“对话”才能建立起来
UDP（User Data Protocol，用户数据报协议）是与TCP相对应的协议。它是面向非连接的协议，它不与对方建立连接，而是直接就把数据包发送过去！
UDP适用于一次只传送少量数据、对可靠性要求不高的应用环境。
### 创建ajax过程
(1)创建XMLHttpRequest对象,也就是创建一个异步调用对象.

(2)创建一个新的HTTP请求,并指定该HTTP请求的方法、URL及验证信息.

(3)设置响应HTTP请求状态变化的函数.

(4)发送HTTP请求.

(5)获取异步调用返回的数据.

(6)使用JavaScript和DOM实现局部刷新.
### 渐进增强和优雅降级
渐进增强 ：针对低版本浏览器进行构建页面，保证最基本的功能，然后再针对高级浏览器进行效果、交互等改进和追加功能达到更好的用户体验。

优雅降级 ：一开始就构建完整的功能，然后再针对低版本浏览器进行兼容。


css:

M端 如何防止标签被选中？
div如何让标签失效
img标签图片如何 100*100完美展示
## display:none;
## visibility:hidden 这意味着元素仍占据其本来的空间，不过可以完全不可见。
## opacity:0

##position的值， relative和absolute分别是相对于谁进行定位的？


absolute :生成绝对定位的元素， 相对于最近一级的 定位不是 static 的父元素来进行定位。
fixed （老IE不支持）生成绝对定位的元素，相对于浏览器窗口进行定位。
relative 生成相对定位的元素，相对于其在普通流中的位置进行定位。
static  默认值。没有定位，元素出现在正常的流中

##reletive 会改变什么