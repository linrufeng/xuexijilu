安装 
sudo npm install -g @vue/cli
注意 node版本目前要求在8.9以上
然后通过 vue create test 创建一个项目
可以选择第一个 balel eslint
也可以手动选择要安装的插件
如果选择 手动会又很多配置,最后我们启动项目 npm run 
这时候一个vue的项目就启动了，接下来介绍如何引用nut vue 
这里推荐
安需加载的方式
首先，  下载 npm i @nutui/babel-plugin-separate-import -D
然后在根目录下找到 babel.config.js 在里面增加一个配置
```
"plugins": [
["@nutui/babel-plugin-separate-import", {
    "style": "css"
}]
]
```
然后根据使用的频率，如果希望整个项目都能用例如 Dialog 这类的组件推荐在main.js这个文件中引用
```
import { Dialog} from '@nutui/nutui';
Dialog.install(Vue);
```
接下来就可以在项目里面使用了。
如果我们选择了手动配置里面的ts，那么我们创建的项目就变成了xx.ts 引用多个组件时候记得注意书写格式
```
import { Dialog, Picker} from '@nutui/nutui';
```
最后，祝愿大家使用开心，我在 git上同时也放了demo，vue-cli+nut 需要的可以直接拿走~

  
