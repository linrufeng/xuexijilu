## minimize 
属性不算是SplitChunksPlugin下的属性，但事两者之间是兄弟关系，都是Optimization（优化）下的属性, 是负责代码压缩的，在这里介绍minimize的原因是webpackV4版本中默认将压缩打开了，为了更直观的看到本文的效果，我们暂时将minimize设置为false，minimize的具体设置就不在这里展开了。 （看例1）
## minSize
模块被分离前最小的模块大小以字节（b）为单位，例如class-a（~137b）、class-b (~142b) class-c (~142b) 都将会被分离被打包为一个新文件，如果minSize设置为421以上它们就不会被分离，如果设置为421以下（包括421）它们就会被分离合并为新的文件。（看例1）
## maxSize
使用maxSize告诉webpack尝试将大于maxSize的块拆分成更小的部分。拆解后的文件最小值为minSize，或接近minSize的值。这样做的目的是避免单个文件过大，增加请求数量，达到减少下载时间的目的。但是这个值设置的时候要掌握好，如果设置的过小产生过多小文件会适得其反。另外HTTP1.0中最大的请求数量为6。在HTTP2.0中则没有限制。

```
 optimization: {
    minimize: false,
    splitChunks: {
        chunks: "all",  //  async
        minSize: 400,
        automaticNameDelimiter: '~',
    }
},
```
## chunks
表示对哪些模快进行优化，可选字符串值有 all 、 async 、 initial 、和函数。

* async 表示对动态（异步）导入的模块进行分离。（看例2.1）
* initial 表示对初始化值进行分离优化。（看例2.2）
* all 表示对所有模块进行分离优化，一般情况下都用all （看例2.3）
* 函数 代表对指定的模块快分离优化，相当于定制优化。（看例2.4）