# 第三周
## EVE-NG 中間人攻擊
### 1.首先見一台Switch、一台Windows機、一台Kali Linux、一台Tiny Linux,並把線連好。
#### 1.Tiny Linux設定
```
#sudo ip addr add 192.168.1.3/24 brd + dev eth0 \\網卡設定
```
#### 2.Windows設定
進入控制台\網路和網際網路\網路和共用中心→變更介面卡設定→右鍵內容→TCP/IPv4→IP位址192.168.1.1,子網路遮罩255.255.255.0   
> ### 這樣Tiny Linux、Windows兩台可以互Ping了。
#### 3.Kali Linux設定
```
#sudo ip addr add 192.168.1.2/24 brd + dev eth0 \\網卡設定
```
設定完記得去Ping另外兩台做檢查。   
接著監聽Windows,Tiny Linux通訊，本次用Ping做測試。   
```
#tcpdump -nni eth0 icmp   \\監聽指令
```
開另一個Terminal開駭客工具Ettercap   
```
#ettercap -G  
```
選取工具列Sniff→Unified sniffing→介面卡eth0    
了解網路有哪些主機，選取工具列Hosts→Scan for hosts    
選取工具列Hosts→Host List     
選取192.168.1.1→list下方的add to target1     
選取192.168.1.3→list下方的add to target2      
選取工具列Mitm→ARP poisoning→選取sniff remote connections     
接著就可以看到剛剛下tcpdump -nni eth0 icmp的Terminal可以監聽到icmp的封包了。    
如果不確定可以點工具列Mitm→stop mitm attack，就會看到監聽的Terminal停住了。    
也可以用之前用過的Wireshark在Switch做監聽。
## EVE-NG封閉網路新增SSH套件
如果你EVE-NG的Linux沒SSH功能要怎麼辦?    
### 1.建Tiny Linux和Network，並連線
Tiny Linux跟之前的設置是一樣，Network選NAT。    
### 2.在google查詢tinycore install ssh server
> ### 記得設定Tiny Linux的IP。
### 3.SSH安裝
```
#tce-load -wi openssh
#sudo /usr/local/etc/init.d/openssh start
#netstat -tunlp | grep 22  \\查看ssh是否開啟
#sudo adduser user   \\新增使用者
#ssh user@127.0.0.1   \\這樣就架設了封閉網路的SSH
```
## EVE-NG模擬網路封包遺失或延遲
### 1.先建一台Linux-Netem(emulation)和兩台Tiny Linux(TL1,TL2)，Linux-Netem記得改2條網路線，TL1連到Linux-Netem，Linux-Netem連到TL2。
### 2.中間的Linux-Netem當作一朵雲(Net)，TL1傳封包到TL2(要先設好網卡IP)。
```
\\TL1
#sudo ip addr add 192.168.1.1/24 brd + dev eth0
\\TL2
#sudo ip addr add 192.168.1.2/24 brd + dev eth0
\\TL1
#ping 192.168.1.2
\\結果封包序號是連續的，所以沒封包遺失。
```
### 3.接著更改Linux-Netem的設定，登入後有個Loss \<None>選項，選取後Packet Loss [%]輸入50後，再去用TL1 ping TL2序號會出現不連續，有遺失封包的情形。
### 4.延遲的話回主選單有個Delay選項，延遲單位為ms，如果設定50的話，原本icmp封包只要10ms左右就可以收到，改完後變100ms左右，這樣就模擬到延遲效果了。
## Cisco Data Center Multi-Tier Model Topology
重點是Layer 3以上封包broadcast沒關係，因為其他端口不會回應，Layer 3之下要避免封包broadcast產生迴圈，所以會用到STP(Spanning Tree Protocol)。    
參考資料[Cisco Data Center Multi-Tier Model Topology](https://www.cisco.com/c/en/us/td/docs/solutions/Enterprise/Data_Center/DC_Infra2_5/DCInfra_2.html)
