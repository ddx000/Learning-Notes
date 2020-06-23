# Redis

- 建議在linux上安裝，windows也有fork 版本
- A dictionary is a data structure, Redis is an infrastructure which uses key-value stores at its heart. 
- https://stackoverflow.com/questions/47988087/what-is-the-difference-between-python-native-data-structure-dictionary-and-re
- 有點像是python 的dictionary，但是可以handle的量級完全就是不能比的，一種數據存儲引擎
- 數據(redis)和程式變數(dic)本來就要分開來

NoSQL又分成四大類：
1.Key-Value，如Redis。
2.Document-Oriented，如MongoDB。
3.Wide Column Store，如Cassandra。
4.Graph-Oriented，如Neo4J。


https://kknews.cc/code/jj88ev6.html
https://ek21.com/news/1/18146/

## 為什麼使用
- 解決應用伺服器的cpu和內存壓力
- 配合SQL做高速緩存(Redis在SQL前面)
- 緩存高頻次訪問的數據，降低資料庫io

## 使用場景
- 常常被查 但是不常修改的數據 且不是非常重要的數據
- Select 資料庫前查詢redis，有的話使用redis數據，放棄select 資料庫，沒有的話，select 資料庫，然後將數據插入redis
- 例如: 最新列表
- 例如新聞列表頁面最新的新聞列表，如果總數量很大的情況下，儘量不要使用select a from A limit 10，嘗試redis的 LPUSH命令構建List，一個個順序都塞進去就可以啦。不過萬一內存清掉了，查詢不到存儲key的話，用mysql查詢並且初始化一個List到redis中就好
- 排行榜/熱度排行榜，如果使用傳統的關係型數據庫來做這個事兒，非常的麻煩，而利用Redis的SortSet數據結構能夠非常方便搞定；
- 計算器/限速器，利用Redis中原子性的自增操作，我們可以統計類似用戶點讚數、用戶訪問數等，這類操作如果用MySQL，頻繁的讀寫會帶來相當大的壓力；限速器比較典型的使用場景是限制某個用戶訪問某個API的頻率，常用的有搶購時，防止用戶瘋狂點擊帶來不必要的壓力；
![](https://i.imgur.com/SEEilMs.png)

## 不建議
- Redis的持久化不是很可靠，需要長期儲存的資料，且有可靠性需求的，不要塞進redis


