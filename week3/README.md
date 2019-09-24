# 第三周
## EVE-NG 中間人攻擊
### 1.首先見一台Switch、一台Windows機、一台Kali Linux、一台Tiny Linux,並把線連好。
#### 1.Tiny Linux設定
```
#sudo ip addr add 192.160.1.3/24 brd + dev eth0 \\網卡設定
```
#### 2.Windows設定
進入控制台\網路和網際網路\網路和共用中心→變更介面卡設定→右鍵內容→TCP/IPv4→IP位址192.168.1.1,子網路遮罩255.255.255.0   
> ### 這樣Tiny Linux、Windows兩台可以互Ping了。
#### 3.Kali Linux設定
```
#sudo ip addr add 192.160.1.2/24 brd + dev eth0 \\網卡設定
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
## EVE-NG模擬網路封包遺失
