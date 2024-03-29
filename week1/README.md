# 第一周
## Cisco 模擬兩台pc在router上做連線
### 1.首先拉出兩台pc和一台路由器,並把線連好。
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p1.png)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p2.png)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p3.png)
> #### 如果要看到各個接口代號，請把options的preferences裡，下圖的選項勾選。 
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
### 3.之後一樣也把fa1/0在同一個視窗做IP設定。
```
Router(config)#interface fa1/0
Router(config-if)#ip address 192.168.43.1 255.255.255.0 //設定IP與遮罩
Router(config-if)#no shutdown //啟動fa1/0
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p6.png)
### 4.再來設定兩個PC的IP和Default GateWay，要與router做對應的IP喔。
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p7.png)
### 5.最後測試兩台PC是否可以互相做ping的動作。
![image](https://github.com/LarrySu508/cisco-note/blob/master/week1/p8.png)
