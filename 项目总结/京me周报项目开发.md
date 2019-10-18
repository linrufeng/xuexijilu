### 项目背景

这个项目主要是每周五统计大家本周的工作情况，所以基本都是非工作时间，在最后还有一个鼓励的卡片，为了吸引大家的眼球，同时缓解大家一周来的压力，基本每个场景都加入了动画，同时使用了一些优化手段，提高了整体的效果。

### 技术构成

1. 开发框架vue + ts
2. 构建工具 webpack + yg
3. 开发环境 node
4. 开发语言 js、svg、h5、css、ts

### 开发心得

为了使用者的最大体验，我主要精力花在了项目图片处理，动画处理这两块,不知道在读我文章的你早遇到动画这类的项目，是不是很蛋疼的去百度各种动画的实现，各种动画的效果，希望可以用到自己的项目，其实我觉得一个动画的项目开发，百度来的效果往往是不合适的，我们要针对项目有自己的想想法，好的想法还要对应好的技术，在动画效果过程中我尽量把一些技巧总结给大家，那么正文开始吧。

### 性能提高篇

关于前端性能的提高有很多的方法，

* 减少 http 请求次数；
* 减少 DOM 元素数量；
* 延迟加载；
* 提前加载；

针对这些方法我会在下面的内容具体的向大家展示我的做法。

### 曝光元素技巧（实现元素曝光开始动画的原理）

为何要有这么一个功能，因为这个项目有很多动画只执行一次，为了能够让用户可以看到这些动画，只有在动画元素所在页面曝光的情况下我们才开始使用动动画，页面离开回到最初的状态。
```
verInViewport(el) {      
    let viewHeight = this.viewHeight;
    let rect = el.getBoundingClientRect();
    return (rect.top >= -10 && rect.top < viewHeight);
}   
```

