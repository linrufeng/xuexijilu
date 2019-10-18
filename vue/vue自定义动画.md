### 自定义 动画 vue 自定义类 animate.css
**自定义进出动画的class名**
```
<transition 
    enter-active-class="rollIn"
    leave-active-class="rollOut"
    mode="out-in"
>   
    <div class="animated" v-show="exchange">
        .....
    </div>
</transition>
```
说明:
 enter,leave 代表离开和进入的两个状态 可以自己定义使用css的名字 例如例子使用的 是 rollIn rollOut
里面的div 是用来渲染现实与否的 这里我使用的是 setTimeout 0 就是用来切换下视图来出发过度效果函数的发生
animated 则是 animate.css 中需要驱动动画的方法