---
tags: database
---


# SQL數據庫系統

## ACID
A: Atomic，一次做完或者rollback
C: Consistency 一致性
數據庫在事務執行前後都保持一致性狀態。
> constarint不能被改變
例如: 約束了A+B = 10，A4B6，改動A6的話，B必須改成4

I: Isolation 隔離性
資料庫允許多個並發事務同時對其數據進行讀寫和修改的能力
- 主要是解決"並發"下必須保持隔離性，因為不保持隔離性會有很多問題

> 隔離性由並發事務所作的修改必須與任何其它並發事務所作的修改隔離。事務查看數據時數據所處的狀態，要麼是另一併發事務修改它之前的狀態，要麼是另一事務修改它之後的狀態，事務不會查看中間狀態的數據

D: 持久性

## 如果沒有隔離性 會有那些問題

- 丟失修改: A和B都改同一個數字
- 幻影讀(不可重複讀) B讀到A正在修改的數據，讀了兩次
- 讀贓數據(讀到中間的) B讀到A中間的數據，最後A搞不好rollback了



## 樂觀鎖和悲觀鎖
https://www.cnblogs.com/kismetv/p/10787228.html

> 樂觀: 提交時我檢查是否被更改，有修改就失敗 // 悲觀: 進去就加鎖

- 樂觀鎖: 我認為你不會修改數據，所以只在提交時，檢查是否有人修改，若有人修改過，我就放棄此事務(使用版本號 CAS Compare And Swap )
> 樂觀鎖本身是不加鎖的，只是在更新時判斷一下數據是否被其他線程更新了

- 悲觀鎖: 我覺得你會修改數據，所以我一進去就加鎖
> 悲觀鎖的實現方式是加鎖，加鎖既可以是對代碼塊加鎖（如Java的synchronized關鍵字），也可以是對數據加鎖（如MySQL中的排它鎖）
- 如果高併發，適合悲觀鎖，因為樂觀鎖會一直失敗(大家都搶著修改，一直失敗)，一般不是高併發的情況下，適合樂觀鎖(因為衝突機會少，加鎖是需要資源的)
 
## 封鎖粒度: 行級鎖以及表級鎖(MYSQL)
- InnoDB支持行級鎖(row-level locking)和表級鎖,默認為row-level locking
- 在選擇封鎖粒度時，需要在鎖開銷和並發程度之間做一個權衡。
- 應該盡量只鎖定需要修改的那部分數據，而不是所有的資源。鎖定的數據量越少，發生鎖爭用的可能就越小，系統的並發程度就越高。
- 因此封鎖粒度越小，系統開銷就越大。
### 行級鎖
- 一次只鎖一個row，優點: 併發程度高 缺點:系統開銷大，加鎖慢
行級鎖是mysql中鎖定粒度最細的一種鎖，表示只針對當前操作的行進行加鎖。行級鎖能大大減少數據庫操作的沖突。其加鎖粒度最小，但加鎖的開銷也最大
### 表級鎖
- 一次鎖一個table 優點: 系統開銷小 缺點: 併發程度低

## 封鎖類型:
互斥鎖（Exclusive），簡寫為X 鎖，又稱寫鎖。  
- 加互斥鎖後，就我獨佔，其他人都不能加鎖  
共享鎖（Shared），簡寫為S 鎖，又稱讀鎖。
- 大家都可以加S鎖 加上去後，大家都可以讀
- 加鎖期間其它事務能對A 加S 鎖，但是不能加X 鎖

## MYSQL遇到死鎖:
持有最少X鎖的，釋放資源


## MySQL架構:
**連結層**: 處理socket, thread,connection pool, ssl,tcp/ip等客戶連結服務的問題
**處理層**: 處理sql指令, 有parser去驗證是否syntax error, 順利的話會丟給optimizer, opt的原理是搜尋最佳化(屬性投影，白話就是要的才取，不是取完全部再篩)，sql有cache and buffer(有hit 就不用進去engine)
cache搜尋結果: 這樣下次一樣的指令不用再進到engine(原理: 兩個hash value comparsion)

**Engine層**: innodb等 可抽換
**數據存儲層**: disk


## 索引原理:

- 優點:
提高速度，保證unique value, 原理balance tree
- 缺點:
占空間: time& space trade off
耗效能: 如果後續修改資料，維護index都需要效能

- 建立索引:
foreign key, 常常select/ 需要sort/ where的

- 不該建立索引:
很少查 / 類型category的欄位 / 當修改性能遠遠大於檢索性能時，不應該創建索引

- 創建索引的方法：直接創建和間接創建（在表中定義pk時，pk就是index)

