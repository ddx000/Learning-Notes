# 計算機網路 - Network Layer

## 概論
- Network layer是以router和IP組成
- router會分辨IP datagram要去哪裡(使用Routing table查表)
- 到達目的地後，再把IP address的header去掉 把segment給transport layer
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
- 跑算法(routing)-->做表-->分送(forwarding)
- Run routing algorithms/protocol (RIP, OSPF, BGP) -->做出表
- 做出表後 -->forwarding datagrams from incoming to outgoing link
- 困難點: 大量封包交換
- 進出速度不一致時要buffering (priority Queue)

## IP datagram format
- Version/head/sourceip/destinationip
- IP fragmentation & reassembly (IP太大可以切 但是HEAD要COPY)
- time to alive(-到0 繞不出去 丟掉)
- 記是UDP還是TCP
- Header checksum

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

## CIDR (任意切IP，不用classA-D這個強硬切法)

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

## Routing Algorithms
- 路由協定可以分成內部路由協定和外部路由協定兩大類型
![](https://i.imgur.com/B6iTd1A.png)
- 主要概念來自於有權重的向量圖，有三個重點算法如下:
- 先講內部路由協定
- Link state
- Distance Vector
- Hierarchical routing

### Link state
- Global，我要知道所有人
- Dijkstra’s algorithm
- 強韌度較好

### Distance Vector
- Decentralized，我只知道附近的
- Bellman-Ford Equation (dynamic programming)
- 錯誤會擴散出去

### 比較

(詳細比較如下)
https://www.netadmin.com.tw/netadmin/zh-tw/technology/822A114C71954CF1AAED070C4CF260F7

Link-State: 複雜，算法和空間複雜度高，因為是針對全部人做計算，反應快速收斂
Distance Vector: 簡單，但是可能出現路由迴圈（Routing Loop）

以比較非專業的說法則是, Link State Router 會蒐集來自其他 Routers 的 "第一手" 資訊, 並據此 "獨立自主" 地做出判斷 (去哪個目的地怎麼走), 而 Distance Vector Router 則以鄰居給的二手資訊來做決定, 兩者優劣立見!

![](https://i.imgur.com/ejCiKOU.png)

![](https://i.imgur.com/wxMZIcK.png)

- 選用 
- 如果每個網路區段的路由器設備相當多的話，基本上不太建議使用Link-State路由演算法，因為LSA封包的更新在路由器設備數目很高的情況之下，很可能造成一定的負擔。  
- 如果路由器設備的硬體規格（CPU和記憶體等等）還算不錯的話，可以考慮使用Link-State路由演算法。
- 混合式路由協定截長補短  
EIGRP

### Hierarchical routing
- 每個公司/學校 RUN自己的演算法就好(Autonomous Systems, 在自治區內管自己該做的算法就好)
 
## 路由協定比較(IGP & EGP)

- 路由通訊協定可分為IGP（Interior Gateway Protocols，也可稱為Interior Routing Protocols）
- 及EGP（Exterior Gateway Protocols，也可稱為Exterior Routing Protocols）等2種，
- 企業應用以IGP為主，而EGP則通常是電信服務業者在使用。
- IGP包括
- RIP（Routing Information Protocol）IGRP、EIGRP....... IGP中RIP最常用
![](https://i.imgur.com/TznouvR.png)

## Intra-AS Routing RIP / OSPF / IGRP(Cisco)
- Interior Gateway Protocols (IGP)

### RIP
- Base on Distance vector algorithm
- 沒有權重 只有hop
- 限制HOPS最多為15(代表不能經過太多路由器
- 每30秒交換(SYNC)一次(advertisement),
- Routing table放在app層，然後在ad時 路由表 透過UDP送出去
- 如果沒有advertisement超過180秒，代表link dead

### OSPF
- Base on Link State algorithm (有權重, Dijkstra)
- messages直接over ip
- 比RIP進步的地方:
- Security: messages authenticated
- Multiple same-cost paths allowed (only one path in RIP)

## Internet inter-AS routing: BGP
- 外部溝通協定，讓不同AS之間可以溝通
- BGP（Border Gateway Protocol，邊界網關協議）
- AS 之間的路由選擇很困難，主要是由於互聯網規模很大，各個AS 內部使用不同的路由選擇協議，無法準確定義路徑的度量；
- AS 之間的路由選擇必須考慮有關的策略，比如有些AS 不願意讓其它AS 經過。
- BGP 只能尋找一條比較好的路由，而不是最佳路由。
- 每個AS 都必須配置BGP 發言人，通過在兩個相鄰BGP 發言人之間建立TCP 連接(TCP connections: BGP sessions)來交換路由信息。
每個BGP speaker當新的學習資訊改變了區域網路的視圖，
則告訴所有其他鄰居其已經學習了什麼。
這很像一個社交謠言網路，每個聽到新的謠言的人都會馬上通知所有的朋友。

![](https://i.imgur.com/NUndifm.png)
![](https://i.imgur.com/t1KiBun.png)

## Broadcast and multicast routing 

### Broadcast routing 
- deliver a packet from a source node to all other nodes
- uncontroll flooding會造成超大的流量
- 

### Multicast routing 
– deliver a packet from a source node to a subset of other nodes