![](https://user-gold-cdn.xitu.io/2019/9/19/16d473b1d2fe11d2?w=500&h=500&f=png&s=25422)
通过上面的图我们不难理解当 rect.top ===0 时候元素就完全曝光到我们的视野中了。
当 rect.top > viewHeigh 说明 元素位于我们视口的下方
当 rect.top < 0 的时候说明判断的元素在我们视口的上方
上面的代码就是判断元素是否曝光的方法，那么如何做到监听呢？
我是通过 swiper中的回调函数来触发这个函数的，并加大了一点判断的范围
```
slideChangeTransitionEnd() {
  _that.verInViewport();
}
```
我把完整的代码给大家展示下
```
<template>
  <div ref="showbox">
    <slot></slot>
  </div>
</template>
<script lang="ts">
import Vue from "vue";
import { Component, Prop, Emit, Watch } from "vue-property-decorator";
@Component
export default class isShow extends Vue {
  eleBox = null;
  viewHeight: number = 0;
  @Prop(Boolean) readonly onChanges!: Boolean;
  @Watch("onChanges") onChildChanged(val: Boolean) {
    let _that: any = this;
    if (val) {
      let show = _that.verInViewport(_that.eleBox);
      this.$emit("isShow", show);
    }
  }
  verInViewport(el) {
    let viewHeight = this.viewHeight;
    let rect = el.getBoundingClientRect();
  
    return rect.top >= -10 && rect.top < viewHeight;
  }
  mounted() {
    let _that: any = this;
    _that.eleBox = _that.$refs.showbox;
    _that.viewHeight = document.body.clientHeight;
  }
}
</script>
```
代码中的 `onChanges` 代替了 滚动等其他事件，去触发我对当前元素的检查，来完成曝光是否可以，
其实最初的想法是使用下面的方法
```
 new IntersectionObserver(this.Intersections,
 {  
    // 用于计算相交区域的根元素
    root: this.viewport,
    // 同 margin，可以是 1、2、3、4 个值，允许时负值。
    // 如果显式指定了跟元素，该值可以使用百分比，即根元素大小的百分之多少。
    // 如果没指定根元素，使用百分比会出错。
    rootMargin,
    // 触发回调函数的临界值，用 0 ~ 1 的比率指定，也可以是一个数组。
    // 其值是被观测元素可视面积 / 总面积。
    // 当可视比率经过这个值的时候，回调函数就会被调用。
    threshold: [ 0, Number.MIN_VALUE, 0.01]        
})
Intersections(entries){               
        if(// 正在交叉
            entries[0].isIntersecting ||
            // 交叉率大于0
            entries[0].intersectionRatio){
            //这时候我们元素曝光了
            this.showComponent();    
        }
}
```
无奈 ios 不支持，但是 IntersectionObserver 真的是一个不错的api ，使用起来不需要其它的手段配合，如果大家有的项目不需要在 ios上,还是可以使用这个方法的。
总的来说这个技术还可以用在懒加载上，例如我们在实际使用中只需要加载用户要看的部分内容。

##### ps
很多人问我为什么他的元素曝光不好用，其实最主要的原因就是你需要设置一个初始的站位，如果没有占位的话，包裹元素的那一层是空的初始化加载就会认为曝光了。我在很在之前在我们的 **NutUI** 中其实封装过一个组件，原理就是上面的代码，不同的是我多了一个兼容性的处理给大家看下兼容性的核心代码
```
requestAnimationFrame (callback) {
// 防止等待太久没有执行回调
// 设置最大等待时间
    setTimeout(() => {
        if (this.isInit ) {
            return
        }                
        callback()
    }, this.maxWaitingTime)
// 兼容不支持requestAnimationFrame 的浏览器
    return (window.requestAnimationFrame || ((callback) => setTimeout(callback, 1000 / 60)))(callback)
},
//元素滚动，直接使用的化严重的影响了性能，所以我在这里做了一个防抖节流
onScroll(){
    let that = this;
    window.addEventListener("scroll",that.listenElm)
},
listenElm(){
    let that = this;  
    // 定时函数代码见上
    this.requestAnimationFrame(() => {
            //  NewIntersectionObserver 就是在滚动过程中不停地去判断元素是否曝光，原理和上图的类似，我把代码粘贴到了下方
         if(that.NewIntersectionObserver({direction:that.direction})){
            that.showComponent(); // 这里是判是否完成曝光的函数
            //移除监听
            window.removeEventListener("scroll",that.listenElm)
        }
    })              
},
```
元素曝光的计算方式
```
NewIntersectionObserver(options){      
    //判断是否可见   
    let isShow = null;   
    switch (options.direction) {
        case 'vertical':
            isShow = verInViewport(this.$el,this.viewHeight);     // 横向的 见上面的原理不在给大家展示代码             
            break;
        case 'horizontal':
            isShow = horInViewport(this.$el,this.viewWidth);    // 水平的            
            break;
    }
    return isShow;
}
```
左右方向的代码
```
function horInViewport(el, viewWidth) {
    var rect = el.getBoundingClientRect();
    return (rect.left > 0 && rect.left < viewWidth);
}
```
这就是元素曝光展示动画的核心部分了。如果能够很好的理解这些，那么在以后的项目中遇到这类的需求就不会在满世界找方法了。

### 图片性能优化经验

随着人们对美好事物的追求，一个页面上图片越来越多，尤其是工作做的一些宣传类行的项目图片+动画的形式，让我们对于图片的处理不得不去好好考虑。

如果一个项目中图片没有处理过，完全来自设计师的切图，那么这个项目上线了体验也是极差的，它会带来一些不爽的体验例如：
1. 页面首次加载空白。部分图片因为网络请求失败加载不出来。
2. 图片加载出来了半张，剩下的半张一点点的出现
3. 页面加载一直处于loading状态 脚本加载不出来，页面卡死。

就先举这么几条，拿我来讲如果我遇到这样的站点，除非必须要用不然会毫不在意的x掉。那么我们怎么做呢？
先说下这个项目的情况，一共是18个场景，每个场景都有1个大图，还需要做一些动画处理，我们先看下没有优化前的图片文件截图:

![](https://user-gold-cdn.xitu.io/2019/9/20/16d4daab897c58b4?w=1590&h=626&f=png&s=362042)

如果这么多图片不经过处理，全部展示出来，那画面真心卡爆了，为了避免大家对我的吐槽，这块我格外用心处理，只希望人家用的时候说句，这项目真刁。

图片优化一般方法：
1. 精灵图。
2. 图片质量压缩。
精灵图有效的减少了网络的请求，图片压缩减少了图片的大小，但这远远不够呀。

放大招：

目前我们的项目是使用 sketch 做的，身为技术控的我自然也会一节基本操作，管设计要来原稿，我发现一部分是使用 线性构图 （即不是图片而是矢量图形）当我把这些导出来的时候发现每个图只有40kb左右，但也有一些500多kb，然后我就重点去看这些 500kb的图，经过一顿操作，发现这些图多参杂了bmp图片，然后我对它进行了单独的拆分。有人说为啥不找UI去帮我们做，首先UI不熟悉前端技术，它们做的不一定是我们想要的，为了做的更好，这块有能力的还是自己上手处理吧，就这样我拆出来了 3 张 bmp的图片并且转成了png格式，大小同样很小。下面这张图大家可以看到明显的改变。


也就是说在一定情况下svg是小于png的，这取决于图片的复杂程度，这里并不是色彩，主要是构成这个图的元素复杂度，如果一个人物，他的svg基本就比png大但是，如果是超清版例如分辨率 4k的呢，svg就比 png小，所以这块要灵活的去分析处理，不要让固化了我们的思维。那么我们对svg还可以进行一下处理，因为svg也是标记性语言，所以里面有很多的代码是无效的，在开发过程中，我们不必对svg进行压缩，但是在打包部署上线的时候我们最后使用处理工具处理一下svg压缩掉里面一些没用的字符。
```
rules: [{
  test: /\.(gif|png|jpe?g|svg)$/i,
  use: [
    'file-loader',
    {
      loader: 'image-webpack-loader',
      options: {
        mozjpeg: {
          progressive: true,
          quality: 65
        },
        // optipng.enabled: false will disable optipng
        optipng: {
          enabled: false,
        },
        pngquant: {
          quality: [0.65, 0.90],
          speed: 4
        },
        gifsicle: {
          interlaced: false,
        },
        // the webp option will enable WEBP
        webp: {
          quality: 75
        }
      }
    },
  ],
}]
```
在打包的配置位置里面增加 上面的 rules 就会对我们的图片资源进行一定程度的压缩，来提高我们的性能。

### webpack + svg-loader 实现 svg 精灵图

为了进一步的提高性能高,我又考虑对小的svg文件进行处理，svg列表如下

![](https://user-gold-cdn.xitu.io/2019/9/20/16d4d9dd7fac3165?w=1144&h=438&f=png&s=128107)

虽然我们拆出来了很多 svg 但是存在了一个性能问题就是我们页面加载要去请求这些文件，如果 这个svg 很小 那我们去请求这个文件，显然对于性能存在明显的浪费，于是这里我选用和雪碧图一样特性的 svg 雪碧图，通过webpack增加一个 svg-loader,下面是在 webpack中的配置 
```
{
    test: /\.svg$/,
    loader: 'svg-sprite-loader',
    include:[path.resolve('src/asset/svgSprite')],
    options:{
        symbolId:'icon-[name]'
    }
}
```
主要 include 这个路径要写清楚它用来告诉webpack 我要抽取的svg文件的位置，options 用来配置我们svg抽取之后在页面打印的名称。之后我们配置一个js文件在代码中
```
const req = require.context('./../../asset/svgSprite', false, /\.svg$/)
const requireAll = requireContext => {
    requireContext.keys().map(requireContext)
}
requireAll(req)
```
其中`require.context`是webpack中的一个方法，这时我们在启动项目的时候可以看到下图的信息

![](https://user-gold-cdn.xitu.io/2019/9/10/16d18e0e3ac92f9f?w=2226&h=468&f=jpeg&s=936301)
由svg 包裹了一系列的 symbol 标签 以及 `id="icon-xxxxx"`这样就完成了 svg文件的注入，但是这些信息都在js文件中，所以使用这种方式要注意svg文件的大小。

如何使用呢？看下面的代码,只要在用的地方使用一个use方法 就可以了然后在href里面写上我们早已弄好的id
```
  <svg  class="svgClass"  aria-hidden="true" >
    <use href="icon-xxxx" ref="svgRefs"></use>
  </svg>
```
为了让我们使用起来更加的方便我把它封装成了一个vue组件,大家看下组件的代码
```
<template>
  <svg  :class="svgClass" 
        :style="{
            'width':`${width}rem`,
            'height':`${height}rem`
        }" 
        aria-hidden="true" >
    <use :xlink:href="iconName" ref="svgRefs"></use>
  </svg>
</template>
<script lang="ts">
import Vue from 'vue';
import { Component, Prop} from 'vue-property-decorator'; 
@Component
export default class svgIcon extends Vue { 
    name:string =  'svg-icon'
    @Prop(String) iconClass ;
    @Prop(String) className ;
    @Prop(Number) width ;
    @Prop(Number) height ;
    get iconName():string {
        return `#icon-${this.iconClass}`
    }
    get svgClass(): string{
        if (this.className) {
            return 'svg-icon ' + this.className
        } else {
            return 'svg-icon'
        }
    }  
}
</script>
<style scoped>
.svg-icon {  
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```
给大家看下如何使用这个组件
```
svg-icon class="lls" icon-class="jingjingyeye" :width="4.46" :height="2.05"></svg-icon>
```
这样我们的svg文件就可以随处安放了，是不是很效率。我们的图片元素可以一瞬间的加载出来，让我们的体验有了质量的飞跃。

最终我们的优化结果看下图:
![](https://user-gold-cdn.xitu.io/2019/9/20/16d4db0ea77b61ab?w=1608&h=636&f=png&s=231251)

由 58 减少到了 15 由3.8m减少到了 437kb,当看到这个数据时候我是很有成就感的。

### node环境 fontmin 插件抽离特殊字体

我们项目中同时使用了不少漂亮的字体，为了保证和设计稿的高度还原，我这里是使用 `fontmin` node 插件来处理字体的
目录结构如下
![](https://user-gold-cdn.xitu.io/2019/9/11/16d1fcc62ffc7b7f?w=442&h=342&f=jpeg&s=59198)
源文件大小

![](https://user-gold-cdn.xitu.io/2019/9/19/16d4736246f6fde2?w=586&h=19&f=png&s=5863)

在fonts 中放入我们用到的字体包，此时他们没有经过处理大多 10多 m ，
接下来 在 font.js 中,(核心代码)
```
var Fontmin = require('fontmin');//引入插件，当第一步中，使用全局下载的插件，这里路径要注意匹配，否则后面运行时会报找不到模块的错误，所以建议使用第二种：下载到当前项目的依赖中，本文件（fontmin.js）也建在当前项目目录下
var path = require('path');
var srcPath =path.join(__dirname,'./fonts/FZBANGSXJW.TTF') ; // 字体源文件路径，需要保证该ttf文件存在
var destPath = path.join(__dirname,'./../src/asset/font/');    // 字体输出路径
var text = '奋斗路上，好好照顾自己未完成待办待审批流程PMP工时填报机会是创造出来的而不是寻觅到的';//这里进行配置需要用到的字符，可以是中文或英文字母 
// 初始化
var fontmin = new Fontmin()
    .src(srcPath)               // 输入配置
    .use(Fontmin.glyph({        // 字型提取插件
        text: text              // 所需文字
    }))
    .use(Fontmin.ttf2eot())     // eot 转换插件
    .use(Fontmin.ttf2woff())    // woff 转换插件     
    .use(Fontmin.ttf2svg())     // svg 转换插件
    .use(Fontmin.css())         // css 生成插件
    .dest(destPath);            // 输出配置
 
// 执行
fontmin.run(function (err, files, stream) {
    if (err) {                  // 异常捕捉
        console.error(err);
    }
 
    console.log('done'); 
})

```

我在每一行代码都做了注释，如果需要可以直接拿走。这样抽离之后的字体文件 就只有几kb了。下面是输入的文件

![](https://user-gold-cdn.xitu.io/2019/9/19/16d473435ca3ee67?w=1000&h=200&f=jpeg&s=102008)
通过这样的方式我们把设计稿中的文字抽离而不是用png图片代替，其好处自然多多，减少了https的请求，文字也不会有虚影，重点是小嘛。不过问题也是有的，这种方式只限于静态的文案，如果动态文案还要使用特殊字体，只能使用网络开源的字体库了。

### webpack+pwa 改造实现 离线缓存和版本控制

虽然经过了很多图片优化，我也尝试看了一下vue的同构，都是希望可以更快的把页面呈现出来，可是后台不是node，而且公司环境限制，没有办法前端自己部署node服务器，值得放弃，最终选择了pwa，可以让一部分人体验的更好,pwa的优点我这没有提案获得介绍，主要给大家说下我的使用心得吧。
 pwa 的核心在 service worker。它有 4个特点
 1. 运行在他自己的全局脚本上下文中。
 2. 不绑定到具体的网页。
 3. 无法修改网页中的元素，因为它无法访问dom。
 4. 只能使用 https。
 5. 它是事件驱动的，我们不需要掌握它的全部，只需要把我们要做的事情写好就ok了
 掌握这些特点，对我们的编码很有帮助，可以避免一些不不要的麻烦总上来说 service worker的开发成本很低，并且他和js 服务器处在不通的线程里面。而 service worker 的整个流程是：
1 注册 
2. 下载解析
3. 激活安装
4. 实现https请求的控制
也就是说 pwa 在url输入回车的一瞬间就开始执行注册，所以我们的注册代码要放在 html中,页面的最底下
```
if ('serviceWorker' in navigator) {           
    navigator.serviceWorker.register('/service-worker.js').then(function (registration) {
      // 注册成功
      console.log('ServiceWorker registration successful with scope: ', registration.scope);
    }).catch(function (err) {                   
      // 注册失败 :(
      console.log('ServiceWorker registration failed: ', err);
    });
}
```
我们看 `/service-worker.js` 是和 我的html页面平级的主要是为了对 https请求做拦截处理，我试过放在其他的地方失效了。
如何做缓存处理主要就是使用 fetch ，当发起请求的时候 可以决定如何进行，虽然简单，但是很实用


### 如何在 vue项目中使用ts

使用ts的好处我就不多说了，而且ts 的出现已经很长一段时间了 ts 语言一直是在 ts 或者 tsx 中出现，显然在 .vue中使用需要一点配置和改动，同时要配置一些插件来作为辅助


webpack中配置
```
{
    test: /\.tsx?$/,
    exclude: /node_modules/,
    use: [
      "babel-loader",
      {
        loader: "ts-loader",
        options: { appendTsxSuffixTo: [/\.vue$/] }
      },
      {
        loader: 'tslint-loader'
      }
    ]
}
```
文件根目录增加tsconfig.json文件
```
{
    "include": [
        "src/*",
        "src/**/*"
    ],
    "exclude": [
        "node_modules"
    ],
    "compilerOptions": {
        "strict": true, // 以严格模式解析
        "allowSyntheticDefaultImports": true, //允许从没有设置默认导出的模块中默认导入。这并不影响代码的显示，仅为了类型检查。
        "experimentalDecorators": true, //启用装饰器。
        "allowJs": true, //	允许编译javascript文件。
        "module": "esnext", // 采用的模块系统
        "jsx": "preserve", // 在.tsx文件里支持JSX
        "jsxFactory": "h",  // 使用的JSX工厂函数
        "target": "es2015", // 编译输出目标 ES 版本
        "noImplicitAny": false, // 在表达式和声明上有隐含的any类型时报错
        "moduleResolution": "node", //决定如何处理模块
        "isolatedModules": true, //将每个文件作为单独的模块
        "lib": [
          "dom",
          "es5",
          "es6",
          "es7",
          "es2015.promise"
        ],
        "sourceMap": true, //把 ts 文件编译成 js 文件的时候，同时生成对应的 map 文件
        "pretty": true //给错误和消息设置样式，使用颜色和上下文。
    }
}

```
入口文件目录内也就是src内部增加 vue-shim.d.ts 
```
declare module "*.vue" {
    import Vue from "vue";
    export default Vue;
}
```
配置完这些我们就可以在项目中使用ts 语言了
```
// ts + vue 辅助工具
   import { Component, Prop, Emit, Watch } from 'vue-property-decorator'
   // 接口声明
   interface list {
            year:string;
            month:string;          
            startDate:string;
            endDate:string;
    }
    // component 很重要 一定要加上不然会报错
    @Component
    export default class App extends Vue {   
        // 相当于 data
        weekList:list[]=[]
```

### 动画实现经验总结


搞定了图片的优化，接下来就是处理动画部分了。这个项目主要就是动画，主要表现就是来自于本人的想象，我们主要，聊下这些动画是如何实现的。虽然动画的构成很简单 只有 transition animation 这两个属性。但是往往对于一个动画如何实现，让我们十分的困扰。接下来我想把我写动画的心得分享给大家。

#### 1. 时间函数调试
我的动画调试工具是chrome主要用到的就是它的时间函数自动生成功能，我自己叫它时间函数生成器

![](https://user-gold-cdn.xitu.io/2019/9/23/16d5bc15c9526896?w=544&h=706&f=jpeg&s=63056)

我们可以拖动它的两个可以移动的点就可以看到时间函数曲线，如果我们管左边的叫p1 ，右边的叫p2 那么它最后生成的函数对应坐标点是 cubic-bezier(x1,y1,x2,y2),我们通过这个工具就可以很直观的看到随着时间我们元素的变化规律，这个主要方便我们再有AE设计稿时候按照设计稿中的曲线去高度还原AE动画。也方便了我们自己去描述一个元素的运动规律，例如自由落体，弹性碰撞。

如果在js脚本中去控制 也可以通过 tween.js 这类的时间函数插件去控制，原理是一样的。

#### 2.多帧的书写方式让动画更细腻

往往大家在使用 动画属性时候大多都是这样用：
```
animation : 属性 过渡时间 时间函数 延迟;
```
就结束了，其实它本质上代表的是一个关键帧动画，而这样大多数的动画表现形式比较生硬，大家看到比较好看的动画往往是多个关键帧完成的，这样的表现形式也更加的细腻,例如我们可以这样写
```
animation : 属性 过渡时间 时间函数 延迟,属性 过渡时间 时间函数 延迟,属性 过渡时间 时间函数 延迟,...;
```
每一个 `,` 代表一个关键帧可以按照需要无限的写下去。大家可以看这个例子,它的延迟时间是关键
```
.demo-11{
  animation: runlef 1s ease,runTop 3s ease 1s;
}
@keyframes runlef{
    to{
        transform: translateX(300px);
    }
}
@keyframes runTop{
    from{
        transform: translate(300px,0);
    }
    to{
    
        transform: translate(300px,-300px);
    }
}
```
这样有一个问题就就是前面帧的动画只能只能执行一次，不能无限的重复而且我发现一个有意思事情就是当我们使用下面这种方式
```
animation-name:runTop,runlef;
```
表面上先执行`runTop` 然后执行 `runlef` 事实上它的执行顺序是由右向左的，所以这样的方式我不推荐大家在动画中使用，不要理解，也不好维护。


#### 3. 静态属性的配置

我们在做动画中为了减少GPU内存的占用，在静态属性的配置上也需要做一些文章，例如不要用`all`，少用 `margin-top`这一类可以引起整个文当流重新变化的属性，如果用多了这类的属性，会造成动画的卡顿，抖动，等不正常的现象，更有甚者可以造成整哥页面的崩溃。

举个例子，例如我们要实现宽度高度的一起变化，即多个属性同时执行可以这样
```
.demo-11{
    transition: width  1s ease, height 1s ease; 
    &:hover{
        width: 200px;
        height: 300px;
    }
}
```
这样的好处是避免了使用 `all` ，因为我们一个元素的静态属性太多了，`all` 看似省时省力但是它会对所有的属性进行监听，分开写也可以去单独的属性进行控制，有人已经发现了 它和我们的 animaion 的连写类似，但是它的功能是完全不一样的，我们要区分开,transition  因为功能的现在只能针对一个关键帧，即元素的属性一次变化之后，如果要实现多个关键帧的变化虽然使用它+js脚本控制也可实现，但是如果我们使用 `animation` 不是更好么。

这个例子我们是变化的 `width` 和 `height` 如果你把这哥代码片段拿过去用会发现当它的属性改变时候，会让旁边的元素跟着一起动，如果这是一个页面显然是不理想的，所以我们涉及到动画的地方尽量给它做 **脱离文档流** 处理 ，最简单的方法就是

```
position:absolute; //动画元素做脱离文档流处理
```
如果改变元素的位置等状态我推荐使用 `transform`。
我们在一些场景里面需要对文字放大，这里我的经验就是 少用 `scale` 它总是在过渡的过程中造成文字的抖动 和模糊，体验很差，虽然网上有不少解决的方案，但是在一些浏览器中是无效的，为了减少麻烦大家少用。
那话说回来了`scale`我们就不用了吗？当然任何属性都是有用的，我们可以用它做图片的放大缩小，这里同样需要一个小技巧，我这里总结给大家
* 如果是图片放大效果，我就是准备一个大图，用`scale`把图片缩小了，使其放大后是原图的大小就可以啦，这样可以避免图片虚化和抖动，反之亦然。

#### 4. 特殊的css静态属性

1. 一些生僻的属性例如 ` offset-path` 这类的，因为它可以让我们的动画元素按照指定的线路去运动。
2. `clip-path` 裁剪区域，可以自由的控制图片展示的形状，我们如果通过一些path生成工具就可以实现一个海岸的波纹效果， 也可以实现一个元素的边缘扭曲效果。
3. `filter` 也是一个动画常用的属性，我们如今经常使用`opicity`,它也可以处理图片的透明，配合动画可以做一些很库的例子。我们最常用的是很多CSS的效果都是`filter+mix-blend-mode`，例如我们最常见的水滴滴入一个大的元素的融合效果就是这样做的
```
filter: blur(5px) contrast(20);
```
这些我们不怎么用的属性往往在动画的实现上,却能给我们眼前一亮的感觉。

在例如outline我们经常用它来去掉我们button的外面的虚框 其实它也可以做动画，经过几次调试 在单击这个元素就变成了一个➕号
```
.add{   
    width:100px;
    height:100px;
    outline: 20px solid #000;
    outline-offset: 10px;
    background: #009fff;
    transition:   outline-offset 1s;
     &:hover{
         outline-offset: -68px; //变成了➕号
     }
}

```


#### 5. 掌握基本的svg动画元素

2d动画我觉得最优秀的语言就是svg了，它比canvas学习成本底，操作简单，首先给大家普及下 svg 的动画元素

1. animate  改变svg元素的属性
```
<rect x="10" y="10" width="200" height="20" stroke="black" fill="none"> 
<animate attributeName="width" attributeType="XML" from="200" to="20" begin="0s" dur="5s" fill="freeze" /> </rect>

```

2. animateMotion 路径动画，它和 css3 里面的` offset-path`相类似
```
  <animateMotion
                 xlink:href="#ant"
                 path="M0,0 Q50,60 80,140 T340,100"
                 dur="10s"
                 begin="click"
                 fill="freeze">
  </animateMotion> 
```
通过传入一个 path 在 `click`事件触发之后就可以沿着这个路径进行了 
3. animateTransform 是对 transform属性变化的属性 和 `animate` 类似

接下来我们具体看下这三个属性可以配置的一些通用参数:

* `attributetype` svg 的类型可以填 `css` 或者 `xml`
*  `attributeName` 要变化的属性名称
*  `from to` 也可以用 ` values="0;.2;0;.3;0;.1;0;.4;0"` 可以使用`;` 就是 `to` 的意思 。
* `fill` 和 css3 的最后状态类似.
  * freeze：动画结束以后，动画保持最后状态。
  * remove：动画结束之后，恢复到初始状态。
* `begin` 控制动画如何开始或者何时开始可以接受一个事件或者一个时间长度。
* `dur` 动画的时间长度
* `repeatCount` 重复的次数
* `calcMode` 和css3 的时间函数相似
掌握这些基本的动画属性，我们可以对svg进行动画的补间了，至于 svg其它的静态属性，太多了，我们用的时候最好通过工具去查下。

svg 还可以通过 css属性像操作 html标签一样，看下面的例子
```
 <style type="text/css" >
    <![CDATA[      
      ....
    ]]>    
    </style>
```
这是在svg文件中中使用css语言的基本格式，当然如果把直接嵌入到页面中就可以和正常的写法一样了，也就是说css的静态属性同样可以操作svg。例如我们控制一条直线从 0 到 300px
```
<svg width="300" height="300">
    <line x1="0" y1="300" x2="300" y2="300" stroke="skyblue"  id="Line"></line>
</svg>
```
给大家翻译下上面的意思 一条直线 开始坐标 (0,300)到 (300,300) 也就是水平方向的一条直线，我们下面用css控制下它
```
#Line {
  stroke-dasharray: 300px;
  animation: drawwww 1s linear forwards;
  stroke-width: 5px;        
}
@keyframes drawwww {
  from{
    stroke-dashoffset: 300px;
  }      
  to {
    stroke-dashoffset: 0;
  }
}
```
我给大家解释下，这里用到的svg属性是描边 `stroke-dasharray` 是用来创建虚线的正常来写是这样的
```
 stroke-dasharray: 30px,10px; // 30 长度，10间隔
```
` stroke-dashoffset` 代表的是偏移 通过改变它就会形成一个看似直线慢慢身长的动画，其实直线只是偏移到了我们看不到的地方,它通常可以做一个漂亮的按钮 hover 效果。书归正题，我继续说下svg下的使用心得，在一些路径生成软件下我们生成的路径是机器编号，这时候我们使用 css直接操作 svg 是有问题的，这时候你就需要对svg的代码进行改造
```
 <path d="..." id="Z-Copy-3" fill="#0C2FFF" transform="translate(190.738572, 169.030000) scale(-1, 1) translate(-190.738572, -169.030000) ">
```
上面是项目中的一段 svg 代码 我把 d 的内容省去了。我们重点看下`transform ` 这里有一些生成的属性 如果我们直接使用 css 操作 这个图片是倒着运动的，所以就要对它进行改造  `scale(-1, 1)` 实际上就是 x轴 反转，我们把它去掉 才行，所以 在使用 svg 文件时候要看看动画是不是我们想要的，以及可以从代码中找到问题。
这些都是在svg中经常遇到的问题。

####  6.常用的动画变化方案
路径变化：
涉及路径变化大部分都要用到svg了，而svg路径的生成我们可以手写也可以用工具生成
接下来我想给大家举例几个常用的静态属性，配合动画元素实现我们想要的效果。

##### 元素做曲线运动offset 

```
{
//这里定义一个运动的路径
    offset-path: path("M0,0a72.5,72.5 0 1,0 145,0a72.5,72.5 0 1,0 -145,0");
    animation: load 1.5s ease-in-out infinite;
}
@keyframes load {
    from {
        offset-distance: 0;
    }
    to {
        offset-distance: 100%;
    }
}
```

