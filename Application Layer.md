---
tags: 基礎學科, 計算機網路
---

# 計算機網路 - Application Layer

## HTTP
- 建立於TCP/IP協議之上，默認端口號80，而加密的HTTPS則是443
- HTTP無狀態-無連結(所以後續有cookie和session來解決)

## HTTP Method(GET vs. Post)
- GET就是在URL中 Query String (取得)，只是唯獨，安全
- POST則是像是放在信封message-body 比較安全 (更新)，會用到資料
- GET使用URL或Cookie傳參。而POST將數據放在BODY中
- GET的URL會有長度上的限制，則POST的數據則可以非常大(瀏覽器限制)
- GET容易被XSS跨站攻擊
- PUT/DELETE對應到 新增/刪除(比較少用)

## Restful API
https://note.artchiu.org/2017/09/30/%E5%B8%B8%E8%A6%8B%E7%9A%84http-method%E7%9A%84%E4%B8%8D%E5%90%8C%E6%80%A7%E8%B3%AA%E5%88%86%E6%9E%90%EF%BC%9Agetpost%E5%92%8C%E5%85%B6%E4%BB%964%E7%A8%AEmethod%E7%9A%84%E5%B7%AE%E5%88%A5/
- GET/PUT/POST/DELETE
- post：新增一項資料。（如果存在會新增一個新的）
- put: 新增一項資料，如果存在就覆蓋過去。（還是只有一筆資料）
- 統一命名 一致風格 用唯一的URL表達資源位置
- 無狀態，要得東西就放在GET或POST裡面
- 可快取(GET若不修改，可以GET緩存)
獲取商品資料 /GET     /items/1
新增商品資料 /POST   /items
更新商品資料 /PUT /items/1  (Idempotence)
刪除商品資料 /DELETE /items/1
每一個URI代表一種資源(意即: URL裡面不可以出現動詞)，客戶端通過四個HTTP動詞，對服務器端資源進行操作，實現"表現層狀態轉化"

## SOAP
SOAP指的是一種提供給Web Services以XML製作出來的通訊協定
不同的應用程序之間按照HTTP通信協議，遵從XML格式執行數據交換

