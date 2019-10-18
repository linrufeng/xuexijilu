我们团队（京东用户体验设计部-前端开发部）近期发布了移动端Vue组件库NutUI的2.0版，2.0不是1.0的升级，而是一个全新的组件库。从1.0到2.0一路走来，我们积累了一些Vue组件库的开发经验，接下来的一段时间，我们将以系列文章的方式与大家进行分享，欢迎大家关注。

作为开篇之作，本篇文章我们就从盘古开天地说起吧。

从当年的静态页面到如今的Web App，前端工程越来越复杂，对于一个稍大些的前端项目来说，代码都写在一起难以维护，团队分工协作也成问题。根据软件工程领域的经验，解决这些问题的一个可行思路就是代码的模块化，即对代码按功能模块进行分拆，封装成组件，而反过来讲，组件就是指能完成某个特定功能的独立的、可重用的代码块。

把一个大的应用分解成若干小的组件，而每个组件只需要关注于某个小范围的特定功能，但是把组件组合起来，就能构成一个功能庞大的应用。组件化的网页开发也是如此，就像搭积木，各个组件拼接在一起就组成了一个完整的页面。

组件化开发可大大降低代码耦合度、提高代码的可维护性和开发效率，同时有利于团队分工协作和降低开发成本。这种开发模式已日渐流行起来。

当前，前端开发领域最流行的三大框架Vue、React、Angular都推崇组件化开发，组件是这些框架中极为重要的概念和功能。

以Vue.js来说，组件 (Component) 可以说是其最强大的功能，它可以扩展HTML元素，封装可重用的代码。Vue.js的组件系统让我们可以用这些独立可复用的小组件来构建大型Vue应用，几乎任意类型的Vue应用的界面都可以抽象为一个组件树。

如果我们把日常应用开发中常用的组件累积起来，后续的项目就可以复用这些组件，这对提高开发效率、降低开发成本有重要意义。

因此，一个前端团队，拥有一个常用框架的组件库是十分必要的。

## 模块化与构建工具

组件库自身就是一个大的工程，需要按照模块化开发思想进行模块划分。通常，在一个组件库里，组件、组件的样式文件、配置文件…都是模块，而最终我们需要把这些模块组合成一个完整的组件库文件，承担这种组装工作的就是打包构建工具。当下主流的库构建工具主要有Rollup和Webpack等。在说这些模块打包构建工具之前，我们先来了解一下目前主流的JavaScript模块化方案。

JavaScript语言一直以来饱受诟病的一个地方就是它的语言标准里没有模块（module）体系，这对开发大型的、复杂的项目形成了巨大障碍。直到ES6时期，才在语言标准层面实现模块功能（ES6 Module）。在ES6之前，社区制定了一些模块加载方案，如CommonJS 和 AMD 。ES6 Module作为官方规范，且浏览器端和服务器端通用，未来一定会一统天下，但由于ES6 Module来的太晚，受限于兼容性等因素，可以预见的是今后一段时期内，多种模块化方案会共存。

* ES6 Modue 规范： JavaScript语言标准模块化方案，浏览器和服务器通用，模块功能主要由export和import两个命令构成。export用于定义模块的对外接口，import用于输入其他模块提供的功能。
* CommonJS 规范：主要用于服务端的JavaScript模块化方案，Node.js采用的就是这种方案，所以各种Node.js环境的前端构建工具都支持该规范。CommonJS规范规定通过require命令加载其他模块，通过module.exports 或者exports对外暴露接口。
* AMD 规范：全称是Asynchronous Modules Definition，异步模块定义规范，一种更主要用于浏览器端的JavaScript模块化方案，该方案的代表实现者是RequireJS，通过define方法定义模块，通过require方法加载模块。

一些“上年纪”的国内前端老艺人们可能还会提到CMD规范，它是 SeaJS 在推广过程中对模块定义的规范化产出，只是SeaJS并未实现国际化，且在2015年项目就已宣布停止维护了，算不上当前主流模块化方案。

介绍完主流模块化规范，我们再回过头来看Rollup和Webpack这两个模块打包构建工具。

