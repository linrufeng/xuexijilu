Vue构建工具的设计与实现

原创： 汪楠  全栈探索  今天
简介：想了解Vue技术栈构建工具设计思路吗？一个简易好用的构建工具都是为了提高生产效率，用工具解放生产力，关注业务开发。

一、前言

在前端界 React、 Vue、 Angular三足鼎立的形式下，我们也针对 Vue 技术栈设计了一套构建工具解决方案。力求做到开箱即用，又可以根据实际的项目需求插件式的配置项目模版工程，包含开发、调试、上线整个流程的功能。本文主要讲一下整体的设计思路。

一般 Vue2.0技术栈的构建工具需要具备以下几个功能：

脚手架工具
Webpack模版
核心组件库
辅助功能
二、脚手架工具

从 Vue-cli 的整体设计可以学习到cli构建工具和模版相分离的设计思路，我们可以通过命令行，选择匹配不同的模版下载，用于不同的开发需求。分开维护的好处是，模版的变动更新，并不需要用户端重新发布和安装cli构建工具，并且也保证了一致性，当我们有发现模版bug，修复后，也能及时同步远端。

如上图所示，将不同场景的可直接运行的webpack模版代码放在 git 上维护，通过 nodejs命令行交互方式获取用户配置项目信息和插件信息，通过 metadata 模版渲染方式，将模版和自定义部分相结合，最后输出用户需要的模版工程代码。

1、需要的第三库
commander.js，可以自动解析命令参数，用于处理用户输入的命令。
download-git-repo，下载并提取git仓库，用于下载模版。
Inquirer.js，通用的命令行用户界面集合，用于和用户进行交互。
ora，下载过程loading动画效果。
chalk，可以给终端字体颜色。
log-symbols，在终端上显示对勾或者叉图案。
latest-version，获取库最新版本号。
metalsmith，静态网站生成器，批量处理模版。
handlebars.js，模版引擎，将用户提交的动态信息填充到文件中。
2、第一步：commander.js命令行界面
第一步：通过 commander.js 实现命令init项目

program.usage(
'<name>'
).parse(process.argv)
let projectName = program.args[
0
]
if
(!projectName){
    program.help()
    
return
}
mycli init projectName 也许你觉得命令太长，想短一点，也可以支持 m2 init projectName。因为 commander 支持 git 风格的子命令处理，格式是 [command]-[subcommand]，例如：

mycli init => mycli-init
m2 init => m2-init
所以我在原先的 mycli-init.js 文件基础上新增加了一个 m2-init.js 文件。

3、第二步：Inquirer.js交互
通过 Inquirer.js 命令行问答交互方式配置项目信息和插件信息，在 myCli 中，除了一些基本信息外，还需要选择需要提前打包的第三库，以及一些开发中常用的功能。这里设置的参数名字也是后面模版生成时候占位的名字。

 const answer = await  new 
Promise
((resolve,reject)=>{
        
if
(projectRoot != 
'.'
){
            fs.mkdirSync(projectRoot);
        }
        
return
 resolve( inquirer.prompt([
            {
                name:
'projectName'
,
                message:
'项目名称'
,
                default:
'mycli-init'
            },
            ...
        ]))
    })
4、第三步：download-git-repo下载模板
通过 download-git-repo 去 git 库下载模版集合。这里的 dev 分支存储了 vue、 vuex、 skeleton、 typescript四套模版。这里是统一全部都下载下来，然后根据命令行交互的信息进行筛选和处理。

module.exports = 
function
(target){
    target = path.join(target || 
'.'
,
'.download-temp'
)
    
return
 new 
Promise
 (
function
(resolve,reject){
        const url = 
'direct:https://github.com/'
        const spinner = ora(
'正在下载模版'
)
        spinner.start()
        download(url,target,{clone:true},(err)=>{
            
if
(err){
                spinner.fail()
                reject(err)
            }
else
{
                spinner.succeed()
                resolve(target)
            }
        })
    })
}
5、第四步：生成模板
generator.js 处理所有信息输出最终工程模版。传入 metadata 数据、下载的临时文件夹地址、最终模版生成的目录地址。通过 metalsmith 和 handlebars 处理模版，删除不需要的文件，填入模版动态数据。

 const res = await generator(context.metadata,context.downloadTemp,dest);
因为download的模版是一个集合，那么我们怎么根据 Inquirer.js 选择的工程信息匹配到对应的模版，这里还需要一个 template.ignore 文件。该文件配置了文件查找的匹配规则。

