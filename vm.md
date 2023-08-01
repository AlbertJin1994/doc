# 1、下载&创建虚拟机

<img src="./assets/centosdownload01.png" alt="image-20221027200546522" style="zoom: 25%;" />

<img src="./assets/centosdownload02.png" alt="image-20221027200742863" style="zoom:25%;" />

<img src="./assets/centosdownload03.png" alt="image-20221027200941441" style="zoom:25%;" />

<img src="./assets/centosdownload04.png" alt="image-20221027202724488" style="zoom:25%;" />

<img src="./assets/centosinstall01.png" alt="image-20221027215502140" style="zoom:25%;" />

<img src="./assets/centosinstall02.png" alt="image-20221027215752323" style="zoom: 50%;" />

<img src="./assets/centosinstall03.png" alt="image-20221027220006005" style="zoom: 50%;" />

<img src="./assets/centosinstall04.png" alt="image-20221027220210059" style="zoom: 50%;" />

<img src="./assets/centosinstall05.png" alt="image-20221027221421275" style="zoom: 50%;" />

<img src="./assets/centosinstall06.png" alt="image-20221027221553730" style="zoom: 33%;" />

<img src="./assets/centosinstall07.png" alt="image-20221027221743945" style="zoom: 50%;" />

<img src="./assets/centosinstall08.png" alt="image-20221027222532593" style="zoom:50%;" />

<img src="./assets/centosinstall09.png" alt="image-20221027222639363" style="zoom: 50%;" />

<img src="./assets/centosinstall10.png" alt="image-20221027222732192" style="zoom: 50%;" />

<img src="./assets/centosinstall11.png" alt="image-20221027222809908" style="zoom: 50%;" />

<img src="./assets/centosinstall12.png" alt="image-20221027223019092" style="zoom: 50%;" />

<img src="./assets/centosinstall13.png" alt="image-20221027223118846" style="zoom: 50%;" />

<img src="./assets/centosinstall14.png" alt="image-20221027223246699" style="zoom: 25%;" />

# 2、安装CentOS

<img src="./assets/centosinstall15.png" alt="image-20221027223644407" style="zoom: 33%;" />

<img src="./assets/centosinstall16.png" alt="image-20221027223954774" style="zoom:33%;" />

<img src="./assets/centosinstall17.png" alt="image-20221027224156403" style="zoom: 25%;" />

<img src="./assets/centosinstall18.png" alt="image-20221028200340771" style="zoom: 67%;" />

<img src="./assets/centosinstall19.png" alt="image-20221028200914596" style="zoom: 67%;" />

<img src="./assets/centosinstall20.png" alt="image-20221028201028634" style="zoom: 67%;" />

<img src="./assets/centosinstall30.png" alt="image-20221028204457559" style="zoom: 67%;" />

<img src="./assets/centosinstall21.png" alt="image-20221028202246878" style="zoom: 67%;" />

<img src="./assets/centosinstall22.png" alt="image-20221028202502832" style="zoom: 67%;" />

<img src="./assets/centosinstall23.png" alt="image-20221028202933836" style="zoom: 67%;" />

<img src="./assets/centosinstall26.png" alt="image-20221028203622457" style="zoom: 67%;" />

<img src="./assets/centosinstall28.png" alt="image-20221028203432776" style="zoom: 67%;" />

<img src="./assets/centosinstall25.png" alt="image-20221028203535181" style="zoom: 67%;" />

<img src="./assets/centosinstall29.png" alt="image-20221028203859706" style="zoom: 67%;" />

<img src="./assets/centosinstall31.png" alt="image-20221028204623606" style="zoom: 67%;" />

<img src="./assets/centosinstall32.png" alt="image-20221028204732498" style="zoom:67%;" />

<img src="./assets/centosinstall34.png" alt="image-20221028205049639" style="zoom:67%;" />

<img src="./assets/centosinstall33.png" alt="image-20221028204940009" style="zoom:67%;" />

<img src="./assets/centosinstall35.png" alt="image-20221028205500728" style="zoom:67%;" />

<img src="./assets/centosinstall36.png" alt="image-20221028205347869" style="zoom:67%;" />

<img src="./assets/centosinstall37.png" alt="image-20221028210002365" style="zoom:67%;" />

<img src="./assets/centosinstall38.png" alt="image-20221028210124225" style="zoom: 25%;" />

# 3、网络设置

## 3.1、网络连接模式

<img src="./assets/othersettings01.png" alt="image-20221028213445948" style="zoom:50%;" />

<img src="./assets/othersettings02.png" alt="image-20221028213724244" style="zoom: 33%;" />

<img src="./assets/othersettings03.png" alt="image-20221028214006687" style="zoom:33%;" />

### 3.1.1、桥接模式

虚拟机直接连接外部物理网络的模式，主机起到了网桥的作用。这种模式下，虚拟机可以直接访问外部网络，并且对外部网络是可见的。（不安全，不推荐）

![image-20221028221805072](./assets/桥接模式.png)

### 3.1.2、仅主机模式

虚拟机只与主机共享一个专用网络，与外部网络无法通信。

![image-20221028225550211](./assets/仅主机模式.png)

### 3.1.3、NAT模式

Network Address Transition。虚拟机和主机构建一个专用网络，并通过虚拟网络地址转换（NAT）设备对IP进行转换。虚拟机通过共享主机IP可以访问外部网络，但外部网络无法访问虚拟机。(简单说，也就是通过VMnet8构建了一个虚拟子网)

![image-20221028224252967](./assets/NAT模式.png)

## 3.2、设置静态IP

<img src="./assets/更改静态IP.png" alt="image-20221029100625035" style="zoom:50%;" />

<img src="./assets/image-20221029101843921.png" alt="image-20221029101843921" style="zoom: 33%;" />

<img src="./assets/设置静态IP02.png" alt="image-20221029102014754" style="zoom: 25%;" />

<img src="./assets/设置静态IP03.png" alt="image-20221029102226400" style="zoom: 50%;" />

<img src="./assets/设置静态IP04.png" alt="image-20221029102336042" style="zoom: 33%;" />

<img src="./assets/设置静态IP05.png" alt="image-20221029102754994" style="zoom:33%;" />

<img src="./assets/设置静态IP06.png" alt="image-20221029102905707" style="zoom: 50%;" />

## 3.3、设置主机名

<img src="./assets/更改主机名.png" alt="image-20221029103800091" style="zoom:50%;" />

<img src="./assets/设置主机名02.png" alt="image-20221029104626083" style="zoom: 67%;" />

<img src="./assets/设置主机名01.png" alt="image-20221029104515577" style="zoom:50%;" />

<img src="./assets/设置主机名03.png" alt="image-20221029105104824" style="zoom: 50%;" />

<img src="./assets/设置主机名04.png" alt="image-20221029105217884" style="zoom:50%;" />

<img src="./assets/设置主机名05.png" alt="image-20221029105309531" style="zoom:50%;" />

<img src="./assets/设置主机名06.png" alt="image-20221029110229148" style="zoom:50%;" />

# 4、克隆（创建完整克隆）

https://blog.csdn.net/u013916029/article/details/126557005

http://t.zoukankan.com/snhk-p-10850061.html