## RPC 
一種通過網絡從遠程計算機程序上請求服務，而不需要了解底層網絡技術的協議。
RPC只是對底層協議的封裝，其實對具體的通信協議是啥並沒有太多要求
RPC協議假定某些傳輸協議的存在，如TCP或UDP，為通信程序之間攜帶信息數據。
在OSI網絡通信模型中，RPC跨越了傳輸層和應用層。
一個完整的RPC架構裡麵包含了四個核心的組件，分別是Client ,Server,Client Stub以及Server Stub，
![](https://i.imgur.com/HhNM0Cd.png)
客戶端(Client)：服務調用方。  
客戶端存根(Client Stub)：存放服務端地址信息，將客戶端的請求參數數據信息打包成網絡消息，再通過網絡傳輸發送給服務端。  
服務端存根(Server Stub)：接收客戶端發送過來的請求消息並進行解包，然後再調用本地服務進行處理。  
服務端(Server)：服務的真正提供者。  
Network Service：底層傳輸，可以是TCP 或HTTP。  

## 冪等 Idempotence
HTTP方法的冪等性是指一次和多次請求某一個資源應該具有同樣的副作用
GET 具有 Idempotence, POST則沒有

## CGI & WSGI
![](https://i.imgur.com/gJHxV25.png)
CGI是通用網關接口，是連接Web服務器和應用程序的接口，用戶通過CGI來獲取動態數據或文件等。CGI程序是一個獨立的程序，它可以使用幾乎所有語言來寫，包括perl，c，lua ，python等等。
WSGI，Web服務器網關接口，是Python應用程序或框架和Web服務器之間的一種接口


## HTTP OVER SSL & TLS
- 建構在TCP/IP之上，在傳輸資料前先交換憑證(加密方法)，使用對稱與非對稱加密封包不被攔截和偷窺
- SSL出現的話，會從http://變成https://
- TLS和SSL類型功能是一樣，TLS的早期版本是SSL，一般認為TLS較安全
- TCP三次握手後，先下來開始TLS交互，Client先跟Server要說(**TLS版本/加密算法/CA證書+公鑰**)
- Client驗證CA是否可靠，可靠的話，用公鑰將亂數加密成私鑰
- 大概後面就是交換KEY的過程
- 所有經過傳輸層傳送的封包，都被加密過，只有雙方有Key可以解
- 駭客或許能夠瞭解使用者正在連線至哪個主機，但最重要的是，無法瞭解詳細的網址。因為連線被加密，重要的資料仍將受到保護
- 不管應用層是啥，都可以配合


## HTTP 短連結與長連結 persistent connection
- 從HTTP/1.1開始默認是長連接的(keepalive or not)
- 使用同一個TCP連結(註 TCP就是一對一) 來傳送接收多個HTTP請求響應，而不是每一個請求就開一個連結
- 可叫做keep alive, HTTP1.0不支援, HTTP1.1則是默認持續連結
- 優點: 較少的CPU和RAM 缺點:像是imgur被每個使用者要求後，會持續一段時間才關(預設15秒)
- 小心很多半連結造成資源浪費，每隔一段時間送訊號給對方確認還在不在線上
- (小心SYN 半連結攻擊)
![](https://i.imgur.com/3Fq3wBP.png)


## HTTP Pipelining 管線化
- 默認情況下，HTTP 請求是按順序發出的，下一個請求只有在當前請求收到響應之後才會被發出
- 需要HTTP1.1+persistent連結+只有GET/HEAD可以管線化
- 減少延遲
- 主要還是看伺服器支不支援
![](https://i.imgur.com/axNKTLd.png)

## HTTP版本1.0/1.1
- KEEP ALIVE / Pipelining
- HTTP 1.1支持持久連接，在一個TCP連接上可以傳送多個HTTP請求和響應，減少了建立和關閉連接的消耗和延遲。
- HTTP 1.1還允許客戶端不用等待上一次請求結果返回，就可以發出下一次請求，
- http 2.0 連線多工，將 Requests/Responses 都拆碎成小 frames 進行傳輸，而這些 frames 是可以交錯的

## HTTP/2
https://ihower.tw/blog/archives/8489
- 向下相容V1.1
- 基於Google提出的SPDY協議，簡單來說就是比較快
- Request Multiplexing - 類似async的概念，同時發布多個請求
- 連線多工(Multiplexing)，在單一網路連線上，就可以同時傳輸多個 HTTP Request 和 Response，併發請求 CSS/JS/Images 等等資源。它的原理是將 Requests/Responses 都拆碎成小 frames 進行傳輸，而這些 frames 是可以交錯的，因此檔案再多也不怕，不會發生佔用網路連線(TCP connection)的情況。這就是為什麼在圖檔多的情況下，HTTP/2 特別有優勢。
- Header 壓縮，在 HTTP/1.1 的 Headers 其實是沒有壓縮的，大小佔了約 200 bytes 到 2KB 不等，而且同一瀏覽器的每個 Requests 其實絕大部份的 Headers 都是重複的。HTTP/2 用了 HPACK 壓縮技術，大大減少每次都要重複傳輸一樣的 Headers。
- 主動推送功能

## Cookie
- 因為http是無狀態的stateless
- Cookie 是服務器發送到用戶瀏覽器並保存在本地的一小塊數據
- cookie / httponly(怕xss)- 作用domain
- 敏感訊息不應該透過cookie存放-會被改

## Cookie 與Session 選擇
- Cookie存客戶端/Session存服務器端，不過Cookie可以拿來存SessionID，兩者都是為了追蹤會話，Session較安全


- 除了可以將用戶信息通過Cookie 存儲在用戶瀏覽器中，也可以利用Session 存儲在服務器端，存儲在服務器端的信息更加安全
使用Session 維護用戶登錄狀態的過程如下：
用戶進行登錄時，用戶提交包含用戶名和密碼的表單，放入HTTP 請求報文中；
服務器驗證該用戶名和密碼，如果正確則把用戶信息存儲到Redis 中，它在Redis 中的Key 稱為Session ID；
服務器返回的響應報文的Set-Cookie 首部字段包含了這個Session ID，客戶端收到響應報文之後將該Cookie 值存入瀏覽器中；
客戶端之後對同一個服務器進行請求時會包含該Cookie 值，服務器收到之後提取出Session ID，從Redis 中取出用戶信息，繼續之前的業務操作。
- 應該注意Session ID 的安全性問題，不能讓它被惡意攻擊者輕易獲取，那麼就不能產生一個容易被猜到的Session ID 值。此外，還需要經常重新生成Session ID。
- 在對安全性要求極高的場景下，例如轉賬等操作，除了使用Session 管理用戶狀態之外，還需要對用戶進行重新驗證，比如重新輸入密碼，或者使用短信驗證碼等方式
- Cookie 只能存儲ASCII 碼字符串，而Session 則可以存儲任何類型的數據，因此在考慮數據複雜性時首選Session；
- Cookie 存儲在瀏覽器中，容易被惡意查看。如果非要將一些隱私數據存在Cookie 中，可以將Cookie 值進行加密，然後在服務器進行解密；
- 對於大型網站，如果用戶所有的信息都存儲在Session 中，那麼開銷是非常大的，因此不建議將所有的用戶信息都存儲到Session 中。

## Apache vs. Nginx
- Nginx
效能好(大流量)，佔用更少的記憶體及資源，
抗併發，nginx處理請求是非同步非阻塞的，異步I/O
nginx是非同步的，多個連線（萬級別）可以對應一個程序
處理**靜態資源**有優勢(靜態泛指CSS JS HTML等...)
> Nginx uses asynchronous, non-blocking event-driven architecture.

- Apache
穩定，但是是同步模型，一個連線對應一個程序
處理**動態**有優勢，rewrite等，(泛指資料庫相關)
> Apache uses processes for every connection (and with worker mpm it uses threads). As traffic rises, it quickly becomes too expensive.

- 選擇
需要效能的web 服務，用nginx
如果不需要效能只求穩定，那就apache吧

## 狀態碼
1xx報告	接收到請求，繼續進行
2xx成功	步驟成功接收，被理解，並被接受
3xx重新設定	為了完成請求，必須採取進一步措施
4xx客戶端錯誤	請求包括錯的順序或不能完成
5xx服務器出錯	服務器無法完成明確有效的請求
403：禁止訪問404：未找到

## CSRF
> 跨站**請求**
- 跨站攻擊，舉例: 你點下按鈕時，幫你刪掉你部落格的文章
- 解法: TOKEN(最常見，亂數TOKEN)，POST(會被破解)，驗證碼(煩人)，檢查refer url(會被破解)
- 使用者送出post時 需要一起送出TOKEN以供驗證

## XSS
> XSS（跨站點腳本）跨站**腳本**攻擊
- 舉例 在留言區插入html/js
- 解法: 過濾使用者輸入內容(白名單)
- 會被插入一些程式碼
- 將重要的cookie標記為http only, 這樣的話js 中的document.cookie語句就不能獲取到cookie了
- 只允許使用者輸入我們期望的資料。 例如：age使用者年齡只允許使用者輸入數字，而數字之外的字元都過濾掉
- 過濾js事件的標籤。例如 "onclick=", "onfocus" 等等

## SQL INJECTION
- 不要拼接SQL字串，使用內建lib func等，把args傳進去
- 基本上常見的lib用法都有防此
- 善用cursor.execute(語句, 引數)
- 採用ORM

## 中間人攻擊
https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB
- 中途攔截封包
- SSL可以解決


## What happend after client and server?

- 基於網路模型
輸入url後，先看這個url有沒有在本地host的DNS(遞迴查詢), 沒有就用UDP問 DNS server，會得到一個IP
**DNS可能壞掉**
基本上server端和client端都是http協議(應用層)，
client這邊會發出http請求，有header+body，包括請求的方法（GET/POST）、目標url、遵循的協議（http/https/ftp..）3
這些資訊會透過網路模型，從應用層-->傳輸層-->網路層-->鏈路層(data link)-->物理層
application layer的info是要通過transport layer進行通訊的，其資料為了確保不會掉，
會經過TCP的協議，先拆成封包，透過ACK, SYNC後開始進行三次握手(如果是https 有TLS/SSL 就是握手完後傳輸金鑰)
**握手過程傳輸金鑰可能出問題**
握手完後，會透過滑動框口進行流量及壅塞控制，確保盡量不掉包(掉包就重傳)
這些封包會繼續傳到network layer 進行傳輸
在network裡面，會有router幫忙轉送封包，這邊pass的過程就叫做routing algorithm
有像是RIP(distance vector)/ OSPF(link state)等路由協議
**Router可能壞掉**
在往下就會到Datalink，透過ARP協議（Address Resolution Protocol）將IP換成MAC解析
最後就是physical的部分，會將這些封包根據IEEE轉換成高低電平透過網路線去傳輸
**網路線也可以被干擾**
接下來會往反向上傳到對方的應用層


- 基於應用層
http是無狀態協議，大概就是Get post等等方法
那我傳過去的話可能會先碰到nginx/apache- 在到uwsgi - 最後到web app(django/flask等等)
web app處理一些request後，再包成response給nginx發出去

---- 

推薦：https://www.zhihu.com/question/34873227/answer/518086565

1，客戶端瀏覽器通過DNS解析到www.baidu.com的IP地址202.108.22.5，通過這個IP地址找到客戶端到服務器的路徑。客戶端瀏覽器發起一個HTTP會話到202.108.22.5，然後通過TCP進行封裝數據包，輸入到網絡層。

2，在客戶端的傳輸層，把HTTP會話請求分段報文段，添加源和目的端口，如服務器使用80端口監聽客戶端的請求，客戶端由系統隨機選擇一個端口如5000，與服務器進行交換，服務器將相應的請求返回給客戶端的5000端口。然後使用IP層的IP地址查找目的端。

3，客戶端的網絡層不用擔心應用層或傳輸層的東西，主要做的是通過查找路由表確定如何到達服務器，期間可能通過多個路由器，這些都是由路由器來完成的工作，我不作過多的描述，無非就是通過查找路由表決定通過那個路徑到達服務器。

4，客戶端的串口層，包通過互連層發送到路由器，通過鄰居協議查找給定IP地址的MAC地址，然後發送ARP請求查找目的地址，如果得到回應後就可以使用ARP的請求應答交換的IP數據包現在就可以傳輸了，然後發送IP數據包到達服務器的地址。

事件順序：

瀏覽器獲取輸入的域名www.baidu.com
瀏覽器向DNS請求解析www.baidu.com的IP地址
域名系統DNS解析出百度服務器的IP地址
瀏覽器與該服務器建立TCP連接（默認端口號80）
瀏覽器發出HTTP請求，請求百度首頁
服務器通過HTTP響應把首頁文件發送給瀏覽器
TCP連接釋放
瀏覽器將首頁文件進行解析，放入Web頁顯示給用戶。
涉及到的協議：

1，應用層：HTTP（www訪問協議），DNS（域名解析服務）DNS解析域名為目的IP，通過IP查找服務器路徑，客戶端向服務器發起HTTP會話，然後通過運輸層TCP協議封裝數據包，在TCP協議基礎上進行傳輸。

2，傳輸層：TCP（為HTTP提供可靠的數據傳輸），UDP（DNS使用UDP傳輸）HTTP會話會被分段報文段，添加源，目的端口； TCP協議進行主要工作。

3，網絡層：IP（IP數據包傳輸和路由選擇），ICMP（提供網絡傳輸過程中的差錯檢測），ARP（將本機的網關設置為IP地址映射成物理MAC地址）為數據包選擇路由，IP協議進行主要工作，相鄰結點的可靠傳輸，ARP協議將IP地址轉成MAC地址。


