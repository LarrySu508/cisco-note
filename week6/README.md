# 第六周
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