{{#
if
 
Skeleton
}}
skeleton/**
skeleton/.*
{{
else
 
if
 
TypeScript
}}
ts/**
ts/.*
{{
else
 
if
 vuex}}
vuex/**
vuex/.*
{{
else
 
if
 vue}}
vue/**
vue/.*
{{/
if
}}
  return
 new 
Promise
((
resolve
,
reject
)=>{
    const metalsmith 
=
 
Metalsmith
(
process
.
cwd
()).
metadata
(
metadata
).
clean
(
false
).
source
(
src
).
destination
(
dest
);
    
//判断是否存在
ignore
文件
    const ignoreFile 
=
 path
.
join
(
src
,
'templates.ignore'
);
    
if
(
fs
.
existsSync
(
ignoreFile
)){
        metalsmith
.
use
((
files
,
metalsmith
,
done
)
 
=>{
            const meta 
=
 metalsmith
.
metadata
();
            const ignores 
=
 
Handlebars
.
compile
(
fs
.
readFileSync
(
ignoreFile
).
toString
())(
meta
)
            
.
split
(
'\n'
).
filter
(
item
=>!!
item
.
length
)
            
//删除文件
            
done
()
        
}).
build
(
err
=>{
            metalsmith
.
use
((
files
,
 metalsmith
,
done
)
 
=>
 
{
                const meta 
=
 metalsmith
.
metadata
()
                
Object
.
keys
(
files
).
forEach
(
fileName 
=>
 
{
                    
if
(
fileName
.
indexOf
(
'/src/'
)
 
==
 
-
1
){
 
//忽略业务代码
                        let t 
=
 files
[
fileName
].
contents
.
toString
()
                        files
[
fileName
].
contents 
=
 new 
Buffer
(
Handlebars
.
compile
(
t
)(
meta
))
                    
}
                
})
                
done
();
            
}).
build
(
err 
=>
 
{
                
//移动文件
            
})
        
})
  
}
  })
6、第五步：ora、log-symbols美化
通过 ora 、 chalk 和 log-symbols 美化脚手架。在下载模版时候，或者获取库版本时候，都会比较慢，如果终端没有给出任何信息似乎体验上会比较差，在下载模版的时候，增加了 ora 代码：

const spinner = ora(
'正在下载模版'
)
spinner.start()
download(url,target,{clone:true},(err)=>{
    
if
(err){
        spinner.fail()
        reject(err)
    }
else
{
        spinner.succeed()
        resolve(target)
    }
})
在最终工程模版生成的时候，可以给出字体颜色表示是否成功，例如：

console.log(logSymbols.success,chalk.green(
'创建成功:)'
));
console.log(logSymbols.error,chalk.red(
`创建失败：${err.message}`
));
三、Webpack模板

有了命令行工具，我们还需要 Webpack 工程模版。与时俱进，当然需要一些新的变化，比如 Webpack4.0 和 Babel7。

1、Webpack4.0
将Webpack从 3.x 升级到了 4.x。简单罗列一下升级中需要注意的点：

1）推荐使用 node 版本 >=8.9.0，在使用时候可以有更好的体验
2）借鉴了 parcel 零配置构建思想，新增了模式配置： mode，可选值是 production 和 development。 mode 不可缺省，需要二选一：
production 模式：
默认提供所有可能的优化，如代码压缩/作用域提升/ tree-shaking 等 不支持watching process.env.NODE_ENV 的值不需要再定义，默认是 production

development 模式：
主要优化了增量构建速度和开发体验 process.env.NODE_ENV 的值不需要再定义，默认是 development


在 Webpack 配置文件中，我们往往需要拿到当前的 npm 命令执行的模式，可以如下拿到参数值：

"dev": " webpack-dev-server --mode development   -d --open --hide-modules --progress",
 "build": "webpack --mode production --hide-modules --progress",
 "upload": "webpack --mode production --env.upload --hide-modules --progress",
module
.
exports 
=
 
(
env
,
argv
)=>
 
{
    
if
(
argv
.
mode 
===
 
'production'
){
       
//判断是否是生产环境
    
}
    
if
(
env 
&&
 env
.
upload
){
      
//判断是否是编译上传
    
}
}
在 Webpack 打包优化方面，对于第三方的依赖库，例如 vue、 vuex、 vue-router等我们会单独提前预先 Dllplugin 打包 vendor 文件。那么可以对比一下构建 vendor 的大小和时间。

Webpack3 效果如下：

Webpack4 效果如下：

从打包上看，对于Vue第三方库全家桶， 4.x 比 3.x 减少了14%。时间也减少了8%。 4.x在打包编译构建速度上做了很大优化，这里就不展开了，对于整体项目构建提升效果是很明显的。

3）MiniCssExtractPlugin 替代 ExtractTextWebpackPlugin，如果还想用 ExtractTextWebpackPlugin，可以下载 next 版本。

2、Babel7
曾经在做移动端页面的时候，为了使用 es6 或者 es7 语法，又要兼容比较低版本的 android 或者 ios 等使用场景，我们只能加载完整版本80kb左右的 polyfill 文件。 Babel7 真正做到按目标浏览器兼容的浏览器按需加载 polyfill文件。如何做到呢？我们看一下 .babelrc 文件。首先配置 babel preset env：将 ES6 的代码转成 ES5 (注意： babel-preset-es2015 已经被废弃了)。设置目标浏览器的范围，设置 useBuiltIns:usage。 usage是新特性，可以只打包业务代码中需要转换的 polyfill。

{
    
"presets"
: [
        [
"@babel/preset-env"
,
        {
           
"modules"
:false,
           
"targets"
:{
            
"browsers"
: [
"> 1%"
, 
"last 2 versions"
, 
"not ie <= 8"
,
"Android >= 4"
,
"iOS >= 8"
]
           },
           
"useBuiltIns"
:
"usage"

        }]
    ],
    
"plugins"
: [
        [
"@nutui/babel-plugin-separate-import"
,{
"style"
:
"css"
}],
        
"@babel/plugin-syntax-dynamic-import"

    ]
}
测试源码如下：

 let m = [
3
,
4
];
 console.log([...this.spread,...m]);
 let obj = 
Object
.assign({},this.obj); 
 
for
(let a of m){
       console.log(a);
  }
使用 useBuiltIns:usage 打包：



这结果是不是很让人兴奋。

3、模版集合
当我们搭建完 Webpack4 的工程模版后，为了进一步降低用户的使用的难度，从项目的实践中总结和抽取比较完整的模版方案： Vue、 Vuex、 TypeScript、 Skeleton 四套模版。目前暂时支持这些，后续考虑加入活动页模版、PC端模版等。

四、核心组件库NutUI

对于一个 Vue 技术栈来说，少不了实用的组件库。现在业界有很多优秀的组件 iView、 element 等。不过我们团队做的 NutUI2.0  Vue组件库也同样优秀，不仅服务于部门内部，也服务于公司的其他部门团队。目前有30+数量的组件，大多数来源于京东app、京东me等使用场景的实际项目积累和沉淀。基于京东APP 7.0 视觉规范。在项目中，推荐使用按需加载方式调用组件代码，减小打包体积。同时支持组件内挂载，也支持全局挂载。

import { 
DatePicker
,
Button
,
Rating
 } from 
'@nutui/nutui'
;
...
export default {
    data(){
        
return
{
        }
    },
    components: {
        
'nut-datepicker'
:
DatePicker
,
        
'nut-button'
:
Button
,
        
'nut-rating'
:
Rating
    }
}
import { 
Cell
,
Dialog
 } from 
'@nutui/nutui'
;
Vue
.use(
Cell
);
Vue
.use(
Dialog
);
五、辅助功能

我们团队开发了许多提高效率插件和功能，以增强开发体验。

一键上传
carefree：webpack插件，开发阶段自动编译上传测试服务器，提供二维码真机调试
smock：webpack插件，开发阶段基于Swagger的自动化mock假数据
skeleton：利用骨架屏vue组件和vue-server-reander 手写注入骨架屏html和css
六、小结

最后通过脚手架工具、 Webpack 模版、核心组件库 NutUI2.0 和辅助功能四个方面的讲解，大家对Vue2.0构建工具有所了解吧。整体的设计就如下图：

整体的设计学习借鉴了很多业界前辈分享的经验，结合项目中的实践的经验，给出一套解决方案，希望能给大家一点启发和帮助。

欢迎使用和关注：

NutUI2.0：http://nutui.***.com/
七、扩展阅读

1. iView: 
    https://www.iviewui.com/
2. element: 
    http://element-cn.eleme.io
3. carefree: 
    http://carefree.***.com
4. smock: 
    http://smock.***.com
5. commander.js: 
    https://github.com/tj/commander.js/
6. download-git-repo:
    https://github.com/flipxfx/download-git-
    repo
7. Inquirer.js: 
    https://github.com/SBoudrias/Inquirer.js/
8. ora: 
    https://github.com/sindresorhus/ora
9. chalk: 
    https://github.com/chalk/chalk
10. log-symbols: 
    https://github.com/sindresorhus/log-
    symbols
11. latest-version: 
    https://github.com/sindresorhus/latest-
    version
12. metalsmith: 
    https://github.com/segmentio/metalsmith
13. handlebars.js: 
    https://github.com/wycats/handlebars.js/



阅读 306 在看2
写留言