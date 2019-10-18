### webpack文件列表
    "autoprefixer": "^9.4.9",   //css 动管理浏览器前缀的插件
    "autoprefixer-loader": "^3.2.0", // 自动添加css前缀的功能。
    "chokidar": "^2.1.2", //node.JS的监听文件夹改变模块
    "cross-env": "^5.2.0",//运行跨平台设置和使用环境变量的脚本
    "css-loader": "^2.1.0",
    "extract-text-webpack-plugin": "^3.0.2", //抽离css样式,防止将样式打包在js中引起页面样式加载错乱的现象
    "folder-hash": "^2.1.2", //文件hash
    "html-webpack-plugin": "^3.2.0", //生成创建html入口文件，比如单页面可以生成一个html文件入
    "inquirer": "^6.2.2", // node 命令行选择功能
    "intersection-observer": "^0.5.1", //intersection-observer api的向下兼容方案
    "less-loader": "^4.1.0",
    "marked": "^0.6.1", // md转换工具
    "mini-css-extract-plugin": "^0.5.0", //单独打包css文件时
    "node-sass": "^4.11.0",
    "optimize-css-assets-webpack-plugin": "^5.0.1", ///优化或者压缩CSS资源 
    "postcss-cssnext": "^3.1.0",
    "postcss-loader": "^3.0.0", //css 自动补全 
    "sass-loader": "^7.1.0",
    "style-loader": "^0.23.1",
    "vue": "^2.6.7",
    "vue-loader": "^15.6.4",
    "vue-router": "^3.0.2",
    "vue-style-loader": "4.1.2",
    "vue-template-compiler": "^2.6.7",
    "vueg": "^1.4.5",
    "webpack": "^4.29.5",
    "webpack-cli": "^3.2.3",
    "webpack-dev-server": "^3.2.0"
### 从0开始构建自己的webpack vue项目
    本文重在记录构建webpack vue项目中遇到的一些问题，有些从网上找到了答案，但是还是希望把他们全部记录下来，如果你也想做一个这样的项目，看完本文，可以完成项目的
    搭建，如果懒得看，本人的git上有完整的配置开箱即用
#### 基本配置
配置 项目的输入和输出路径
```
entry:{
        "app":path.join(__dirname,'../webVueSite/app.js')
    },
    output:{
        filename:'[name].js',
        path:path.join(__dirname,'../dist')
    }
 ```

 如果 使用 .vue 这种组件模式的方法开发项目
 ```
 //开头
 const VueLoaderPlugin = require('vue-loader/lib/plugin')
//模块内
module:{
    rules:[
        {
            test: /\.vue$/,
            loader: 'vue-loader',                 
            options: {                               
                loaders:{
                    scss: 'vue-style-loader!css-loader!sass-loader', 
                    sass: 'vue-style-loader!css-loader!sass-loader?indentedSyntax',                       
                }
            }  
        }  
    ]
},
 plugins: [    
    new VueLoaderPlugin(),
    new htmlWebpackPlugin({           
        template: './webVueSite/template.html',
        filename: 'index.html',
        minify: {
            collapseWhitespace: true
        },
        hash: true
    }),
]

 ```
  配置项目的服务
 ```
devServer: {
    contentBase: path.resolve(__dirname, '../dist/'),
    index: 'index.html',
    compress: true, //gzip压缩
    historyApiFallback: true,
    disableHostCheck: true
}
 ```
这样开发环境就基本完成了，接下来对这些配置进行升级

 ---

### 搭建中遇到的 问题总结

运行之后chrome控制台报错
```
vue.runtime.esm.js?2b0e:619 [Vue warn]: You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build.
```
这个错误  具体是怎么产生的我是参考 http://www.bubuko.com/infodetail-2497887.html 这兄弟的介绍,简单来说就是 vue的引用问题 ，webpack 有个mian 默认引用的
和我们项目中需要引用的是不一样的
```
  resolve: {
        alias: {
            'vue$': 'vue/dist/vue.esm.js' //内部为正则表达式  vue结尾的
        }
    },
```


无法解析css样式 

```
ERROR in ./webVueSite/App.vue?vue&type=style&index=0&lang=css& (./node_modules/vue-loader/lib??vue-loader-options!./webVueSite/App.vue?vue&type=style&index=0&lang=css&) 25:0
Module parse failed: Unexpected token (25:0)
You may need an appropriate loader to handle this file type.
```
按照 vue-loader 官网文档我们使用如下配置
```
{
    test: /\.vue$/,
    loader: 'vue-loader',                 
    options: {                    
        loaders:{
            scss: 'vue-style-loader!css-loader!sass-loader', 
            sass: 'vue-style-loader!css-loader!sass-loader?indentedSyntax' 
        }
    }  
}   
```
发现并不能解析  .vue 中的
```
<style>
.example{
    color:red;
}
</style>
```
我还做了 如下尝试 
```
 loaders:{
    scss: 'vue-style-loader!css-loader!sass-loader', 
    sass: 'vue-style-loader!css-loader!sass-loader?indentedSyntax',
    css:'vue-style-loader!css-loader!'
}
```
发现错误还在，于是我按照传统的方法
```
{
    test: /(\.sc|\.c|\.sa)ss$/,
    use: [
        'style-loader',
        "css-loader",
        "postcss-loader",
        "sass-loader",                             
    ]            
},             
{
    test: /\.vue$/,
    loader: 'vue-loader',                 
    options: {                               
        loaders:{
            scss: 'vue-style-loader!css-loader!sass-loader', 
            sass: 'vue-style-loader!css-loader!sass-loader?indentedSyntax',           
        }
    }  
} 
```
这样错误就消失了不过发现又引入了一个新的错误
`
Error: No PostCSS Config found in: 
`
发现还需要配置 Postcss,即在项目的根目录创建 一个名字叫 **postcss.config.js** 的文件,内部这样写
```
module.exports = {  
    plugins: {  
      'autoprefixer': {browsers: 'last 5 version'}  
    }  
  } 
```
这时完整的 **module** 的配置方案
```
module:{
        rules:[            
            {
                test: /(\.sc|\.c|\.sa)ss$/,
                use: [
                   'style-loader',
                    "css-loader",
                    "postcss-loader",
                    "sass-loader",                             
                ]            
            },             
            {
                test: /\.vue$/,
                loader: 'vue-loader',                 
                options: {                               
                    loaders:{
                        scss: 'vue-style-loader!css-loader!sass-loader', 
                        sass: 'vue-style-loader!css-loader!sass-loader?indentedSyntax',                       
                    }
                }  
            }     
        ]
    },
```