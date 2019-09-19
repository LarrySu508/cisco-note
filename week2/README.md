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
enable password (password) //低階密碼設定，只要查詢即可看到。
enable secret (password)   //高階設定，查詢的話會出現雜湊函數。
show running-config        //密碼會寫在這個檔案，可簡寫成show running。
no enable password         //取消密碼。
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p4.png)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p5.png)
> ### 如果password、secret都有，遠端登入時請登入secret的密碼，還有查看通常在特權模式才行，在全局模式時要加do，像是do show running，上圖紅線處有示範。  

### 3.查看連線狀態。
可以Ping看看，還有查看ARP和interface的資料是否一致。
```
R2(config-if)#do ping 12.1.1.1          //R2上做Ping。
R2(config-if)#do show arp               //R2上查ARP。
R1(config-if)#do show interface f0/0    //R1上查詢網路介面卡。
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p6.png)
### 4.Telnet設定與遠端連線。
```
R1(config)#line vty 0 4
R1(config-line)#password ccna
R1(config-line)#login
```
```
R2#telnet 12.1.1.1
Password: //telnet密碼
R1>en
Password: //secret密碼
R1#
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p7.png)
> ### 當要回到上一層介面可用exit,但如果進入太深的介面可以用Ctrl+z回特權模式。
## Cisco 回存執行設定到開機設定及清除startup設定
### 1.回傳指令
```
R1#copy running-config startup-config //要在特權模式下。
R1#write  //這是另一種回存方法，有些設備可以執行，有些不行。
```
> ### 補充指令：要看路由器IP簡要時用。
```
R1#show ip interface brief
```
> ### 補充指令：重開路由器。
```
R1#reload
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p8.png)
### 2.清除startup
```
R1#write erase //請別隨便下，會把startup-config所有設定，使用情境為路由器設定太亂要全重設的情況。
```
## Cisco 中斷不是內部指令的指令
有可能你下了一個指令不正確，機子會去Domain Server一直找，此時要跳出請按Ctrl+Shift+6。  
也可用指令直接把不存在的指令排掉，這樣就不會去Domain Server一直找。
```
R1(config)#no ip domain-lookup 
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p9.png)
## Cisco 設定逾時待機
當你在設定時有事抽身，又怕有人來亂動你的路由設定，你可以把逾時待機的時間縮短。   
```
R1(config-line)#exec-timeout 0 5 //設定逾時0至5秒待機。
```
如果機子太多要設，覺得這設定太煩，可以用no關掉。   
```
R1(config-line)#no exec-timeout
```
## Cisco 延遲生效問題
當你在下指令時前一個指令突然生效，這時出現一些生效訊息，而你指令打到一半，此時怎麼辦?你可以入下面的指令。   
```
R1(config-line)#logging synchronous
```
這樣你把後來的指令輸入完時，會在下方顯示。  

## EVE-NG遠端連線模擬
Window作業系統的要先去下載window integration pack
### 1.開啟虛擬機，在瀏覽器輸入虛擬機IP
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p10.png)
### 2.登入後，開新的工作檔
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p11.png)
### 3.建置兩台路由器並連線
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p12.png)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p13.png)
### 4.啟動兩台路由器，並開啟命令列介面(CLI)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p14.png)
### 5.啟動後，做路由器名稱設定、IP Address設定、啟動路由器網卡
```
//R1設定
>enable
#conf t
(config)#hostname R1
(config)#interface e0/0
(config-if)#ip addr 12.1.1.1 255.255.255.0
(config-if)#no shut
```
```
//R2設定
>enable
#conf t
(config)#hostname R2
(config)#interface e0/0
(config-if)#ip addr 12.1.1.2 255.255.255.0
(config-if)#no shut
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p15.png)
### 6.開啟Wireshark,當R2 ping R1時會看得到ICMP的傳輸狀態
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p16.png)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p17.png)
### 7.做Telnet設定
```
//R1設定
R1(config)#enable secret (password)
R1(config)#line vty 0 4
R1(config-line)#password ccna
R1(config-line)#login
R1(config-line)#transport input telnet //可以發現這裡會比Cisco更真實一點。
```
```
R2#telnet 12.1.1.1
Password: //telnet密碼
R1>en
Password: //secret密碼
R1#
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p18.png)
> ### 補充指令：如果要看執行時的設定檔用。   
```
R1(config-line)#do show running
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p19.png)
### 8.此時查看剛剛開啟的Wireshark，會看到剛剛再登入Telnet的動作，輸入的密碼也看得一清二楚
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p20.png)

## EVE-NG 創建交換機與兩台Linux，並讓兩台電腦互Ping
### 1.先下載eve ng linux client
![image](https://github.com/LarrySu508/cisco-note/blob/master/week2/p21.png)