Rollup是一个知名的库打包工具，很多知名的库、框架都是使用它打的包，包括Vue和React自身。Rollup可以直接对ES6模块进行打包，它率先提出并实现了Tree-shaking功能，即在打包时静态分析ES6模块代码中的 import，排除未实际使用的代码，这有助于减小构建包的体积。

另一个打包工具Webpack名气更大，不过我们通常用它来打包应用，而它对库打包也能提供很好的支持。Webpack支持代码分割、模块的热更新（HMR）,这让它看起来非常适合打包应用。而Webpack 2及后续版本陆续增加了对ES6模块、Tree-shaking、Scope Hoisting的支持，大大增强了其库打包能力。

如今，Rollup在库打包方面的优势已不再那么明显，而在对应用打包的支持方面却明显落后于Webpack。所以打包应用推荐使用Webpack，而打包库Rollup和Webpack基本都能胜任。

那么我们在开发NutUI 2的时候为什么选择了Webpack而不是Rollup呢？其实主要还是上述这个原因，按照规划，NutUI的官网（包含示例和文档）与库在同一个项目中，因此我们需要一个既能打包库，又能打包应用的工具，Webpack显然更适合。

## Webpack打包library

使用Webpack来打包应用，相信大多前端小伙伴都不会感到陌生。而如何使用Webpack来打包一个组件库呢？我们继续往下说。

首先，虽然基于ES6模块规范开发，但考虑到浏览器兼容性，我们需要打包出来的组件库能兼容AMD等浏览器端模块规范。同时，为了使组件库能支持服务端渲染（SSR）等场景，它还需要支持commonJS规范。此外，还有一种常见的库使用场景，即在页面上直接通过script标签引入，也就是非模块化环境同样需要兼容。

Webpack中，output.libraryTarget选项用来配置如何暴露库，可配置以commonJS模块、AMD模块，甚至全局变量形式暴露库。可是如何让这个库可以同时兼容commonJS、AMD和全局变量呢？

所幸，这个选项还支持一个可选值——umd。UMD（Universal Module Definition，通用模块规范）可以同时支持CommonJS和AMD规范，以及非模块化引用。

综上，我们需要把output.libraryTarget选项的值设为“umd”。

另外两个与库打包关系密切的Webpack配置项如下：

* output.library，对外暴露的变量名或模块名，具体作用与output.libraryTarget选项的值有关。
* output.umdNamedDefine，当output.libraryTarget的值为“umd”时，设置该选项的值为true会对 UMD 的构建过程中的AMD模块进行命名，否则就使用匿名的 define，匿名的AMD模块。

这几个选项配置完，就可以打包出一个基于umd规范库了。

``` js
output: {
        path: path.resolve(__dirname, '../dist/'),
        filename: 'nutui.js',
        library: 'nutui',
        libraryTarget: 'umd',
        umdNamedDefine: true
}
```

但是我们会发现构建出来的库在Node.js环境使用时会报错：

``` shell
WebpackError: ReferenceError: window is not defined
```

是不是感到莫名其妙？说好的UMD兼容commonJS呢？查看Webpack构建出的包代码，我们会发现，UMD部分的代码里的全局对象竟然是window，非浏览器环境哪有window对象，Node.js中不报错才怪。

``` js
(function webpackUniversalModuleDefinition(root, factory) {
 if(typeof exports === 'object' && typeof module === 'object')
 module.exports = factory(require("vue"));
 else if(typeof define === 'function' && define.amd)
 define("nutui", ["vue"], factory);
 else if(typeof exports === 'object')
 exports["nutui"] = factory(require("vue"));
 else
    root["nutui"] = factory(root["Vue"]);
})(window, function(__WEBPACK_EXTERNAL_MODULE__2__) {
```

查阅Webpack文档，可以发现output对象还有一个属性叫globalObject，用来指定挂载这个库的全局对象，默认值是window。而文档明确指出，当构建UMD包需要兼容浏览器和Node.js环境时，值应该设为this。

``` js
output: {
        path: path.resolve(__dirname, '../dist/'),
        filename: 'nutui.js',
        library: 'nutui',
        libraryTarget: 'umd',
        umdNamedDefine: true,
        globalObject: 'this'
}
```

我们将globalObject设置为'this'后，构建出来的包中UMD部分的window被替换为了this，这样在Node.js环境就不会再报上面那个错了，这对实现组件库兼容服务端渲染功能来说非常重要。

