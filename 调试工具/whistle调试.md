whistle 调试安装及使用总结
whistle 是一个调试工具，使用 whistlejs 的好处有很多
例如：
1. 开发过程中可以支持跨越配置
2. host快速切换无缓冲
3. js文件快速映射，可以快速的定位线上问题
4. M端调试利器

### 安装

npm install -g whistle 
w2 -h //帮助信息

### 启动
w2 start 
如果想指定端口 w2 start -p 8899

打开页面 
http://127.0.0.1:8899/

### 如何抓包 

#### chrome浏览器

#### cp wife 代理 