# 第六周(直接看[2019网络工程师入门CCNA 0基础学网络系列课程8:EIGRP路由协议](https://edu.51cto.com/course/14620.html))
## EIGRP 增強型內部閘道路由協定(Enhanced Interior Gateway Routing Protocol，cisco自己內定協定)
### 與RIP相比
RIP缺點:    

1. 跳數作為Metric不能衡量好一個路徑的質量好壞。      
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/a.png)   
2. RIP最大支持15跳，16跳不可達到，不適合大型網路。      
3. 30/180S時間機制，收斂時間非常慢，不適合大型網路。(收斂時間為最終路由表完成並穩定所需的時間)      
4. RIP更新整張路由表，占用網路頻寬。(如果更新時占用過多頻寬，又加上時間機制會造成網路品質下降)      
   
EIGRP優點：      

 1. 動態路由協定。   
 2. 高級距離向量/混合向量路由協定。   
 3. IGP(內部閘道協定，Interior Gateway Protocol)     
 4. 無類路由協定(攜帶子網遮罩)   
### EIGRP鄰居
#### 以EIGRP方式，讓R1學到10.1.2.1非直連的網段。   
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/b.png)    
```
//R1
>en
#conf t
#hostname R1
#int f0/0
#ip address 10.1.1.1 255.255.255.0
#no shut
//R2
>en
#conf t
#hostname R2
#int f0/0
#ip address 10.1.1.2 255.255.255.0
#no shut
#int f0/1
#ip address 10.1.2.1 255.255.255.0
#no shut
//R3
>en
#conf t
#hostname R3
#int f0/0
#ip address 10.1.2.2 255.255.255.0
#no shut
//R2
#do ping 10.1.1.1 
#do ping 10.1.2.2
//都有連線成功
//R1
#exit
(config)#router eigrp 1     //1這個值為AS(自治系統，Autonomous system)
#network 10.0.0.0
//R2
#exit
(config)#router eigrp 1
#network 10.1.1.0
#do show run
/*
配置會出現這樣
router eigrp 1
network 10.0.0.0
auto-summary
*/
//有一行鄰居啟動提示%DUAL-5NBRCHANGE:IP-EIGRP 1:Neighbor 10.1.1.1 (FastEthernet0/0)is up: new adjacency
//R1
#do sh ip eigrp neighbors  //查看鄰居訊息，RIP沒此訊息。
/*
會出現這樣訊息
IP-EIGRP neighbors for process 1
H   Address         Interface      Hold Uptime    SRTT   RTO   Q   Seq
                                   (sec)          (ms)        Cnt  Num
0   10.1.1.2        Fa0/0          13   00:02:28  40     1000  0   1
*/
/*
H代表鄰居建立順序，Address鄰居IP位址，Interface與鄰居建立關係的接口，Hold不超過15秒，Uptime與鄰居建立的時間，SRTT平均往返時間為發送一個封包到接收這封包回應所需的時間，RTO超時重傳時間一般不超過5000，Q隊列為到現在還沒被傳出的封包正常為0。
*/
```     
##### EIGRP先建立鄰居關係再傳遞路由表，由五個封包來完成建立鄰居關係再傳遞路由表。     
 1. Hello封包：建立鄰居關係，不是傳遞路由訊息。    
 ![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/c.png)     
 2. Update封包：更新封包，更新路由表。      
 3. Query查詢封包：當網路中路由表發生變化且滿足一定條件時，就會發送查詢封包。       
 4. Reply回復封包(和Query成對出現，專門回應Query封包)     
 5. ACK回應封包      
