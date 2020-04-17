# Spark簡介
- Spark是一個分散式運算引擎(最上層)，基於RDD
- Resilient Distributed Datasets 彈性分散式資料集
- Spark常常拿來和Hadoop比較，本質上是不同的東西，應該是要說Spark是和hadoop的map-reduce功能相當
- RDD 可以建立在其他資料源上面 / 例如：HDFS、HBase或其他Hadoop 資料來源，Spark本身不具資料管理功能

## Spark的五大特點

1. Transformation
- RDD本身 Immutable，自己具備轉換Transformation功能，但是是等動作觸發後才執行動作，從RDD1->RDD2->RDD3這樣轉換
* 舉例: [1]-->[2,3] 他不是直接inplace改，而是記錄了-1 +2 +3這樣的步驟，這些步驟稱為Transformation
* 有點像git版控的感覺

2. Action
- RDD透過運算可以得出新的RDD，但Spark會延遲這個「轉換」動作的發生時間點。它並不會馬上執行，而是等到執行了Action之後，才會基於所有的RDD關係來執行轉換。

3. 容錯性
- 果某節點機器故障，儲存於節點上的RDD損毀，能重新執行一連串的「轉換」指令，產生新的輸出資料，如此就可以避免因為特定節點故障，造成整個系統無法運作的問題。

4. 四大特點
- Lazy evaluation: 只有觸發Action時才會運算 否則只是關聯
- Partitioned: 可以平行處理
- Cachable: 和Hadoop的不同的地方是，可以把運算後的資料放進去記憶體裡面
- Reuseable: RDD可以重複使用

5. 步驟核心(ETL)
- Extract (input)
    - 從spark context取得rdd
    - sc.parallelize(list)
    - sc.textFile(path)
- Transformation
- Load (output)
https://www.slideshare.net/popcornylu/spark-77220371

## Hadoop 分散式存儲
- 資料大到存不下，總不能一直狂買主機吧-->分散式存儲
- 基本上會有很多node, 詳細原理就跟有個網路教的漫畫很像
- https://www.itread01.com/articles/1476065707.html