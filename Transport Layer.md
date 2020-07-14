---
tags: 基礎學科, 計算機網路
---

# 計算機網路- 傳輸層Transport Layer
傳輸層(TCP UDP)提供了網路層(IP)和應用層(HTTP)之間的交互
封裝了網路層複雜的概念，對應用層來講，就像一個管道Socket

## TCP與UDP
TCP: 有連線 -僅有兩方握手 - 資料不會掉，在不可靠的情況下提供可靠
UDP: 無連線 - 快 - 資料可能會掉，支援廣播和多播

## TCP機制
- TCP不保證我送出去一定會接收到，只是接收不到會說沒接到，就再傳一次
- 超時重傳(ROT微大於RTT)
- 校驗和
- 滑動窗口
- 流量控制(看接收方Buffer滿不滿，太滿就不送，讓接收方來得及收)
- 壅塞控制(網路塞，就會調降速度，控制整個流量速度): 如果不控制會一直重傳會越來越擁擠


## TCP之滑動窗口
https://www.itread01.com/articles/1476615670.html
- 滑動窗口是流量控制的一種方式，發送和接收端有兩個窗口，
- 藉由接收端告訴發送端窗口有多大而控制
- TCP發送端窗口會分成四類
- 1. 已經發送且確認 2.已經發送未確認 3.允許發送未發送 4.不允許發送且未發送
![](https://i.imgur.com/dCc0QZn.png)
- 引進滑動窗口的好處：原本是一包一包傳，但是沒效率，滑動窗口可以保證"順序"，將亂序的先保存起來，等他重傳
- 多個報文段一起回ACK


## TCP連結(三次握手，四次握手)
- TCP是在不可靠的network層上封裝一層機制想辦法讓他看起來可靠(timeout/重傳)
- 第一次 客戶端 SYN=1 SEQ =X
    - 客戶端通過向服務器端發送一個SYN來創建一個主動打開，作為三次握手的一部分。客戶端將其連接的序號設置為隨機數X
- 第二次 主機端 SYN=1 ACK=1 acknum = X+1 SEQ=y
    - 服務器端正確為回送SYN /ACK。ACK的確認碼應為X + 1，SYN / ACK包本身又有一個隨機序號Y。
- 第三次 客戶端 ACK=1 ACKnum = y+1
    - 客戶端再發送一個ACK。當服務端受到這個ACK的時候，就完成了三路握手
![](https://i.imgur.com/RhsmUSW.png)

- 連結拆除- 四次握手
    - 注意：中斷連接端可以是客戶端，也可以是服務器端。
- 客戶 第一次 FIN=1 , SEQ=X
    - 客戶端發送一個數據分段，其中的FIN標記設置為1。客戶端進入FIN-WAIT狀態。該狀態下客戶端只接收數據，不再發送數據
- 主機 第二次 ACK=1 , acknum = x+1 
    - 服務器接收到包含FIN = 1的數據分段，發送帶有ACK = 1的剩餘數據分段，確認收到客戶端發來的FIN信息
- 主機準備進去關閉連結狀態，關好後開始第三次握手
- 主機 第三次 FIN=1 , SEQ=Y
- 客戶 第四次 ACK=1 , acknum = y+1
- 第四次握手完後，會等兩段最長周期MSL，確認主機端關了才close

![](https://i.imgur.com/AzZ0V2P.png)




## Demultiplexing(解多工)與Multiplexing(多工)

- Demultiplexing: 傳輸層到應用層的socket
- Multiplexing: 應用層socket到傳輸層 segment = (header+ content)


## SYN攻擊
- 握手握一半，大量偽造假IP，發送SYN第一次握手
- 伺服器就對等待連結...(DDOS)
- 解決: 縮短超時時間

## RDT Reliable Data Transfer 可靠資料傳輸
- 例如接收端在等待編號0的封包，結果收到封包1，此時會回傳ACK1給來源端，而正在等候ACK0的來源端
- 收到ACK1 表示封包0可能遺失，所以會再重送封包0。
- 另外在傳送端多了倒數計時器，封包送出去如果超過時間仍未收到ACK或是收到不正確編號的ACK，則再送出封包一次
- Stop-and-Wait 太慢了
- 所以改用Pipelined Protocol 一次丟很多封包，傳輸端與接收端 都必須增加封包的暫存空間與序列號碼
- Go-Back-N(GBN) 如果要N，跳號N+1，就把N後面全部丟掉
- Selective Repeat(SR)，用滑動窗口只接收沒接到的

## Flow Control & Congestion Control
- buffer size快滿了就不硬丟 會回傳可用buffer size
- 如果網絡出現擁塞，分組將會丟失，此時發送方會繼續重傳，從而導致網絡擁塞程度更高。因此當出現擁塞時，應當控制發送方的速率。這一點和流量控制很像，但是出發點不同。流量控制是為了讓接收方能來得及接收，而擁塞控制是為了降低整個網絡的擁塞程度。
![](https://i.imgur.com/wsRI25J.png)



## 慢開始
https://github.com/CyC2018/CS-Notes/blob/master/notes/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%20-%20%E4%BC%A0%E8%BE%93%E5%B1%82.md
- 一開始cwindow設定1，然後才能2 4 8 16...指數成長
- 會設定一個門檻為壅塞避免門檻，之後就放慢成長
- 如果一直超時，門檻會越來越低，成長速度就會放慢
- (確保不容易超時)

## 快重傳
- 如果只是掉一個封包而已，不是網路壅塞，我會讓cwindows設成門檻值
- 總結，慢開始代表網路壅塞，cwin設成1，快恢復代表掉一個封包，cwin設為ssthresh

![](https://i.imgur.com/mnAfKe3.png)
