---
tags: web
---


# Web Server

## WSGI
wsgi(Web Server Gateway Interface)
Python APP 與WEB SERVER的溝通橋梁

Web Server負責接收瀏覽器請求，透過wsgi跟python溝通
middle ware則負責類似url routing的功能

- 常見django flask都有內建wsgi server 但是通常都是debug用，真正部屬上去時，建議換成gunicorn，再根據需求，包上nginx or apache

[flask-flow](https://www.maxlist.xyz/2020/05/06/flask-wsgi-nginx/)
![](https://i.imgur.com/4VTeJ0m.png)
1. 使用 Apache / Nginx 擇一做為反向代理伺服器：負責靜網頁與動態網頁請求和結果的回覆
2. 使用 gunicorn / uWSGI 擇一做為 WSGI 伺服器：負責接收代理伺服器的請求後，轉發給 Flask 以及接收 Flask 返回信息轉發給 Nginx
3. Flask 收到請求後處理數據並返回頁面給 uWSGI 伺服器

![](https://i.imgur.com/Q0lpecX.png)
uwsgi性能較高 / 比內建的wsgiref好很多

## 如果使用內建wsgi
single preocess, 掛掉後不會重啟, 只適合Debug, 對請求處理很弱

## gunicorn(WSGI HTTP SERVER)
gunicorn就能達到mutiprocess + 設定worker數量等等, 掛掉後可以自動重啟
pre-forking: 每個requests產生一個worker去處理(所以外面建議加上一個反向代理)
- 為什麼需要 gunicorn / uWSGI?
因為 Nginx 沒辦法實現 WSGI 規範，所以如果不使用Gunicorn，將使用框架 自帶的 WSGI Server
同等地位: uWsgi(配置較複雜，性能較高)

## 為何外面還要 nginx?
- 提高性能，例如處理靜態資源，負載均衡，反向代理等等，下面解釋


# Apache vs. Nginx
- Nginx
效能好(大流量)，佔用更少的記憶體及資源，
抗併發，nginx處理請求是非同步非阻塞的，異步I/O
nginx是非同步的，多個連線（萬級別）可以對應一個程序
處理**靜態資源**有優勢(靜態泛指圖片等...)
> Nginx uses asynchronous, non-blocking event-driven architecture.

- Apache
穩定，但是是同步模型，一個連線對應一個程序
處理**動態**有優勢，rewrite等，(泛指資料庫相關)
> Apache uses processes for every connection (and with worker mpm it uses threads). As traffic rises, it quickly becomes too expensive.

- 選擇
需要效能的web 服務，用nginx
如果不需要效能只求穩定，那就apache吧
 
## rewrite
偽靜態頁面是指動態網頁通過重寫URL的方法實現去掉動態網頁的參數,但在實際的網頁目錄中並沒有必要實現存在重寫的頁面。
https://zh.wikipedia.org/wiki/URL%E9%87%8D%E5%AF%AB
- SEO / 隱藏參數 / 或定期抓取動態網頁轉成靜態
```
http://www.somebloghost.com/Blogs/Posts.php?Year=2006&Month=12&Day=10
rewrite
http://www.somebloghost.com/Blogs/2006/12/10/
```
URL Rewrite：當伺服器收到請求時發現這段URL已經異動過，伺服器會幫你訪問異動後的URL，再將結果回傳給瀏覽器。  
URL Redirect：當伺服器收到請求時發現這段URL已經異動過，伺服器會將異動後的URL回傳給瀏覽器(301 or 302)，再由瀏覽器重新發送請求至異動後的URL。



# Nginx

如果只是單純的API，就不用額外包一層NGINX
[Nginx、Gunicorn在服务器中分别起什么作用？](https://www.zhihu.com/question/38528616)
Nginx 可以提高 Web Server 性能
1. 緩存，可以減少 response 的等待時間，如果網頁服務上有大量的靜態檔案(如：CSS、圖片檔、JS 檔案)，設置 Cache 可以讓瀏覽器就會緩存這些文件，節省網路流量的費用，也可以讓使用者訪問網站會顯得更快。
2. 負載均衡(反向代理，client不知道真正的server是誰，把請求轉發給各個伺服器)
3. 靜態文件檔案
3. 抗並發能力(連線限制等等)



# 正向代理與反向代理

https://kknews.cc/zh-tw/tech/k66p2gb.html

![](https://i.imgur.com/iqN3dfN.png)
![](https://i.imgur.com/Lpr8Wco.png)

不論是正向代理和反向代理，都能提高速度
正向代理，是指代理"客戶端"去送出請求，伺服器會不知道誰是真的客戶端
反向代理，是指代理"伺服器"去接收請求，客戶端會不知道誰是真的伺服器

正向代理:
突破訪問限制，提高速度

反向代理:
隱藏伺服器真實IP
負載均衡-->轉發需求給不同的真實伺服器
提供安全保障