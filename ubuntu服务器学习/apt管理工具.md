sudo apt-get update #更新源

sudo apt-get install package #安装包

sudo apt-get remove package # 删除包

sudo apt-cache search package # 搜索软件包

sudo apt-cache show package #获取包的相关信息，如说明、大小、版本等

sudo apt-get install package –reinstall #重新安装包

sudo apt-get -f install #修复安装

sudo apt-get remove package –purge #删除包，包括配置文件等

sudo apt-get build-dep package #安装相关的编译环境

sudo apt-get upgrade #更新已安装的包

sudo apt-get dist-upgrade #升级系统

sudo apt-cache depends package #了解使用该包依赖那些包

sudo apt-cache rdepends package #查看该包被哪些包依赖

sudo apt-get source package #下载该包的源代码

sudo apt-get clean && sudo apt-get autoclean #清理无用的包

sudo apt-get check #检查是否有损坏的依赖 
 
 ##  mongodb
 sudo apt-get update

 sudo apt-get install mongodb

 ## 查看是否启动 
 pgrep mongo -l。

 ## 启动MongoDB命令:sudo service mongodb start。

###关闭MongoDB命令:sudo service mongodb stop。

查看MongoDB的安装位置命名：locate mongo


查看配置文件信息，默认mongodb 配置文件存放在 /etc/mongodb.conf。

卸载MOngoDB命令：

sudo apt-get --purge remove mongodb mongodb-clients mongodb-server

## nodejs

 sudo apt-get install  nodejs
  sudo apt-get install npm 
 sudo    npm    install    -g    n

 sudo    n    latest 最新

sudo    n    stable 稳定

sudo    n    lts 长期支持 
##

## nginx

sudo apt-get install nginx

 启动Nginx 服务。


sudo systemctl start nginx

 开机自动启动nginx 服务

sudo systemctl enable nginx

sudo nginx start #运行nginx
修改配置后重新加载
nginx -s reload
service nginx stop
. 关闭开机自动启动nginx 服务
### 查看端口号占用
sudo netstat -anp | grep nginx

## 如何升级

ubuntu apt-get 安装完nginx后是1.4.6版的，以下是对该版本的升级


Nginx Stable PPA是由Ubuntu社区维护的源，本源更新自稳定版分支，是Kaijia目前使用的源，这个源的特点是文件的目录结构和Ubuntu自带的Nginx相同，因此安装这个版本时不需要修改/etc/nginx/下面的配置文件。不过这个源更新比较慢，一般Nginx
新版本发布后要过一至两周才会更新，但质量是保证的。安装此源只需要在终端中运行：


apt-add-repository ppa:nginx/stable
然后即可安装Nginx：

     
apt-get update
apt-get install nginx

用以上信息安装完后nginx并不能解析php了，需要加入
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;