- MYSQL語法:
- https://medium.com/@michael80402/mysql%E7%B4%A2%E5%BC%95-e002f707a5f4
#### index 語法
```
CREATE TABLE member (
id      INT UNSIGNED PRIMARY KEY,
name    VARCHAR(20),
email   VARCHAR(36) UNIQUE KEY,)

CREATE INDEX email_index ON member(email);
ALTER TABLE member ADD INDEX(email);
```

## 主鍵 Primary Key
PRIMARY KEY 有點類似 UNIQUE 加上 NOT NULL
Pk是一種 index 但不能為空值(NULL)，PK 會自動建立 index

## 外來鍵 Foreign key
**適用情境：用來存放來別張資料表的資料主鍵，透過JOIN合併資料**
常見的外來鍵參考對象的刪除/更新規則：
- 連鎖反應做法 CASCADE:
來源的資料被刪除了，有關聯來源的資料也會一併刪除
如果是 parent FK更新了新值，則child 對應的FK會更新。
- 限制性做法 RESTRICT:
有關聯來源的資料如果沒有被刪除，來源的資料無法刪除
- 虛值化做法 SET NULL:
有關聯來源的資料如果沒有被刪除，來源的資料刪除後，有關聯來源的資料值將被修改為NULL
```
CREATE TABLE ORDERS
(Order_ID integer,
Order_Date date,
Customer_SID integer,
Amount double,
PRIMARY KEY (Order_ID),
FOREIGN KEY (Customer_SID) REFERENCES CUSTOMER (SID));

ALTER TABLE ORDERS
ADD FOREIGN KEY (Customer_SID) REFERENCES CUSTOMER (SID);
```

## 組合鍵 (Composite Key)
```
CREATE TABLE customer (
  C_Id INT NOT NULL,
  Name VARCHAR(50) NOT NULL,
  Address VARCHAR(255),
  Phone VARCHAR(20),
  CONSTRAINT pk_Customer_Id PRIMARY KEY (C_Id, Name)
);
```
我們限制 C_Id 及 Name 這兩個欄位為主鍵，CONSTRAINT 後面接著的即是此主鍵的名稱。
當主鍵包含多個欄位時，我們稱之為組合鍵 (Composite Key)。


