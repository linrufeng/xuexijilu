## 本地资源强制与 git库代码同步，覆盖本地
* git fetch --all && git reset --hard origin/master && git pull
## 代码提交
* git pull 拉取
* git add . 
* git commit -m ""
* git push 

## 全局设定 密码 啥的 
* git config --global credential.helper store
* 这样输入一次以后再也不用输入了
* 其实就是config 文件里面 多了他 ```[credential]
    helper = store```
* 查看配置文件