```
//Router c7200拉三台互連，網卡IP一樣
//R1
#conf t
#int f0/0
#ip address 10.1.1.1 255.255.255.0
#no shut
#exit
//R2
#conf t
#int f0/0
#ip address 10.1.1.2 255.255.255.0
#no shut
#int f0/1
#ip address 10.1.2.1 255.255.255.0
#no shut
#exit
//R3
#conf t
#int f0/1
#ip address 10.1.2.2 255.255.255.0
#no shut
#exit
//R2
#ping 10.1.2.2      //可以連線
//Wireshark抓取R1 f0/0封包
//R1
#router eigrp 1
#network 10.0.0.0
//R2
#conf t
#router eigrp 1
#network 10.0.0.0
#end
```    
切到Wireshark查看，通常Query封包、Reply封包是看不到的，Hello封包挑一個觀察，Src:10.1.1.1(R1傳送的封包)，Dst:224.0.0.10(群播封包)，Protocol: EIGRP(88)。        
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/d.png)     
Hello封包應該匹配的參數，不然無法建立鄰居關係。      

 1. autonomous system: 1(AS)一定要設定一樣，如果一個為1，一個為2，無法建立鄰居關係。     
 2. K1 頻寬，K2 負載，K3 延遲，K4 可靠性，K5 最大傳輸單元(MTU,Maximum Transmission Unit)，K6 實驗位不做討論，K => Key。    

Wireshark查看Update封包，Src:10.1.1.2(R2傳送的封包)，Dst:224.0.0.10(群播封包)，10.1.1.2傳給224.0.0.10。     
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/e.png)     
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/f.png)      
Internal Route = 10.1.2.0/24 對R1而言10.1.2.0/24為一個非直連網段，通過Update封包攜帶來，RouterID:10.1.2.1(10.1.2.1)不討論，Wide Metric(EIGRP距離向量路由協定，距離=Metric值，向量=下一跳)有Bandwidth 頻寬,MTU 最大傳輸單元(MTU,Maximum Transmission Unit),Delay 延遲,Reliability 可靠性,Load 負載，NextHop:0.0.0.0下一跳為自己(R1)。     
Hello封包建立鄰居關係且有Keepalive作用，判斷發送端和鄰居是否為存活狀態。      
```
//R1
#end
#sh ip route
//D        10.1.2.0/24 [90/30720] via 10.1.1.2,00:12:12,FastEthernet0/0
//D為Dual，90為AD值(RIP為120)，30720為Metric。
#sh ip Protocol
//看得到K1=1,K2=0,K3=1,K4=0,K5=0，K1,K3來計算Metric值，公式:256*(10^7/Min(bandwidth)+Sum(Delay)/10)。
#sh int f0/0
//看得到MTU 1500 bytes,BD(bandwidth) 100000 Kbit/sec, DLY(Delay) 100usec...。
//R2
#sh int f0/1
//也看得到MTU 1500 bytes,BD(bandwidth) 100000 Kbit/sec, DLY(Delay) 100usec...。
//Metric值=256*{(10^7/10^5)+[100(R1的)+100(R2的)]/10}=256*(100+200/10)=256*120=30720。
```
### DUAL 擴展更新算法
EIGRP三表:鄰居表，拓撲表，路由表。     
傳聞路由放置於拓撲表，拓撲表再傳給路由表。      
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/g.png)      
R5傳給R1，R1>R2 Metric5值為10，剩下如圖示，有三個選項，R2,R3,R2|R3。     

 1. FD(Feasible distance):本路由器到達目標網段最小的Metric。    
 2. AD(Advertised Distance):鄰居路由器到目標網段的Metric。      
 3. S(Successor)後繼，到目標網段最小Metric的鄰居路由器。      
 4. FS(Feasible Successor)可行後繼，AD小於FD的鄰居路由器。         

