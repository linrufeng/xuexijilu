一、什么是同构
同构是指同开发一个可以跑在不同的平台上的程序。例如开发一段 js 代码可以同时被基于 node.js 开发的 web server 和浏览器使用。本文中我们就要聊聊这种场景下，为什么以及怎么样开发一个同构的 web 应用。

二、同构带来的好处
我们不会平白无故地做出任何决策，大家使用同构肯定是因为同构能够带来一些好处：

减少代码开发量, 提高代码复用量。因为一份代码能同时跑在浏览器和服务器，因此不仅代码量减少了，而且很多业务逻辑不需要在浏览器和服务端两边同时维护，因而同时减小了程序出错的可能。
可以以较小的成本完成 SSR （Server-Side Render）的功能。而 SSR 能带来至少以下两点好处。
首屏性能，让用户更早看到页面内容。
SEO (Search Engine Optimization), 对爬虫友好。
三、同构带来的问题
性能损失，客户端服务端都要渲染页面, 存在一定的性能浪费（可以通过客户端 dom 反收集和 virtual-dom 等手段尽量优化，但不可避免）。
一个可以同构的模块必须同时兼容客户端和 Node.js 环境，因此会带来额外的一些开发成本。特别是习惯客户端开发的人要注意 window，document，DOM 等是客户端才存在的对象。
服务端内存溢出的风险，客户端代码运行环境随着浏览器刷新会重新建立，因此不需要太注意内存溢出的问题，而服务端则不同。
要特别注意异步操作，习惯于客户端开发的同学可能很习惯在前端随意发起异步数据请求和操作，因为所有的操作都会引起页面重绘。而服务端则不同，服务端的组件只能调用一次或有限次的 render，所以所有用于服务端渲染的异步请求必须全部都调用 render 返回 html 前完成。
所有在服务端预取的状态都应该有途径能让客户端获取，以免客户端和服务端渲染结果不同导致闪屏。因为无论如何客户端都会渲染一次页面，若服务端用来渲染的数据和客户端不一样，那么渲染出来的 dom 也会不一样，导致闪屏。
四、应用的哪些部分可以同构
单页应用的路由可以被同构，这样访问任意单页应用的子页面都可以享受 SSR 带来的好处。
模板，前后端共用一个渲染引擎就可以做到前后端共用模板，这样类似于因同一份数据要用于前后端渲染而需要开发两套模板的日子就一去不复返了。
数据请求，开发支持同构的 httpClient，那么前后端请求数据的代码也可以同构了。需要注意的是服务端没有 cookie，因此会话相关的请求代码需要极其小心。
其他平台不相关的代码，例如 react 和 vue 都有的全局状态管理模块、数据处理过程和一些平台无关的纯函数。
五、哪些东西不能同构
平台相关代码，如只能在浏览器端执行 DOM、BOM 相关的操作，只能在服务端执行文件读写，数据库操作等。
六、我们到底需不需要同构
6.1 同构带来的好处可以通过别的途径来获取吗？
SSR

SSR 当然不是必须通过同构来实现的，但使用同构来实现 SSR 可以减少大量重复的代码开发。

减少因为前后端使用两份代码同时维护一份逻辑而出错的可能性

我没有想到比同构能更好地解决这个问题的方案了。

在 SSR 是必需的时候，我感觉同构还是有必要的。

6.2 支持同构，但不滥用 SSR
因此我觉得一个比较良好的方案是开发一个支持同构的应用，但不强制使用 SSR，因为 SSR 带来一定的性能浪费。

支持同构，一份代码即可以跑在客户端又可以跑在服务端，但具体要不要在服务端跑这段代码由具体业务来决定。
仅在首屏性能需求高和有 SEO 需求时使用 SSR，其他情况使用单纯的客户端渲染，这看起来是一个比较好的折中方案。
七、从零开始写一个支持同构的多页应用
例子代码仓库地址。
启动例子的同时阅读下面的段落体验会更好。例子中使用了 express 作为 web server 框架，因此读者若有一些 express 基础会更容易理解例子。

7.1 前后端代码的职责
前端：[（单页应用）处理路由-> ] 请求数据 -> 渲染 -> 绑定事件
后端：[ 处理路由 -> ] 请求数据 -> 渲染
7.2 前后端代码作用的差异
前端：没有输出，代码直接作用于页面元素
后端：输出 html 字符串
7.3 判断代码执行环境
最简单的方式就是通过 window 对象的存在与否来判断当前的代码执行环境，只有在浏览器执行环境 window 对象才存在

    const isBrowser = typeof window !== 'undefined';
