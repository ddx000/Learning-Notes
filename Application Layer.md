# 計算機網路 - Application Layer

## HTTP
- 建立於TCP/IP協議之上，默認端口號80，而加密的HTTPS則是443
- HTTP無狀態-無連結(所以後續有cookie和session來解決)

## HTTP Method
- GET就是在URL中 Query String (取得)，只是唯獨，安全
- POST則是像是放在信封message-body 比較安全 (更新)，會用到資料
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
更新商品資料 /PUT /items/1 
刪除商品資料 /DELETE /items/1


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


## CSRF
- 跨站攻擊，舉例: 你點下按鈕時，幫你刪掉你部落格的文章
- 解法: TOKEN(最常見，亂數TOKEN)，POST(會被破解)，驗證碼(煩人)，檢查refer url(會被破解)
- 使用者送出post時 需要一起送出TOKEN以供驗證

## XSS
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