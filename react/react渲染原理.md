做了两个react项目，在感概react的好用之时，我也抽空研究了下 react的原理。

一段 .jsx的代码是如何渲染的呢
```
return (
    <div className="cn">
         <Header> Hello, This is React </Header>
         <div>Start to learn right now!</div>
         Right Reserve.
    </div>
)
```
首先，它会经过babel编译成React.createElement的表达式。
```
return (
    React.createElement(
        'div',
        { className: 'cn' },
        React.createElement(
            Header,
            null,
            'Hello, This is React'
        ),
        React.createElement(
            'div',
            null,
            'Start to learn right now!'
        ),
        'Right Reserve'
    )
)
```
这个createElement方法是做什么的呢？

其实从它的名字就可以看出，这是用来生成element的。element在React里，其实就是组成虚拟DOM 树的节点，它用来描述你想要在浏览器上看到什么。

它的参数有三个：

1、type -> 标签

2、attributes -> 标签属性，没有的话，可以为null

3、children -> 标签的子节点

这个React.createElement的表达式会在render函数被调用的时候执行，换句话说，当render函数被调用的时候，会返回一个element。

说了那么久element，这个element究竟长什么样呢？

其实，它就是一个对象，如下:

{
  type: 'div',
    props: {
      className: 'cn',
        children: [
          {
            type: function Header,
            props: {
                children: 'Hello, This is React'
            }
          },
          {
            type: 'div',
            props: {
                children: 'start to learn right now！'
            }
          },
          'Right Reserve'
      ]
  }
}
我们来观察一下这个对象的children，现在有三种类型：

1、string

2、原生DOM节点

3、React Component - 自定义组件

除了这三种，还有两种类型：

4、fale ,null, undefined,number

5、数组 - 使用map方法的时候

这里需要记住一个点：element不一定是Object类型。

二、element如何生成真实节点
顺利得到element之后，我们再来看看React是如何把element转化成真实DOM节点的。

首先，需要去初始化element,初始化的规则如下：

以第一条为例：先判断是否为Object类型，是的话，看它的type是否是原生DOM标签，是的话，给它创建ReactDOMComponent的实例对象，其他同理。



这时候有的人可能会有所疑问：这些个ReactDOMComponent, ReactCompositeComponentWrapper怎么开发的时候都没有见过？

其实这些都是React的私有类，React自己使用，不会暴露给用户的。它们的常用方法有：mountComponent,updateComponent等。其中mountComponent 用于创建组件，而updateComponent用于用户更新组件。

而我们自定义组件的生命周期函数以及render函数都是在这些私有类的方法里被调用的。

既然这些私有类的方法那么重要我们就先来简单了解一下吧~

ReactDOMComponent
首先是ReactMComponent的mountComponent方法，这个方法的作用是：将element转成真实DOM节点，并且插入到相应的container里，然后返回markup（realDOM）。

由此可知ReactDOMComponent的mountComponent是element生成真实节点的关键。



下面看个栗子它是怎么做到的吧。

假设有这样一个type类型是原生DOM的element:

{
  type: 'div',
    props: {
    className: 'cn',
      children: 'Hello world',
    }
}
简单mountComponent的实现：

mountComponent(container) {
  const domElement = document.createElement(this._currentElement.type);
  const textNode = document.createTextNode(this._currentElement.props.children);

  domElement.appendChild(textNode);
  container.appendChild(domElement);
  return domElement;
}
其实实现的过程很简单，就是根据type生成domElement,再将子节点append进来返回。当然，真实的mountComponent没有那么简单，感兴趣的可以自己去看源码啦。

这里需要记住的一个点是：这个类的mountComponent方法会自己操作浏览器DOM元素。

讲完ReactDOMComponent，再来看看ReactCompositeComponentWrapper。

ReactCompositeComponentWrapper
这个类的mountComponent方法作用是：实例化自定义组件，最后是通过递归调用到ReactDOMComponent的mountComponent方法来得到真实DOM。 注意：也就是说他自己是不直接生成DOM节点的。 那这个递归是一个怎样的过程呢？我们通过首次渲染来看下。

首次渲染
假设我们有一个Example的组件，它返回<div>hello world</div> 这样一个标签。 首次渲染的过程如下：



首先从React.render开始，由于我们刚刚说，render函数被调用的时候会返回一个element，所以此时返回给我们的element是：

{
  type: function Example,
  props: {
    children: null
  }
}
由于这个type是一个自定义组件类，此时要初始化的类是ReactCompositeComponentWrapper,接着调用它的mountComponent方法。这里面会做四件事情，详情可以看上图。其中，第二步的render的得到的element为：

{
  type: 'div',
    props: {
    children: 'Hello World'
  }
}
由于这个type是一个原生DOM标签，此时要初始化的类是ReactDOMComponent。接下来它的mountComponent方法就可以帮我们生成对应的DOM节点放在浏览器里啦。

这时候有人可能会有疑问，如果第二步render出来的element 类型也是自定义组件呢？

这时候它就会去调用ReactCompositeComponentWrapper的mountComponent方法，从而形成了一个递归。不管你的自定义组件嵌套多少层，最后总会生成原生dom类型的element，所以最后一定能调用到ReactDOMComponent的mountComponent方法。

感兴趣的可以自己在打断点看下这个递归的过程。

由我打的断点图可以看出在ReactCompositeComponent的mountComponent被调用多次之后，最后调用到了ReactDOMComponent的mountComponent方法。 

到这里，首次渲染的过程就基本讲完了:-D。

但是还有一个问题：前面我们说自定义组件的生命周期跟render函数都是在私有类的方法里被调用的，现在只看到render函数被调用了，那么首次渲染时候生命周期函数 componentWillMount 跟 componentDidMount 在哪被调用呢？



由图可知，在第一步得到instance对象之后，就会去看instance.componentWillMount是否有被定义，有的话调用，而在整个渲染过程结束之后调用componentDidMount。 以上，就是渲染原理的部分，让我们来总结以下：

JSX代码经过babel编译之后变成React.createElement的表达式，这个表达式在render函数被调用的时候执行生成一个element。
在首次渲染的时候，先去按照规则初始化element，接着ReactComponentComponentWrapper通过递归，最终调用ReactDOMComponent的mountComponent方法来帮助生成真实DOM节点。