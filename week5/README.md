# 第五周
##  靜態路由與動態路由
### 靜態路由
優點是不會有路由封包佔據頻寬，缺點是只要其中一條傳輸路徑斷了，傳送端到接收端的通訊就斷了。    
### 動態路由
優點是其中一條傳輸路徑斷了，會尋找其他路徑，缺點是維護成本高，意思是每隔一段時間，路由器之間交換訊息確保路徑，佔據部分網路頻寬。    
#### 1.尋徑協定      
##### 1.距離向量(distance-vector)尋徑      
路由與路由彼此交換訊息把路徑選出並建立，週期性地與相鄰路由器交換路徑資訊，路徑隨時更新，但造成佔據許多傳輸頻寬，適合中小型網路。     
![image](https://github.com/LarrySu508/cisco-note/blob/master/week5/IMG20191008142149.jpg)    
距離(distance)為經過路由器的成本，每經過一個路由器加一，以跳數當成本。      
向量(vector)為下一跳方向的意思，比如網路1傳訊息到網路4，中間有經過三個路由順序為R1,R2,R3，此時向量就是R1的下一跳R2。    
##### 2.連線狀態尋徑協定     
透過廣播了解整個網路拓撲，再用演算法計算點到點之間最短距離，最初建立路線時，耗時且耗頻寬，路徑連線狀態更改時，才做路徑資訊更新。     
![image](https://github.com/LarrySu508/cisco-note/blob/master/week5/IMG_20191008_141540.jpg)     
#### 2.距離
並非實際長度量測距離。       
成本有各種，有時間延遲(Propagation Delay)、頻寬(Bandwidth)、負載(Loading)、可靠度(Reliability)、路程計數(Hop Count)、時脈計數(Tic Count)、相對價格(Relative Cost)。      
最短距離並不一定是最快的，也並非實際長度量測最短的距離。         
## RIP
### 1.為距離向量(distance-vector)以跳數當成本，選最段距離時，選跳數少的路徑距離。   
### 2.為動態路由協定隨時更新，每隔一段時間路由表資訊就要更新，花很多頻寬。
### 3.為IGP(Interior Gateway Protocol)，在一個自治系統內部使用的路由選擇協議，不同於EGP(External Gateway Protocol)，在自治系統的相鄰兩個閘道器主機間交換路由資訊的協議，下圖有兩個自治區圓圈，IGP的意思為各個自治區圓圈內部傳輸協定，EGP為兩個自治區圓圈隔一條線，以那條線為傳輸閘道的傳輸協定，而中華電信為一個自治區，臺灣學術網路也是一個自治區。
### 4.有V1,V2,NG三種版本，NG(next generation)給IPV6用的，V1為有類路由(A類,B類,C類,D類,E類)用，不包含Mask，彼此路徑訊息傳遞以廣播的方式，使用IP位置為255.255.255.255，不支援加密，V2為無類路由用，有包含Mask，彼此路徑訊息傳遞以群播的方式，使用IP位置為224.0.0.9，有支援加密。
### 5.AD(Administrative Distance)值為120。
![image](https://github.com/LarrySu508/cisco-note/blob/master/week5/IMG20191008161656.jpg)     
## RIP實作
![image](https://github.com/LarrySu508/cisco-note/blob/master/week5/IMG1.png)    
### 1.開三台Cisco IOL L3_ADVENTERPRISEK9的Router(R1,R2,R3)    
### 2.網卡設定
```
//R1
>en
#conf t
(config)#hostname R1
(config)#int e0/0
(config-if)#ip addr 10.1.1.1 255.255.255.0
(config-if)#no shut
(config-if)#exit
(config)#do ping 10.1.1.2      //測試可否ping到R2，結果是可以。
//R2
>en
#conf t
(config)#hostname R2
(config)#int e0/0
(config-if)#ip addr 10.1.1.2 255.255.255.0
(config-if)#no shut
(config)#int e0/1
(config-if)#ip addr 10.1.2.1 255.255.255.0
(config-if)#no shut
(config-if)#exit
//R3
>en
#conf t
(config)#hostname R3
(config)#int e0/0
(config-if)#ip addr 10.1.2.2 255.255.255.0
(config-if)#no shut
(config-if)#exit
(config)#do ping 10.1.2.1      //測試可否ping到R2，結果是可以。
```
### 3.RIP設定(V1)
```
//R1
(config)#router rip
//開R1 e0/0的Wireshark觀察封包
(config-router)#network 10.1.1.1
//R2
(config)#router rip
(config-router)#network 10.1.2.1
//R1
(config-router)#do show run
/*
  會出現!
        router rip
         network 10.0.0.0
        !
  而network有兩個作用，一是激活介面，二是傳遞網路訊息，network 10.0.0.0為主類位址，所以如果設network 10.1.1.0,network 10.1.0.0,network 10.0.0.0，都一樣會顯示network 10.0.0.0，因為為A類網路。
*/
//R2
(config-router)#do show run
/*
  會出現!
        router rip
         network 10.0.0.0
        !
  和R1一樣顯示network 10.0.0.0
*/
//R1
(config-router)#exit
(config)#exit
#show ip route
//會看到剛剛R2沒設定的R 10.1.2.0/24 [120/1] via 10.1.1.2, 00:00:06, Ethernet0/0，因為R2 network 10.1.2.1被激活，所以R1可以學習到，R代表RIP，[120/1]為[RIP/一跳]，00:00:06為更新時間，30秒更新一次，180秒內認為斷線(hold down)，超過240秒清除路由(flash)。
#show ip protocols
//會看到Routing Protocol is "rip"，裡面有寫30秒更新一次，180秒內認為斷線(hold down)，超過240秒清除路由(flash)。
#ping 10.1.2.1      //ping R2 e0/1成功
#ping 10.1.2.2      //ping R3 e0/0失敗，R1可以傳到R3，但R3沒法傳到R1。
//R3
>en
#show ip route
//沒有R 10.1.1.0/24 [120/1] via 10.1.2.1, 00:00:12, Ethernet0/0
#conf t
(config)#router rip
(config-router)#network 10.1.2.2
(config-router)#do show ip route
//會看到R 10.1.1.0/24 [120/1] via 10.1.2.1, 00:00:12, Ethernet0/0
//R1
#ping 10.1.2.2      //ping R3 e0/0成功
//再看Wireshark，找RIP的封包(Protocol 為RIPv1)
//R2
(config-router)#exit
(config)#exit
#show ip interface e0/0
//網卡位址為aabb.cc00.0200，Wireshark的封包有顯示R2為來源，傳送為廣播ff:ff:ff:ff:ff:ff，目的255.255.255.255，使用UDP協定來源與目的埠號為520，IP Addr 10.1.2.0，路由器在直連網路上傳送非直連路由給另一台router，Wireshark的封包在01 02後有 00 00 00 00 00 00 00 00，前四組0為網路遮罩，因V1不傳送網路遮罩所以皆為0，後四組0為下一跳，這些訊息具有隱性描述的概念，暗指下一跳可能為10.1.1.2，metric為1，因為經過一個路由器。
```
### 4.RIP設定(V2)
延用RIP設定(V1)的三台router。
```
//R2
>en
#conf t
(config)#int e0/1
(config-if)#ip addr 192.168.1.1 255.255.255.0       //直接覆蓋舊的IP Addr。
(config-if)#do show ip int brief        //確認是否修改成功
//R3
>en
#conf t
(config)#int e0/0
(config-if)#ip addr 192.168.1.2 255.255.255.0   
//開R1 e0/0的Wireshark觀察封包
//R1
>en
#conf t
(config)#router rip
(config-router)#version 2
//R2
(config-if)#router rip
(config-router)#version 2
//R3
(config-if)#router rip
(config-router)#version 2
//R2
(config-router)#network 192.168.1.1
/*
  再看Wireshark，找RIP的封包(Protocol 為RIPv2)。
  使用IP Addr為群播的，封包來源IP 10.1.1.2，封包目的224.0.0.9，來源與目的埠號為520，192.168.1.0傳到R1-R2網路，封包在01 00後有 ff ff ff 00 00 00 00 00，前四組0為網路遮罩，因V2會傳送網路遮罩255.255.255.0，後四組0為下一跳，還是隱性描述，暗指下一跳可能為10.1.1.2，metric為1，因為經過一個路由器。
*/
```
## 網路聚合
### 1.完成右圖設定
![image](https://github.com/LarrySu508/cisco-note/blob/master/week5/IMG_20191008_152221.jpg)
#### 1.延用之前三台路由器，首先再加三台VPCS(Virtual PC Simulator)，並把IP設好
```
//h1
>ip 172.16.1.1 255.255.255.0 172.16.1.254
//ipaddr mask gateway
//h2
>ip 10.1.3.1 255.255.255.0 10.1.3.254
//h3
>ip 172.16.2.1 255.255.255.0 172.16.2.254
//R1
>en
#conf t
(config)#hostname R1
(config)#int e0/1
(config-if)#ip addr 172.16.1.254 255.255.255.0
(config-if)#no shut
(config-if)#int e0/0
(config-if)#ip addr 10.1.1.1 255.255.255.0
(config-if)#no shut
(config-if)#do show ip int brief
//確認設定
//R2
>en
#conf t
(config)#hostname R2
(config)#int e0/0
(config-if)#ip addr 10.1.1.2 255.255.255.0
(config-if)#no shut
(config-if)#int e0/1
(config-if)#ip addr 10.1.2.1 255.255.255.0
(config-if)#no shut
(config-if)#int e0/2
(config-if)#ip addr 10.1.3.254 255.255.255.0
(config-if)#no shut
(config-if)#do show ip int brief
//確認設定
//R3
>en
#conf t
(config)#hostname R3
(config)#int e0/0
(config-if)#ip addr 10.1.2.2 255.255.255.0
(config-if)#no shut
(config-if)#int e0/1
(config-if)#ip addr 172.16.2.254 255.255.255.0
(config-if)#no shut
(config-if)#do show ip int brief
//確認設定
```
#### 2.設定路由協定
```
//R1
(config-if)#router rip
(config-router)#version 2
(config-router)#network 172.16.1.254
(config-router)#network 10.1.1.1
//R2
(config-if)#router rip
(config-router)#version 2
(config-router)#network 10.1.1.2
(config-router)#network 10.1.2.1
(config-router)#network 10.1.3.254
(config-router)#do show run
/*
  會出現!
        router rip
         version 2
         network 10.0.0.0
        !
  顯示network 10.0.0.0 主類位址
*/
//R3
(config-if)#router rip
(config-router)#version 2
(config-router)#network 10.1.2.2
(config-router)#network 172.16.2.254
//R2
(config-router)#exit
(config)#do show ip route
//可看到往172.16.0.0/16位址，一個經由10.1.2.2(R2往R3)，另一個經由10.1.1.1(R2往R1)，因為此兩路由為等價路由。
//h2
>ping 10.1.3.254  //成功
>ping 10.1.1.1    //成功
>ping 10.1.2.2    //成功
>ping 172.16.1.1  //成功  
//R2
(config)#no ip cef  //固定路徑移除，變均衡路徑(分別給等價路由配送封包，如上述設定一次往10.1.2.2，一次往10.1.1.1，循環配送)
//h2
>ping 172.16.1.1  //出現一次成功，一次失敗，循環下去，因為左右兩邊都是172.16.0.0的網路，如上述一次往10.1.2.2，一次往10.1.1.1，循環配送，出現此狀況是因為把網路聚合在一起。
```
#### 3.解除網路聚合(有如超網化，把很多個子網路合併在一起，用一個網路位址來代表，可節省路由條目，如果沒做在更新路由表時，子網路全部更新很占頻寬)
網路聚合在子網路不連續情況下(如上述例子172.16.1.1、10.1.3.1、172.16.2.1，中間的子網把兩邊連續子網切開)，會出現上述問題，所以要解除網路聚合。
```
//R1
(config)#router rip
(config-router)#no auto-summary
//R2
(config)#router rip
(config-router)#no auto-summary
//R3
(config-router)#router rip
(config-router)#no auto-summary
(config-router)#do show run 
/*
  會出現!
        router rip
         version 2
         network 10.0.0.0
         network 172.16.0.0
         no auto-summary
        !
*/
//R2
(config-router)#do show ip route
/*
會發現 R 172.16.0.0/16 [120/1] via 10.1.2.2, 00:01:44, Ethernet0/1
                       [120/1] via 10.1.1.1, 00:02:44, Ethernet0/0
  有超過30秒，過了4分鐘就會消除，如果想馬上看到結果可以下以下指令。
*/  
(config-router)#exit
(config)#exit
#clear ip route *
#show ip route
/*
會發現變成  172.16.0.0/24 is subnetted, 2 subnets 
         R    172.16.1.0[120/1] via 10.1.1.1, 00:00:06, Ethernet0/0
         R    172.16.2.0[120/1] via 10.1.2.2, 00:00:06, Ethernet0/1
*/ 
```
### 2.上述實驗再加入個外網的電腦，並做認證
![image](https://github.com/LarrySu508/cisco-note/blob/master/week5/tp.png)
#### 1.對RIP再加入設定
```
//R3
>en
#conf t
(config)#router rip
(config-router)#default-information originate
//R2
>en
#conf t
(config)#do show ip route
//會多個內定路由R* 0.0.0.0/0 [120/1] via 10.1.2.2, 00:00:08, Ethernet 0/1
//R1
>en
#show ip route
//跟R2一樣多個內定路由R* 0.0.0.0/0 [120/1] via 10.1.1.2, 00:00:15, Ethernet 0/0
```
#### 2.認證
```
//R1
#conf t
(config)#key chain MyChain  //MyChain為各人設定的名字，可以自行設定
(config-keychain)#key 1
(config-keychain-key)#key-string 1234   //鑰匙密碼
(config-keychain-key)#int e0/0
(config-if)#ip rip authentication key-chain MyChain
(config-if)#ip rip authentication mode md5
(config-if)#exit
(config)#exit
#clear ip route *
#show ip route
//R1加密了，所以剛剛的R* 0.0.0.0/0 [120/1] via 10.1.1.2, 00:00:15, Ethernet 0/0，此內定路由沒出現。
//R2
(config)#exit
#clear ip route *
#show ip route
/*
R2沒加密，所以剛剛的R* 0.0.0.0/0 [120/1] via 10.1.2.2, 00:00:05, Ethernet 0/1，內定路由有顯示。
還有兩個RIP 172.16.1.0和172.16.2.0消失，因為R1加密R2沒加密，所以R1,R2無法通訊。
*/
##conf t
(config)#key chain MyChain
(config-keychain)#key 1
(config-keychain-key)#key-string 1234  
(config-keychain-key)#int e0/0
(config-if)#ip rip authentication key-chain MyChain
(config-if)#ip rip authentication mode md5
(config-if)#do show ip route
//R1
#show ip route
/*
R1,R2都設定好認證內定路由0.0.0.0/0就可傳，其他路徑RIP 10.1.2.0,10.1.3.0,10.1.4.0都可傳到，此為Version 2，因為Version 1沒有支援加密，V1內定值是自動匯總，不能關閉，V2的話是auto-summary，也可以no auto-summary。
*/
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week5/IMG20191008161657.jpg)