```
                   FD    AD
           |-R2    30    20
R1目標網段 -|-R4    25    10
           |-R3    40    30
```
S為R4，AD 20,30指向25。
```
//R1
#conf t
#int f0/0
#ip address 12.1.1.1 255.255.255.0
#no shut
#exit
#int f0/1
#ip address 13.1.1.1 255.255.255.0
#no shut
#exit
//R2
#conf t
#int f0/0
#ip address 12.1.1.2 255.255.255.0
#no shut
#exit
#int f0/1
#ip address 24.1.1.2 255.255.255.0
#no shut
#exit
//R3
#conf t
#int f0/1
#ip address 13.1.1.3 255.255.255.0
#no shut
#exit
#int f0/0
#ip address 34.1.1.3 255.255.255.0
#no shut
#exit
//R4
#conf t
#int f0/1
#ip address 24.1.1.4 255.255.255.0
#no shut
#exit
#int f0/0
#ip address 34.1.1.4 255.255.255.0
#no shut
#exit
#int f1/0
#ip address 45.1.1.4 255.255.255.0
#no shut
#exit
//R5
#conf t
#int f1/0
#ip address 45.1.1.5 255.255.255.0
#no shut
#exit
//R2
#do ping 12.1.1.1
#do ping 24.1.1.4
//R3
#do ping 13.1.1.1
#do ping 34.1.1.4
//R4
#do ping 45.1.1.5
//R1
#router eigrp 90
#network 12.0.0.0
#network 13.0.0.0
#exit
//R2
#router eigrp 90
#network 12.0.0.0
#network 24.0.0.0
#exit
//R3
#router eigrp 90
#network 13.0.0.0
#network 34.0.0.0
#do show run | sec eigrp
/*顯示   router eigrp 90
         network 13.0.0.0
         network 34.0.0.0
*/
#exit
//R4
#router eigrp 90
#network 24.0.0.0
#network 34.0.0.0
#network 45.0.0.0
#exit
//R1
#end
#sh ip route eigrp
#sh ip eigrp topology
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/h.png)        
```
//R3
#exit
#int f0/1
#bandwidth 90000
//R1
#sh ip route eigrp
#sh ip eigrp topology
//改了沒變化
#no bandwidth
#exit
#int f0/0
#bandwidth 90000
#sh ip route eigrp
#sh ip eigrp topology all-links //查看從鄰居學習到的左右路由表
//都產生變化
```
### EIGRP非等價負載均衡
到達相同目標網段不同的下一跳，同時具有相同的AD和不同的Metric = 等價路由。      
條件：      

 1. 能夠滿足FC Feasible Condition，可行性條件：AD < FD值 = 有FS。        
 2. variance * AD > FD。      

```
//延用之前的設定
//R3
#int f0/0
#bandwidth 95
#end
//R1
#sh ip eigrp topology
//R3
#conf t
#int f0/0
#bandwidth 95000
#end
//R1
#variance 2
/*
D     45.1.1.0[90/34560] via 13.1.1.3, 00:00:14, FastEthernet 0/1
              [90/33280] via 12.1.1.2, 00:00:14, FastEthernet 0/0
*/
//R3
#conf t
#int f0/0
#bandwidth 90000
#end
//R1
#do show ip eigrp topology
#sh ip eigrp topology to all
#do show ip route
```
### EIGRP 自動匯總與手動匯總
```
//R4
#interface loopback 1
#ip address 172.16.1.1 255.255.255.0
#no shut
#exit
//R1
#exit
#int lool
#ip address 172.16.2.1 255.255.255.0
#no shut
#exit
#router eigrp 90
#network 172.16.2.0
//R4
#router eigrp 90
#network 172.16.1.0
#do show run | sec eigrp
//有個子網 172.16.0.0
#sh ip route eigrp
//R3
#sh ip route eigrp
```
IOS 12.4以前的版本自動匯總默認為開啟，需要手動關閉。     
    12.4以後的版本自動匯總默認為關閉。       
```
//R3
#end
#sh ip route
#auto-summary
//R1
#auto-summary
//R4
#auto-summary
//R3
#sh ip route eigrp
//R4
#end
#sh ip eigrp neighbors
#show run | sec eigrp
//R3
#sh ip route eigrp
#sh ip eigrp topology to all
#confit t
#int f0/0
#no bandwidth
#end
#sh ip route eigrp
#conf t
#no ip cef
#end
#ping 172.16.1.1  //封包一個成功一個失敗
//R1
#end
#conf t
#router eigrp 90
#no auto-summary
//R3
#conf t
#router eigrp 90
//R4
#conf t
#router eigrp 90
//R1
#end
//R4
#exit
#int loo2
#ip address 172.16.22.2 255.255.255.0
#exit
#int loo4 
#ip address 172.16.23.2 255.255.255.0
#exit
#router eigrp 90
#ip address 172.16.23.2 255.255.255.0
#exit
#router eigrp 90
#network 172.16.0.0
#show ip route
#no auto-summary
//R1
#conf t
#no int loo1
//R4
#ip address 172.16.55.0 255.255.255.0
#ip address 172.16.55.5 255.255.255.0
#no shut
#exit 
#int f0/0
#ip summary-address eigrp 90
//22=00010110,23=00010111
#ip summary-address eigrp 90 172.16.22.0 255.255.254.0
#exit
#int f0/1
#ip summary-address eigrp 90 172.16.22.0 255.255.254.0
#no ip summary-address eigrp 90 172.16.22.0 255.255.254.0
#ip summary-address eigrp 90 172.0.0.0 255.0.0.0
#exit
#int f0/0
#no ip summary-address eigrp 90 172.0.0.0 255.0.0.0
#ip summary-address eigrp 90 172.0.0.0 255.0.0.0
#int f0/0
#do sh run int f0/1
#no ip summary-address eigrp 90 172.16.22.0 255.255.254.0
#ip summary-address eigrp 90 172.0.0.0 255.0.0.0
```     
### EIGRP 驗證   
     
 1. key-chain     
 2. key     
 3. key-string     
 4. 在接口聲明     
 5. 在接口下調用     
     
```     
//R1
#key
#key chain cisco
#key 1
#key-string cisco123
#exit
//R2
#exit
#key
#key chain cisco
#key 1
#key-string cisco123
#exit
//R1
#exit
#int f0/0
#ip authentication key-chain eigrp 90 cisco
#ip authentication mode eigrp 90 md5
#end
#sh ip eigrp neighbors
//R2
#exit
#int f0/0
#ip authentication mode eigrp 90 md5
#ip authentication key-chain eigrp 90 cisco
#end
```      
EIGRP鄰居建立條件：     
AS相同，5個K值一樣，認證相同。
```
//R1
#sh ip eigrp topology
#sh ip protocols
#sh ip eigrp interfaces f0/0
#sh ip eigrp events     //eigrp記錄
```
### EIGRP 單播更新與水平分割
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/i.png)  
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/j.png)   
```
//R1
#conf t
#int f0/0
#ip address 123.1.1.1 255.255.255.0
#no shut
#exit
//R2
#conf t
#int f0/0
#ip address 123.1.1.2 255.255.255.0
#no shut
#exit
//R1
#int loo1
#ip address 192.168.1.1 255.255.255.0
#no shut
//R2
#int loo1
#ip address 192.168.2.2 255.255.255.0
#no shut
//R3
#conf t
#int loo1
#ip address 192.168.3.3 255.255.255.0
#no shut
#exit
#int f0/0
#ip address 123.1.1.3 255.255.255.0
#no shut
#end
#ping 123.1.1.255
//R2
#end
#ping 192.168.1.1   //不成功
#sh ip rou
#conf t
#router eigrp 90
#network 123.0.0.0
#network 192.168.2.0
#end
#sh ip rou eigrp
#ping 192.168.1.1 //成功
//R3
#conf t
#router eigrp 90
#network 123.0.0.0
#network 192.168.3.0
//R1
#conf t
#router eigrp 90
#neighbor 123.1.1.2 f0/0
#neighbor 123.1.1.3 f0/0
//R2
#conf t
#router eigrp 90
#neighbor 123.1.1.1 f0/0
//R3
#conf t
#router eigrp 90
#neighbor 123.1.1.1 f0/0
//R1
#end
#sh ip eigrp neighbors
//R2
#end
#sh ip eigrp neighbors
//查看R2到SW的封包，都看到單播封包。
//R1
#sh run | section eigrp
//R2
#sh run | sec eigrp
//R3
#sh run | sec eigrp
```
![image](https://github.com/LarrySu508/cisco-note/blob/master/week6/k.png)   