``` js
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory(require("vue"));
	else if(typeof define === 'function' && define.amd)
		define("nutui", ["vue"], factory);
	else if(typeof exports === 'object')
		exports["nutui"] = factory(require("vue"));
	else
		root["nutui"] = factory(root["Vue"]);
})(this, function(__WEBPACK_EXTERNAL_MODULE__2__) {
```

这里吐个槽，个人感觉Webpack这部分设计欠妥，当libraryTarget值为umd时globalObject默认值应该为this，而不能是window，否则umd还有何意义？至少在文档中libraryTarget: 'umd'部分对此问题应该有所提及。不然，还会有不少人踩此坑。

## 外部依赖Vue.js

Vue组件库不需要打包Vue.js，可在运行时从外部获取。Webpack中可以通过externals进行配置外部依赖。以jQuery为例看下externals的配置方法：

``` js
externals: {
    jquery: 'jQuery'
}
```

这样jquery在构建时不会打到包内，而是在运行时去外部环境寻找jQuery这个模块（或属性）。照猫画虎，依葫芦画瓢，我们不需要打包Vue.js，那我们就这么写：

``` js
externals: {
    vue: 'vue'
}
```

这时候构建出来的包在各种模块化场景使用都没毛病，可唯独在非模块化场景会报错：

``` js
vue is not defined
```

这是为什么呢？我们先来看下Vue.js的部分源码：

``` js
/*!
 * Vue.js v2.6.10
 * (c) 2014-2019 Evan You
 * Released under the MIT License.
 */
(function (global, factory) {
    typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
        typeof define === 'function' && define.amd ? define(factory) :
            (global = global || self, global.Vue = factory());
```

从上面的Vue.js源码中，我们可以看到挂到全局对象上的属性名称是首字母大写的Vue，而其NPM包名却是小写的vue，也就是说不同环境下Vue名称不尽一致，这可如何是好？

``` json
{
  "name": "vue",
  "version": "2.6.10",
```

还好，externals中属性的值除了字符串，还支持传一个对象，可针对各种场景单独设置模块名（或属性名），这样一来，我们就可以为非模块化环境配置'Vue'，为模块化环境配置'vue'。

``` js
externals: {
        'vue': {
            root: 'Vue',
            commonjs: 'vue',
            commonjs2: 'vue',
            amd: 'vue'
        }
}
```

Vue.js就是这样被设置为组件库外部依赖的。

## Tree-shaking（摇树）

如前文所述，Tree-shaking功能最早由Rollup提出并实现，曾是Rollup的杀手锏，后来Webpack等工具把它“借鉴”走了。

Tree-shaking的原理是在打包时通过对代码进行静态分析将未使用的代码排除，从而减小包体积。对JavaScript进行静态分析，这在之前是不可能的。直到 ES6 模块化方案提出，才使得JavaScript静态分析成为可能，因为 ES6 模块是编译时加载，不用等到代码运行时，就可以知道加载了哪些模块。因此要使用Tree-shaking功能，就需要在代码中使用ES6模块方案，不管是用Rollup还是Webpack打包。

还有一个影响Tree-shaking施展的可能是Babel在Webpack开始“摇”之前把你的 ES6 模块转成了commonJS模块，那就“摇”不了了。这种情况并不罕见，大部分前端开发者都乐于使用新语法，所以不止模块化方案要用ES6 Module，甚至整个项目的JavaScript代码都恨不得用ES6+语法来写，为了兼容低版本环境，通常会使用Babel等工具把ES6+语法转成ES5语法。这当然没问题，只是如果想让Tree-shaking发挥作用，让我们构建出来的包体积更小，一定要注意，不要让Babel把ES6模块语法转成commonJS，Rollup和较新版本的Webpack都支持直接处理ES6模块，可以也应该把ES6模块部分直接交给它们来处理。Babel不处理ES6模块，并不意味着最终打出来的包就是ES6模块，如前文所述，构建出来的包如何暴露，要兼容哪些模块规范打包工具就能搞定。

``` js
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "modules": false
      }
    ]
  ]
}
```

我们测试了一下，Tree-shaking让NutUI 2.0的完整版的构建文件体积明显减小。