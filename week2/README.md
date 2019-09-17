# 第二周
## Cisco console serial to USB 模擬
### 1.拉出Switch、PC各一台，再用console線連起來。
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p1.png)
> ### 下圖的線就是Cisco console serial to USB的轉接線。
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p2.png)
## Cisco遠端連線模擬
### 1.拉兩台Router並用跳線相接。
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p3.png)
### 2.接著設定路由器名字、管理者密碼及IP,並開啟網路介面卡。
```
//R1設定
>enable
#conf t
(config)#hostname R1
(config)#enable secret (password)
(config)#interface f0/0
(config-if)#ip addr 12.1.1.1 255.255.255.0
(config-if)#no shut
```
```
//R2設定
>enable
#conf t
(config)#hostname R2
(config)#interface f0/0
(config-if)#ip addr 12.1.1.2 255.255.255.0
(config-if)#no shut
```
> ### 密碼設法有兩種，但secret的比較安全，還有如果要看設定下show running。
```
enable password (password) //低階密碼設定，只要查詢即可看到
enable secret (password)   //高階設定，查詢的話會出現雜湊函數
show running-config        //密碼會寫在這個檔案，可簡寫成show running
no enable password         //取消密碼
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p4.png)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p5.png)
> ### 如果password、secret都有，遠端登入時請登入secret的密碼，還有查看通常在特權模式才行，在全局模式時要加do，像是do show running，上圖紅線處有示範。  

### 3.查看連線狀態。
可以Ping看看，還有查看ARP和interface的資料是否一致。
```
R2(config-if)#do ping 12.1.1.1          //R2上做Ping
R2(config-if)#do show arp               //R2上查ARP
R1(config-if)#do show interface f0/0    //R1上查詢網路介面卡
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p6.png)
### 4.Telnet設定與遠端連線。
```
R1(config)#line vty 0 4
R1(config)#password ccna
R1(config)#login
```
```
R2#telnet
Password: //telnet密碼
R1>en
Password: //secret密碼
R1#
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p7.png)
> ### 當要回到上一層介面可用exit,但如果進入太深的介面可以用Ctrl+z回特權模式。
## Cisco 回存執行設定到開機設定
### 1.
