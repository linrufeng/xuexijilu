ssh root@****** 和密码就可以链接了
### 关闭图形化界面 
sudo systemctl set-default multi-user.target
sudo reboot
### 常用命令
cd 路径 （进入一个路径，比如 /usr/local/lib）
cd …　　 　　　　 （返回上一个文件夹）
ls　　　　　　　　 （显示当前文件夹下的所有文件，Linux独有哦，dir 也有相同功能）
sudo 命令　　　　 （获取超级管理权限，需要输入密码）

### 常用新建、删除、拷贝命令：

mkdir 目录名 （新建一个文件夹，文件夹在Linux系统中叫做“目录”）
touch 文件名 （新建一个空文件）
rmdir 目录名 （删除一个空文件夹，文件夹里有内容则不可用）
rm -rf 非空目录名 （删除一个包含文件的文件夹）
rm 文件名 文件名 （删除多个文件）
cp 文件名 目标路径（拷贝一个文件到目标路径，如cp hserver /opt/hqueue）
cp -i　　　　　　 （拷贝，同名文件存在时，输出 [yes/no] 询问是否执行）
cp -f　　　　　　 （强制复制文件，如有同名不询问）

### 常用解压、安装程序、文件更新命令：deb格式双击即可安装

tar -zxvf *.tar.gz　 ( 解压 tar.gz格式的文件 )
source .install　　　 （安装install格式的安装包）
sh 路径/×.sh　　 　 （安装sh格式的文件，如 sudo sh /home/hp/Downloads/.sh）
sudo apt-get upgrade（更新已安装的包）
sudo apt-get update （更新源）
### 安装云 yum
首先检测是否安装了build-essential程序包
sudo apt-get install build-essential
sudo apt-get install yum
alias yum='sudo apt-get 

yum（全称为 Yellow dog Updater, Modified）是一个在Fedora和RedHat以及SUSE中的Shell前端软件包管理器。基於RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

vim 退出编辑模式 qa!