7.4 同构应用基本设计
基本设计

7.5 同构的组件基类
7.5.1 生命周期规划
一个同构的组件，它的生命周期在服务端和客户端的执行情况是不同的。在 mount 操作前的生命周期可以跑在服务端。

客户端：
beforeMount -> render -> mounted
服务端：
preFetch -> beforeMount -> render
beforeMount 和 render 生命周期 在服务端和客户端都会被执行，因此这两个生命周期内也不应该写平台相关的代码。

下面是这次 demo 使用的同构组件基类：

// ./lib/Component.js
const {isBrowser} = require('../utils');

module.exports = class Component {
    constructor (props = {}, options) {
        this.props = props;
        this.options = options;
        this.beforeMount();
        if (isBrowser) {
            // 浏览器端才执行的生命周期
            this.options.mount.innerHTML = this.render();
            this.mounted();
            this.bind();
        }
    }
    // 生命周期
    async preFetch() {}
    // 生命周期
    beforeMount() {}
    // 生命周期
    mounted() {}
    // 绑定事件时使用
    bind() {}
    // 重新渲染时调用
    setState() {
        this.options.mount.innerHTML = this.render();
        this.bind();
    }
    render() {
        return '';
    }
};
所有的业务组件都继承这个基类，例如一个实际业务组件如下：

// ./pages/index.js
const Component = require('../lib/Component');

module.exports = class Index extends Component {
    render() {
        return `
            <h1>我是首页</h1>
            <a href="/list">列表页</a>
        `;
    }
}
启动例子后可以访问 http://localhost:3000/ 来访问这个页面，可以观察一下 SSR 的情况。

7.6 服务端处理
7.6.1 使用 ServerRenderer 来渲染组件
一个简单的 ServerRenderer 实现如下：

// ./lib/ServerRenderer.js
const path = require('path');
const fs = require('fs');

module.exports = async (mod) => {
    // 获取组件
    const Component = require(path.resolve(__dirname, '../', mod));
    // 获取页面模板
    const template = fs.readFileSync(path.resolve(__dirname, '../index.html'), 'utf8');
    // 初始化业务组件
    const com = new Component()
    // 数据预取
    await com.preFetch();
    // 将组件渲染的字符串输出到页面模板
    return template.replace(
        '<!-- ssr -->', 
        com.render() +
            // 把后端获取的数据放到全局变量中供前端代码初始化
            '<script>window.__initial_props__ = ' + 
            JSON.stringify(com.props) +
            '</script>'
    )
    // 替换插入静态资源标签
    .replace('${modName}', mod);
}
7.6.2 页面模板
这次 demo 的所有页面都使用同一份 html 模板:

<!-- ./index.html -->
<html>
    <head>
        <title>test</title>
    </head>
    <body>
        <div id="app">
            <!-- 下面是 ssr 渲染后内容填充的占位符 -->
            <!-- ssr -->
        </div>
        <!-- 插入实际业务的前端 js 代码 -->
        <script src="http://localhost:9000/build/${modName}"></script>
    </body>
</html>
7.6.3 服务端路由和 controller
此次的 demo 是基于 express 框架开发的，下面的代码使用 ServerRenderer 渲染同构的组件，然后输出 页面 html 给浏览器。

// ./routes/index.js
var express = require('express');
var router = express.Router();
var ServerRenderer = require('../lib/ServerRenderer');

router.get('/', function(req, res, next) {
  ServerRenderer('pages/index.js').then((html) => {
    res.send(html); 
  });
}); 
7.7 客户端处理器 ClientLoader
demo 中的 ClientLoader 是一个 webpack loader,该 loader 代码如下：

// ./lib/ClientLoader.js
module.exports = function(source) {
    return `
        ${source}
        // 入口文件 export 的是主组件
        const Com = module.exports;
        // 获取后端渲染时使用的初始状态 window.__initial_props__，保证前后端渲染结果一致。
        new Com(window.__initial_props__, {
            mount: document.querySelector('#app')
        });
    `;
};
在 webpack.config.js 中使用这个插件（仅作用于页面入口组件）

// webpack.config.js
module.exports = {
    ...,
    module: {
        rules: [
        ...,
            {
                test: /pages\/.+\.js$/,
                use: [
                    {loader: path.resolve(__dirname, './lib/ClientLoader.js')}
                ]
            }
        ],
    }
}；
7.8 一个业务组件
有了以上的基础以后，我们可以轻易的写一个支持同构的组件。下面是一个列表页。

7.8.1 代码
// ./routes/index.js
/* GET list page. */
router.get('/list', function(req, res, next) {
  ServerRenderer('pages/list.js').then((html) => {
    res.send(html); 
  });
});