## SQLite
1. pd.read_csv超快 甚至比read_sql快很多
2. sqlite的優勢是select, 當資料越來越多時，pandas的做法是先進來後用loc或filter再去篩
3. 當未來資料越來越多時，一定需要**select的功能，不可能把全部的csv to df放進memory，很容易爆掉**
4. csv雖然可以分檔，但是要畫長時間圖時，只有sql才有辦法做到這件事情
![](https://i.imgur.com/zTBMs1D.png)
5. MYSQL是基於服務器，SQLite則是無服務器 輕量
6. SQLite不可以同時寫入(保持單線程

## ORM(Object Relational Mapping)
- 優點
1. 基本上就是物件導向的語法，開發快
2. 可以接任何接口(高階API)，方便轉移
3. 防止SQL注入攻擊
- 缺點
1. 不是原生 效能差 可自訂少 複雜的還是要自己寫


## 時序資料庫(TSDB:time series databases)
https://www.itread01.com/content/1545502264.html
舉例: InfluxDB
時序列資料庫（Time series database）：用來儲存時序列（time-series）資料並以時間建立索引
資料結構簡單：某一度量指標在某一時間點只會有一個值，沒有複雜的結構（巢狀、層次等）和關係（關聯、主外來鍵等）
資料量大：由於時序列資料由所監控的大量資料來源來產生、收集和傳送，比如主機、IoT裝置、終端或App等
時序資料的幾個特點
（1）基本上都是插入，沒有更新的需求。
（2）資料基本上都有時間屬性，隨著時間的推移不斷產生新的資料。
（3）資料量大，每秒鐘需要寫入成千萬上億條資料


## MYISAM vs INNODB
> MyISAM 表鎖 / INNODB 行鎖 
> INNODB併發和更新較好 /MYISAM讀取查詢較快(適合大量查詢)
> 目前INNODB常用，因為他支持ACID特性等等 詳情請見

MyISAM 適合於一些需要大量查詢的應用，但其對於有大量寫操作並不是很好。
甚至你只是需要update一個字段，整個表都會被鎖起來，
另外，MyISAM 對於SELECT COUNT(*) 這類的計算是超快無比的。

InnoDB 的趨勢會是一個非常複雜的存儲引擎，對於一些小的應用，它會比MyISAM還慢。
他是它支持“行鎖” ，於是在寫操作比較多的時候，會更優秀。並且，他還支持更多的高級應用，比如：事務。
https://segmentfault.com/a/1190000008227211


## 語法 JOIN

```
###INNER JOIN
SELECT a.runoob_id, a.runoob_author, b.runoob_count 
FROM runoob_tbl a INNER JOIN tcount_tbl b ON a.runoob_author = b.runoob_author;

SELECT * FROM TableA INNER JOIN TableB ON TableA.name = TableB.name

SELECT a.*, b.* FROM `test1` as a, `test2` as b where a.id = b.id

###LEFT JOIN

SELECT a.*, b.* FROM `test1` as a LEFT JOIN `test2` as b on a.id = b.id


### FULL OUT JOIN
SELECT * FROM TableA FULL OUTER JOIN TableB ON TableA.name = TableB.name

```

## MYSQL VIEW


例如有兩個sql table和很複雜的select語句
```
create view <view_name> as SELECT ........... JOIN ....
```
這樣之後就可以簡化操作


> 簡化複雜的SQL 操作，比如復雜的連接；
> 只使用實際表的一部分數據；
> 通過只給用戶訪問視圖的權限，保證數據的安全性；
> 更改數據格式和表示。

Q：為什麼要使用視圖？
A：因為視圖的諸多優點，如下
1）簡單：使用視圖的用戶完全不需要關心後面對應的表的結構、關聯條件和篩選條件，
對用戶來說已經是過濾好的複合條件的結果集。
2）安全：使用視圖的用戶只能訪問他們被允許查詢的結果集，對表的權限管理並不能限製到某個行某個列，但是通過視圖就可以簡單的實現。
3）數據獨立：一旦視圖的結構確定了，可以屏蔽表結構變化對用戶的影響，
源表增加列對視圖沒有影響；源表修改列名，則可以通過修改視圖來解決，不會造成對訪問者的影響。
總而言之，使用視圖的大部分情況是為了保障數據安全性，提高查詢效率。


## 正規化概念
### 1NF
定義主鍵值 Primary Key，資料都變成單一值
### 2NF
![](https://i.imgur.com/UQiwY2V.png)
許多欄位如item, user, shop, shop_address等欄位都開始出現重複，就要拉出去再開一個table，設foreign key
### 3NF
像是total = price * quantity這個total就不該出現columns中，到時候再計算就可以了，要減少相依性


> 第一正規化：定義主鍵值（primary key或unique key）以及剔除重複資料，來打好資料表的基礎
> 第二正規化：符合第一正規化，釐清資料表裡面每一個資料欄位的關係，把部分相依的資料另開表格作儲存，來確保每一非鍵值欄位必須「完全功能相依」(Functional Dependency)於主鍵
> 第三正規化：符合第二正規化，且每一個非鍵值欄位都必須不得和其他非鍵值欄位產生相關性
> 
> 而正規化最重要的目的，就是重新整裡關聯式資料庫的表格及欄位，來降低資料重複及減少相依性的過程。而這個過程中，對於減少資料庫空間上的浪費，以及強化維護與操作上的效率和正確性，都有很大的幫助。
> 另外值得一提的就是，基本上執行正規化就等於是原本的Table中的column會變少，而用新的Table 表來表示相關性，故一定會增加Join的次數與SQL指令的複雜度。


---


## NOSQL(MongoDB)
- NO schema NOT ONLY SQL
- JSON LIKE
- flexible database used for big data & real-time web apps
- a collectnios contain every thing in one query
- NO relations (NO TABLES)
![](https://i.imgur.com/3Ua0jaq.png)


![](https://i.imgur.com/b5KoXIt.png)
優點:要query的資料盡量塞在一起，減少relation，增加查詢速度
可以處理大量資料，scaling is easy!


缺點:沒有預先定義schema，重複的資料(因為沒有關聯)，沒有ACID

 
SQL非常困難去 Horizontal Scaling(分散式系統)
只能vertical scaling(增加硬體設備)

NOSQL因為資料stoarge在同一個collections, (雖然有重複)
比較好horizontal scaling
![](https://i.imgur.com/Y8ZCtQg.png)

如果有相關的資料存在NOSQL，這樣更新起來非常慢(因為許許多多collections裡面都有重複...)
![](https://i.imgur.com/xsbvTMM.png)

![](https://i.imgur.com/4ogDosa.jpg)


## MONGODB CRASH COURSE
https://www.youtube.com/watch?v=pWbMrx5rVBE
![](https://i.imgur.com/V3aAkFd.png)
![](https://i.imgur.com/YXWkvPQ.png)
