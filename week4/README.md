# 第四周
## EVE-NG靜態路由(Static Route)
就是於路由表上設置靜態路由。  
重點指令：
```
#ip route 目的網路 遮罩 下一站位址或離開介面
```
### 1.設置兩台Router Node，並連線
![image](https://github.com/LarrySu508/cisco-note/blob/master/week4/a.png)
### 2.對R1,R2做網卡與路由設置
```
\\R1
>en
#conf t
(config)#hostname R1
(config)#int e0/0
(config-if)#ip addr 12.1.1.1 255.255.255.0
(config-if)#no shut
(config-if)#int lo 1
(config-if)#ip addr 1.1.1.1 255.255.255.255
(config-if)#do show ip int brief
\\會顯示Ethernet0/0 12.1.1.1與Loopback1 1.1.1.1狀態是up。
```
```
\\R2
>en
#conf t
(config)#hostname R2
(config)#int e0/0
(config-if)#ip addr 12.1.1.2 255.255.255.0
(config-if)#no shut
(config-if)#int lo 1
(config-if)#ip addr 2.2.2.2 255.255.255.255
(config-if)#do show ip int brief
\\會顯示Ethernet0/0 12.1.1.2與Loopback1 2.2.2.2狀態是up。
```
```
\\R1
(config-if)#exit
(config)#exit
#ping 12.1.1.2
#show ip route  \\顯示路由表，C是連線的意思，L是localhost的意思
#show ip ?  \\提醒問號可接的指令，此功能只在EVE-NG可用
#ping 12.1.1.2 repeat 3   \\ping三次
#ping 12.1.1.2 source 1.1.1.1     
\\開wireshark觀察封包可看到送出，但沒回來，ping出現失敗，因為R2上沒設定到R1的路由
```
```
\\R2
(config-if)#exit
(config)#ip route 1.1.1.1 255.255.255.255 12.1.1.1
(config)#do show ip route       \\可看到S(static) 1.1.1.1 [1/0] via 12.1.1.1的靜態路由
```
> ### S 1.1.1.1 [1/0] via 12.1.1.1 的 [1/0]，[管理距離（Administrative Distance,AD,數值越小權重越大）/ 度量值（Metric）]，度量值為從本點到目的行走的距離，值越低=成本越低，優先選擇，預設為0。
```
\\R1
#ping 12.1.1.2 source 1.1.1.1      \\呈現成功
\\如果要ping 2.2.2.2 source 1.1.1.1
#conf t
(config)#ip route 2.2.2.2 255.255.255.255 12.1.1.2
(config)#do show ip route       \\可看到S 2.2.2.2 [1/0] via 12.1.1.2的靜態路由
(config)#ping 2.2.2.2 source 1.1.1.1 repeat 3   \\就呈現成功
```
> ### (config-if)#do show arp顯示本地端的MAC address
## EVE-NG內定路由
### 1.R2做8.8.8.8的路由設定
```
\\R2
>en
#conf t
#int lo 2
#ip addr 8.8.8.8 255.255.255.255
#do show ip int brief       \\Loopback2 8.8.8.8狀態是up
```
### 2.R1做內定路由設定
```
\\R1
>en
#ping 8.8.8.8   \\結果會是失敗的，因為沒設定路由
#conf t
#ip route 0.0.0.0 0.0.0.0 12.1.1.2   \\設定內定路由
#do show ip route          \\可看到S* 0.0.0.0/0 [1/0] via 12.1.1.2的內定路由
```
### 3.R2做內定路由設定
```
\\R2
#exit
#ip route 0.0.0.0 0.0.0.0 12.1.1.1   \\設定內定路由
#do show ip route          \\可看到S* 0.0.0.0/0 [1/0] via 12.1.1.1的內定路由
```
### 4.R1去ping 8.8.8.8呈現成功
```
\\R1
#do ping 8.8.8.8    \\呈現成功
#do ping 8.8.8.8 source 1.1.1.1    \\呈現成功
```
## EVE-NG浮動路由
當兩條連線L1,L2，L1為正常時的連線，L2為待機連線，L1斷掉時，L2自動做替補動作。  
### 1.再加一台Router(R3)，Node設定跟R1,R2一樣，並與R1連線(可以Add Text作說明)
![image](https://github.com/LarrySu508/cisco-note/blob/master/week4/b.png)
### 2.R1設定內定路由
```
\\R1
\\先設定e0/1網卡
>en
#conf t
(config)#int e0/1
(config-if)#ip addr 13.1.1.1 255.255.255.0
(config-if)#no shut
(config-if)#exit
(config)#do show ip route
(config)#ip route 0.0.0.0 0.0.0.0 13.1.1.3   \\設定內定路由
\\如果下ip route 0.0.0.0 0.0.0.0 13.1.1.3 ?，會出現問號可接的設定，像是[管理距離（Administrative   Distance,AD,數值越小權重越大）/ 度量值（Metric）]設定
(config)#do show ip route         
\\可看到S* 0.0.0.0/0 [1/0] via 13.1.1.3
                    [1/0] via 12.1.1.2的等價路由
\\刪除路由no ip route 0.0.0.0 0.0.0.0 13.1.1.3
(config)#no ip route 0.0.0.0 0.0.0.0 13.1.1.3
(config)#ip route 0.0.0.0 0.0.0.0 13.1.1.3 10
\\在do show ip route上看不到
(config)#do show ip static route 0.0.0.0/0   \\靜態位址到0.0.0.0/0的所有路由列出
\\可看到M 0.0.0.0/0 [1/0] via 12.1.1.2 [A] 
       M           [10/0] via 13.1.1.3 [N]的靜態路由設定([A]為啟動,[N]為未啟動)
```
### 3.R3設定路由
```
\\R3
>en
#conf t
(config)#hostname R3
(config)#int e0/0
(config-if)#ip addr 13.1.1.3 255.255.255.0
(config-if)#no shut
(config-if)#int lo 1
(config-if)#ip addr 8.8.8.8 255.255.255.255
(config-if)#do show ip route
\\會顯示C      8.8.8.8 is directly connected,Loopback1。
```
### 4.R1測試
```
\\R1
(config)#do ping 13.1.1.3
\\可以ping到
```
### 5.R3內定路由設定
```
\\R3
(config-if)#exit
(config)#ip route 0.0.0.0 0.0.0.0 13.1.1.1
\\可看到S* 0.0.0.0/0 [1/0] via 13.1.1.1的內定路由
(config)#exit
#debug ip icmp
```
### 6.R2觀察設定
```
\\R2
>en
#debug ip icmp
```
### 7.R1實際測試
```
\\R1
(config)#do ping 8.8.8.8 source 1.1.1.1
\\觀察到是往R2那條走
(config)#int e0/0
(config-if)#shut
\\關閉R1 e0/0的網卡，意思是R1到R2連線中斷
(config-if)#do ping 8.8.8.8 source 1.1.1.1
\\觀察到是往R3那條走
(config-if)#do show ip route
\\路徑從S* 0.0.0.0/0 [1/0] via 12.1.1.2，變成S* 0.0.0.0/0 [10/0] via 13.1.1.3
(config-if)#do show ip static route 0.0.0.0/0
\\可看到M 0.0.0.0/0 [1/0] via 12.1.1.2 [N]  (從A變N) 
       M           [10/0] via 13.1.1.3 [A]  (從N變A)
```
> ### 上述debug ip icmp可以觀察ping的封包，跟Wireshark一樣可以觀察到ICMP封包
## EVE-NG等價路由(三角拓撲、負載均衡)
記得把剛剛R1關掉的網卡e0/0開啟。  
```
\\R1
>en
#conf t
(config)#int e0/0
(config-if)#no shut
```  
並把連線連好。  
![image](https://github.com/LarrySu508/cisco-note/blob/master/week4/c.png)
### 1.R3路由設定
```
\\R3
>en
#conf t
(config)#int lo 1
(config-if)#no ip addr 8.8.8.8 255.255.255.255
(config-if)#do show ip int brief      \\Loopback1 IP-Address變成unassigned
(config-if)#int e0/1
(config-if)#ip addr 23.1.1.3 255.255.255.0
(config-if)#no shut
(config-if)#do show ip int brief
\\會顯示Ethernet0/1 23.1.1.3狀態是up。
```
### 2.R1路由設定
```
\\R1
>en
#conf t
(config)#int lo 1
(config-if)#do show ip int brief
\\確認兩張網卡是否都開啟並設定好，Ethernet0/0 12.1.1.1狀態是up，Ethernet0/1 13.1.1.1狀態是up。
```
### 3.R2路由設定
```
\\R2
>en
#conf t
(config)#int e0/1
(config-if)#ip addr 23.1.1.2 255.255.255.0
(config-if)#no shut
```
> ### 設定完記得作互ping的測試，R2 ping 23.1.1.3，R3 ping 13.1.1.3，確認連線是否正常，但最主要是確認每個機子對應網卡是否有設好。
### 4.R1設定AD(管理距離)相同的靜態路由
```
\\R1
(config-if)#ip route 2.2.2.2 255.255.255.255 12.1.1.2 1
(config-if)#ip route 2.2.2.2 255.255.255.255 13.1.1.3 1
(config-if)#exit
(config)#do show ip route
\\可看到S  2.2.2.2 [1/0] via 13.1.1.3
                   [1/0] via 12.1.1.2的等價路由
```
### 5.R1上作Ping測試與觀察
首先開啟R1上兩個網卡e0/1、e0/0的Wireshark，觀察ICMP封包發送狀態
```
\\R1
(config)#ip cef     \\cisco的路由指定，意思為一開始傳送時到第三層找址，找到後之後的傳送就用這個址，不會再到第三層找址。
(config)#do ping 2.2.2.2 source 1.1.1.1
\\會發現只有一個Wireshark的視窗有ICMP封包的紀錄
(config)#no ip cef  \\關閉ip cef
(config)#do ping 2.2.2.2 source 1.1.1.1
\\會發現兩個Wireshark的視窗都有ICMP封包的紀錄，此時就有負載均衡的概念在此拓撲上。
```