1.放大镜组件

描述：componentDidMount 函数中执行   this.initAttr() 这时候dom元素都有但是样式属性获取不到。

我发现这是react开发过程中样式注入的顺序导致的，这个页面先注入了js然后是style ，所以解决这个问题的方法是修改其注入的顺序