// ./pages/list.js
const Component = require('../lib/Component');
const {
    getList,
    addToList
} = require('../api/list.api');

module.exports = class Index extends Component {
    constructor (props, options) {
        super(props, options);
    }
    // 服务端执行，预取列表数据
    async preFetch() {
        await this.getList();
    }
    async getList() {
        const list = (await getList()).data;
        this.props.list = list;
    }
    ...,
    render() {
        return `
            <h1>我是列表页</h1>
            <button class="add-btn">add</button>
            <button class="save-btn">save</button>
            <ul>
                ${
                    this.props.list.length ? 
                    this.props.list.map((val, index) => `
                        <li>
                            ${val.name}
                            <button class="del-btn">删除</button>
                        </li>
                    `).join('') :
                    '列表为空'
                }
            </ul>
        `;
    }
}
7.8.2 服务端渲染结果
若已启动 demo 服务器，访问 http://localhost:3000/list 可以看到服务端的渲染结果如下。window.__initial_props__ 的存在保证前后端渲染的结果一致。

<html>
    <head>
        <title>test</title>
    </head>
    <body>
        <div id="app">
            <h1>我是列表页</h1>
            <button class="add-btn">add</button>
            <button class="save-btn">save</button>
            <ul>
                <!-- 服务端渲染预取的列表数据 -->
                <li>
                    1
                    <button class="del-btn">删除</button>
                </li>
                <li>
                    2
                    <button class="del-btn">删除</button>
                </li>
                <li>
                    3
                    <button class="del-btn">删除</button>
                </li>
                <li>
                    4
                    <button class="del-btn">删除</button>
                </li>
            </ul>
            <script>
                // 预取的列表数据, 用来客户端渲染
                // 客户端第一次渲染和服务端渲染结果相同，因此用户看不到客户端渲染的效果。
                window.__initial_props__ = {
                    "list": [{
                        "name": 1
                    }, {
                        "name": 2
                    }, {
                        "name": 3
                    }, {
                        "name": 4
                    }]
                }
            </script>
        </div>
        <script src="http://localhost:9000/build/pages/list.js"></script>
    </body>
</html>
八、一个单页面同构应用。
比起多页面应用，单页面应用需要多同构前端路由的部分。

8.1 服务端处理
初始化组件时带上路由信息:

// ./lib/ServerRenderer.js
module.exports = async (mod, url) => {
    ...
    // 初始化业务组件
    const com = new Component({
        url
    });
    ...
}
书写 controller 时把路由输入 ServerRenderer：

/* GET single page. */
router.get('/single/:type', function(req, res, next) {
  ServerRender('pages/single.js', req.url).then((html) => {
    res.send(html); 
  });
});
8.2 客户端代码
下面是一个单页应用组件，点击切换按钮就可以纯前端的切换路由并改变视图：

// ./pages/single.js
const Component = require('../lib/Component');

module.exports = class Index extends Component {
    switchUrl() {
        const isYou = this.props.url === '/single/you';
        const newUrl = `/single/${isYou ? 'me' : 'you'}`;
        this.props.url = newUrl;
        window.history.pushState({}, 'hahha', newUrl);
        this.setState();
    }
    bind() {
        this.options.mount.getElementsByClassName('switch-btn')[0].onclick = this.switchUrl.bind(this);
    }
    render() {
        ;
        return `
            <h1>${this.props.url}</h1>
            <button class="switch-btn">切换</button>
        `;
    }
}
访问 /single/you 服务端返回的内容为：

<html>
    <head>
        <title>test</title>
    </head>
    <body>
        <div id="app">
            <h1>/single/you</h1>
            <button class="switch-btn">切换</button>
            <script>
                window.__initial_props__ = {
                    "url": "/single/you"
                }
            </script>
        </div>
        <script src="http://localhost:9000/build/pages/single.js"></script>
    </body>
</html>
九、公共状态管理的同构
公共状态管理的同构和组件的 props 同构其实非常类似，都需要把后端预取数据以后的整棵状态树渲染到页面上然后前端初始化状态管理器 store 的时候使用这棵树来做为初始状态，以此来保证前端渲染的结果和后端一致。

十、特殊的 HttpClient
上面使用的 demo 中使用的 httpClient 是 axios，这个库本身就已经支持同构。但还是有一个问题，这个问题之前也提到过。
当涉及到会话相关的请求时，一般情况下浏览器发送请求时会带上 cookie 信息，但是服务端发起的请求并不会。因此，服务端发起请求时，需要手动地把 cookie 加到请求头中去。