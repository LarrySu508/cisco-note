# 第一周
## Cisco 模擬兩台pc在router上做連線
### 1.首先拉出兩台pc和一台路由器,並把線連好。
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p1.png)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p2.png)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p3.png)
> #### 如果要看到各個接口代號，請把options的preferences裡，把下圖的選項勾選。 
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p4.png)
### 2.點選router,到CLI選單做以下指令,設定fa0/0這個接口網路IP。
```
Router>enable
Router#config terminal 
Router(config)#interface fa0/0
Router(config-if)#ip address 192.168.42.1 255.255.255.0 //設定IP與遮罩
Router(config-if)#no shutdown //啟動fa0/0
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p5.png)
