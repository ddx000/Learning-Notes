---
tags: 基礎學科, 計算機網路
---

# 計算機網路 - Network Layer

## 概論

- Network layer是以**router和IP協議組成**
- router會分辨IP datagram要去哪裡(使用Routing table查表)
- 到達目的地後，再把IP address的header去掉 把segment給transport layer (向上傳)
- forwarding (查表 分送 表從routing algorithm來)
- routing algorithm: 根據網路壅塞的程度，決定走那裡最近
- Transport layer: between two processes(Socket)
- Network layer: between two host

## Network Layer Service Models
- **Datagram networks** Internet(現在常用)
- 不建連線，不保證傳輸頻寬，才需要TCP去補償
- 郵政系統：含sourceip / destinationip
- 查Routing table
- **Virtual Circuits** ATM frame relay, X.25, 古老架構 現在不使用
- 專線建連線，可以保證頻寬連線
- 查表，有點像是查哪條管道，速度很快，不是查ip

## Router要幹嘛
https://www.netadmin.com.tw/netadmin/zh-tw/technology/3F5D612B541C4D578C10E7DC75697B60
- 路由器最重要的工作: 知道"packet"該送到哪裡
- 類似郵政系統(有source_ip / destination_ip)
- 尋找這個封包可能要送往的路徑有哪些 / 從可能的路徑中選出最佳路徑 /  維護並更新這些路由所需的資料
![](https://i.imgur.com/WFtpekr.png)
- 為了讓圖1中的10.120.2.0與172.16.1.0能夠找到路徑互相傳遞資料封包，中間的路由器設備就必須互相「分享」所學習到的資料
![](https://i.imgur.com/GjVdcIh.png)
- 跑算法(routing)-->做表-->分送(forwarding)
- Run routing algorithms/protocol (RIP, OSPF, BGP) -->做出表
- 做出表後 -->forwarding datagrams from incoming to outgoing link
- 困難點: 大量封包交換
- 進出速度不一致時要buffering (priority Queue)

## 靜態路由
- 靜態路由: 管理者手動輸入，好處是速度很快，不需要經過學習，缺點: 要維護
一般來說，靜態路由會使用在連接Stub Network之間的連線，Stub Network指的是只能透過單一路由路徑連接的網路，因此有時候也稱Stub Network為Leaf Node。
![](https://i.imgur.com/h65jLXm.png)
在Router A 上，我要讓他知道假如要到172.16.1.0 要先把封包給172.16.2.1 (靜態配置有點像LINKED LIST，我要知道我下一個指向誰)

## 動態路由
- 動態路由: 另一種方式就是透過其他路由器設備來學習，學習的方式就是透過路由協定（Routing Protocol）來交換資訊，這Routing Protocol也是學習路由器設備相關知識最重要的一環，不同的Routing Protocol有不同的使用時機，其運作方式也不盡相同，這種經過學習而建立出來的路由資訊，就稱為動態路由（Dynamic Route）。

## 路由表包含什麼
- 目標IP / 子網掩碼(判斷IP所屬網路) / 下一個要去的地方Next hop interface
- 路由表可能包含: 到達目的地的cost / 路由的品質

## IP datagram format
![](https://i.imgur.com/BSHNymH.png)
一個IP數據報應該有下列東西
- 版本(IPv4 or IPv6)
- 長度(hearder or packet長度)
- 生存時間(TTL): 為了避免數據報繞不出去
- 協議：TCP/UDP
- 檢查碼: Header checksum
- IP fragmentation & reassembly (IP太大可以切 但是HEAD要COPY)

## SUBNET(子網路)
- 在同一個子網路的裝置，**可直接溝通，不需要ROUTER**
- 不同的SUBNET需要ROUTER分配封包
- 223.1.1.X 前三碼一樣
- SUBNET可以切不一樣大小(MASK不一樣大)
![](https://i.imgur.com/L4AwmqS.png)
- MASK如果是24bit 這樣只有後面8bit可以區分host, 故此子網路下只能區分256個裝置

## IP與子網路遮罩
[classA-D + 子網路遮罩](https://notes.andywu.tw/2018/ip%E7%AD%89%E7%B4%9A%E8%88%87%E5%AD%90%E7%B6%B2%E8%B7%AF%E9%81%AE%E7%BD%A9%E4%BB%8B%E7%B4%B9/)
- 之前用Class A-D 後來用子網路遮罩表達是不是在同一個網域
- 遮罩前(被1遮的)當作網路位置 / 之後就是主機位置
- 用來切不同區域網路
- Class A: 0.0.0.0 ~ 127.0.0.0(通常是大政府才有可能拿到)
- Class B: 140.112.X.X
- 後來發現用ABCD去切各個網域，並不是很好
![](https://i.imgur.com/ExEcOev.png)

## 子網路遮罩的好處
http://linux.vbird.org/problem/linux/problem_2.php
同一個子網路遮罩底下-->區域網路-->不必透過router傳送資料
最大的好處就是在同一個網段中，資料將可以透過廣播的方式傳遞，而不必透過 router 來傳送
子網路遮罩就將這兩個IP位址經過辨別是否目的地跟自己的來源地是同一個子網路中，如果是的話那就直接傳送，如果不是的話，那就交給路由器（Router）去傳


## CIDR
(任意切IP，不用classA-D這個強硬切法)  
無分類編址CIDR 消除了傳統A 類、B 類和C 類地址以及劃分子網的概念，使用網絡前綴和主機號來對IP 地址進行編碼，網絡前綴的長度可以根據需要變化。  
IP 地址::= {< 網絡前綴號>, < 主機號>}  
CIDR 的記法上採用在IP 地址後面加上網絡前綴長度的方法，例如    128.14.35.7/20 表示前20 位為網絡前綴。  

## 如何拿到IP
- DHCP(讓ROUTER給)
- 固定IP(要設自己IP MASK 等等 比較複雜)


## 網路傳遞概念
- 各層傳遞間 加上header
![](https://i.imgur.com/ghmuXKw.png)

![](https://i.imgur.com/VUiwgVa.png)

## NAT: Network Address Translation
![](https://i.imgur.com/jYvPsSr.png)
![](https://i.imgur.com/oNWXheB.png)


## ICMP
- Internet Control Message Protocol
- error reporting: unreachable host, network, port, protocol
- echo request/reply (used by ping)

## IPV6
- IPv6: 128-bits IP address
- fixed-length 40 byte header
- Faster processing of IP datagram
no fragmentation allowed
![](https://i.imgur.com/zmIpcJo.png)

## 其他協議
與IP 協議配套使用的還有三個協議：
地址解析協議ARP（Address Resolution Protocol）
網際控制報文協議ICMP（Internet Control Message Protocol）
網際組管理協議IGMP（Internet Group Management Protocol）

![](https://i.imgur.com/IPWNpcx.png)

## ARP協議
地址解析協議（Address Resolution Protocol）
白話: 我知道你IP位置，給我你的MAC位置
![](https://i.imgur.com/NUJQ169.png)
為network layer 到data link layer的橋樑
其基本功能為通過目標設備的IP地址，查詢目標的MAC地址，以保證通信的順利進行。
它是IPv4網絡層必不可少的協議，不過在IPv6中已不再適用，並被鄰居發現協議（NDP）所替代。

## ICMP
- 差錯報告報文和詢問報文
- PING(看有沒有辦法通)
- 測試網路可達性
- 控制單播通信，並用於報告錯誤
- Traceroute

## NAT
https://www.stockfeel.com.tw/nat-%E4%BC%BA%E6%9C%8D%E5%99%A8%E6%98%AF%E4%BB%80%E9%BA%BC%EF%BC%9F%E5%A6%82%E4%BD%95%E9%81%8B%E7%94%A8%EF%BC%9F/
- 讓內部的虛擬IP也可以透過NAT轉換成真實世界的IP，跟外部溝通
- 重點在於內部出去的封包 / 外部進來的封包的header都會經由NAT進行轉換

> 網路位址轉譯（Network Address Translation, NAT）可以改變封包的傳送端 IP 位址與接收端 IP 位址，減少真實 IP 的使用量，也可以將私有 IP（內部 IP）改變成真實 IP（外部 IP）再傳送到網際網路，只需要向網際網路服務供應商，例如：中華電信（2412-TW）申請一個真實 IP，就可以將公司內部所有的電腦連接到網際網路了，因此 NAT 伺服器是最重要的防火牆成員，也是一種「封包過濾器（Packet filter）」。
> 
這種技術被普遍使用在有多台主機但只通過一個公有IP位址存取網際網路的私有網路中。
![](https://i.imgur.com/F6bqg2m.png)
網路位址埠轉換（NAPT）這種方式支援埠的對映，並允許多台主機共享一個公網IP位址。
![](https://i.imgur.com/GLzVFka.png)

## VPN(Virtual Private Network) 
白話: 在公有網路中，建立一個專屬的虛擬通道，大家都不知道你在裡面幹嘛，同時也會變更IP

它利用隧道協定來達到傳送端認證、訊息保密與準確性等功能
你可以將 VPN 想像成一個架設在公共網路之上的虛擬加密資料通道。連線過程當中，你的電腦會和遠端伺服器交換信任的金鑰。
若你是企業員工，那麼 VPN 可讓你從遠端存取你公司的資料中心或網路資源，就好比連上公司的區域網路 (LAN) 一樣。
而連上開放或公共的 Wi-Fi 網路會讓你的隱私和資料暴露於風險。若你希望在使用免費公共 Wi-Fi 時能獲得安全保障，VPN 是個值得信賴的選項。

## VPN VS HTTPS
https://surfshark.com/blog/vpn-vs-https
Both HTTPS and VPNs encrypt your information – but a VPN encrypts more of it.  
HTTPS can be vulnerable to some types of attacks. For example, HTTPS may not hold its own against a Root Certificate Attack  
![](https://i.imgur.com/T6Fs2aK.png)

-----


## Routing Algorithms

大架構:
以AS自治區為單元來選擇協議  
IGP(內部): RIP(Distance Vector), OSPF(Link state)  
EGP(外部): BGP  
- 企業應用以IGP為主，而EGP則通常是電信服務業者在使用。
![](https://i.imgur.com/TznouvR.png)

### 原理: Link state
白話: 
Global來看，知道所有Router的資訊，採用Dijkstra’s algorithm去找最短路徑
因為對所有人計算，收斂快，強韌度較好，但是相對的負擔也大

### 原理: Distance Vector
白話:
Decentralized，我只知道附近的鄰居給的二手資訊來做決定，
Bellman-Ford Equation (dynamic programming)
可能出現路由迴圈（Routing Loop），且錯誤會擴散出去

### 比較與選用
https://www.netadmin.com.tw/netadmin/zh-tw/technology/822A114C71954CF1AAED070C4CF260F7
- 如果每個網路區段的路由器設備相當多的話，基本上不太建議使用Link-State路由演算法，因為負擔大。  
- 混合式路由協定截長補短: EIGRP
![](https://i.imgur.com/ejCiKOU.png)
![](https://i.imgur.com/wxMZIcK.png)

### Hierarchical routing
- 每個公司/學校 RUN自己的演算法就好(Autonomous Systems, 在自治區內管自己該做的算法就好)
 
### 內部: RIP(DV)
- Base on Distance vector algorithm
- 沒有權重 只有hop  
- ** 限制HOPS最多為15(代表不能經過太多路由器，怕無限loop)**
- 由於採用DV 開銷小，但是HOPS只有15，限制了規模
- 每30秒交換(SYNC)一次(advertisement),
- Routing table放在app層，然後在ad時 路由表 透過UDP送出去
- 如果沒有advertisement超過180秒，代表link dead

### 內部: OSPF(LS)
- Base on Link State algorithm (有權重, Dijkstra)
- messages直接over ip
- 比RIP進步的地方:收斂速度快
- Security: messages authenticated
- Multiple same-cost paths allowed (only one path in RIP)

### 外部: BGP（Border Gateway Protocol，邊界網關協議）
- 外部溝通協定，讓不同AS之間可以溝通
- AS 之間的路由選擇很困難，主要是由於互聯網規模很大，各個AS 內部使用不同的路由選擇協議，無法準確定義路徑的度量；
- AS 之間的路由選擇必須考慮有關的策略，比如有些AS 不願意讓其它AS 經過。
- BGP 只能尋找一條比較好的路由，而不是最佳路由。
- 每個AS 都必須配置BGP 發言人，
- 通過在兩個相鄰BGP 發言人之間建立TCP 連接(TCP connections: BGP sessions)來交換路由信息。
- 每個BGP speaker當新的學習資訊改變了區域網路的視圖，則告訴所有其他鄰居其已經學習了什麼。這很像一個社交謠言網路，每個聽到新的謠言的人都會馬上通知所有的朋友。

![](https://i.imgur.com/NUndifm.png)
![](https://i.imgur.com/t1KiBun.png)

## Broadcast and multicast routing 

### Broadcast routing 
- deliver a packet from a source node to all other nodes
- uncontroll flooding會造成超大的流量

### Multicast routing 
– deliver a packet from a source node to a subset of other nodes



