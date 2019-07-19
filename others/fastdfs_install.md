### FastDFS 安装

#### 安装libfastcommon
libfastcommon是FastDFS的一个公共库，在安装FastDFS之前要先安装这个库
~~~
#下载并且解压
wget https://github.com/happyfish100/libfastcommon/archive/master.zip
unzip master.zip
#编译
./make.sh
#安装 
./make.sh install
~~~


#### 安装FastDFS
与安装libfastcommon过程类似
~~~
#下载并且解压
wget https://github.com/happyfish100/fastdfs/archive/master.zip
unzip master.zip
#编译
./make.sh
#安装 
./make.sh install
~~~

#### 启动tracker与storage
找到对应的可执行文件执行启动操作即可
~~~
fdfs_trackerd /etc/fdfs/tracker.conf restart
fdfs_storaged /etc/fdfs/storage.conf restart
~~~


#### Nginx 安装
https://www.jianshu.com/p/7928278db041
https://segmentfault.com/a/1190000016353605