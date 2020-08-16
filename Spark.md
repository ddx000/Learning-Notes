# Spark & Hadoop

## Spark
![](https://i.imgur.com/AlkFNNf.png)
- Spark是一個分散式運算**引擎**(最上層)，基於RDD
- 對外(上面)接口有scala, java, python, r等等
- 本身計算可以做SparkSQL, Streaming, MLlib等
- 往下可以透過data source api如同 - hadoop -hbase-mysql等等
- Spark常常拿來和Hadoop比較，本質上是不同的東西，應該是要說Spark是和hadoop的map-reduce功能相當

## Why Spark - Spark ML vs.Scikit learn
- Spark ML是不同機器節點(RDD)，Scikit learn是for local單機(numpy)
- RDDs表示分佈在多個不同機器節點上，可以被並行處理的數據集合，數據量大到下面HDFS分散在不同機器時，才會用的到
- spark的核心是RDD，是一個DAG版的map reduce，機器學習算法的單機和並行化版本的實現是完全不同的，sklearn作為單機的算法庫是不能簡單的移植到spark上的
- scikit learn是基於local的numpy資料結構，功能強，但數據有侷限
- 所以，我們總結下sklearn：算法容易用矩陣操作來描述，因此，sklearn 內部絕大部分操作用numpy 實現，不僅表達力很好，性能也不錯。sklearn 用joblib 來在多核架構上加速。那為什麼還需要mahout、Spark ML 呢？很簡單，那是因為sklearn 只能用單台機器來計算，因此必須受限於單機的內存，和計算能力。


## Spark3
- MLLib,Python2 is deprecated
- Faster!!!
- GPU support
- Spark-Graph X(Social networks)

## Spark接口選擇
https://spark-nctu.gitbook.io/spark/spark-jing-an/spark-yan-scala-vs.-python
- Scala API最多，且速度最快，但是不會比python來的快上手(因為我是python使用者)
- 社群和教學 上 python會更友好，學習也不用編譯什麼


## RDD
- Resilient Distributed Datasets 彈性分散式資料集
- Spark本身是計算引擎，而有計算引擎就是需要有Data
- 所有對Spark而言，dataset可以都叫做RDD
- (1)RDD本身可以建立在其他資料源上面 / 例如：HDFS、HBase或其他Hadoop 資料來源，Spark本身不具資料管理功能
- (2)RDD可以根據其他RDD Transformation而來

## RDD的五大特點
Spark 是由 Scala撰寫而成，嚴格遵守Functional Program的概念，所以RDD只能讀取無法寫入。
Spark 支援讀取 HDFS 等分散式儲存裝置的檔案，故可以使用HDFS的特性便於進行分散式的運算。

RDD透過計算而變換(added)，A-->A+-->A++，而這個轉換是Lazy evaluation，需要action一次全部轉換
本身具有容錯性，允許部分RDD損毀，並且可以支援**平行處理+記憶體內運算**

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
[用scala講解rdd如何被創造](https://spark-nctu.gitbook.io/spark/rdd-cao-zuo/resilient-distributed-dataset-rdd)

## RDD vs. DataFrame vs. DataSet
https://spark-nctu.gitbook.io/spark/rdd-cao-zuo/rdd-vs.-dataframe-vs.-dataset
- 接下來就是Spark3就是走入DataFrame(schemeRDD)
- 考慮到程式語言的特性，顯然的，Python 無法支援 DataSet API
- 在 Spark 3.0 中，RDD-based 的 API 要停止更新，而主要支援 DataFrame-based 的 API。但是，這並不代表 RDD 這種資料型態將從 Spark 中消失。事實上，不論是 DataFrame 還是 DataSet 都是基於 RDD 的架構完成，RDD 仍然是最基礎的 Spark 資料型態，也是 Spark 可以提供平行化的設計


## Spark Streaming
https://spark-nctu.gitbook.io/spark/
> 在 Spark 中，所有的資料單元被表示為 RDD，因此在進行流運算時，處理的是DStream，也就是一系列的 RDD 輸入，而不是一筆筆連續的資料輸入。考慮到源源不斷的資料輸入，Spark Streaming 會先把進入的資料按照時間切分成不同的區塊，而每一格區塊則直接對應產生一個 RDD 資料集合，並放入Spark中進行RDD 的運算，當然，輸出的格式也是一連串 RDD 的形式。使用者能夠根據需求，將資料轉傳入 HDFS、檔案或是資料庫中。
![](https://i.imgur.com/QMlxrpj.png)

## MLlib
除了 Spark Streaming 之外，在 Spark 上也成立了一個 MLlib 專案，負責提供機器學習 (Machine Learning) 演算法的函式庫。
這些工具都是建立於 Spark 所定義的 RDD 運算上，使得 Spark 的管理節點能夠基於資料所在的位置，將運算分散到叢集中的工作結點上


## Hadoop 分散式存儲
- 資料大到存不下，總不能一直狂買主機吧-->分散式存儲
- 基本上會有很多node, 詳細原理就跟有個網路教的漫畫很像
- https://www.itread01.com/articles/1476065707.html

## RDD SPARK API
- key value鍵值對可以放進RDD就是
- SPARK API
- rdd.reduceByKey(lambda x,y: x+y)
- rdd.groupByKey (): Group values with the same key
- rdd.sortByKey():
- rdd.keys()
With key/value data, use mapValues() and flatMapValues() if your transformation doesn't affect keys.


## 在RDD裡面常常會用到的functional programming
https://www.youtube.com/watch?v=e-5obm1G_FY
![](https://i.imgur.com/ZeqIUBS.png)
白話: 盡量在no mutation的情況下，用map等transformation的function去產生新的RDD




## data1
```
(ID, name, age, numbers of friends)  
0,Will,33,385  
1,Jean,33,2  
2.Hugh,55,221  


def parseLine(line):
   fields = line.split(',')
   age = int(fields[2])
   number_of_friend = int(field[3])
   return (age, number_of_friend)

lines = sc.textFile('friends.csv')
rdd = lines.map(parseLine)

Output:
33, 385
33, 2
55, 221
40, 465 
```

```
rdd.mapValues(lambda x:(x,1)).reduceByKey(lambda x,y: (x[0]+y[0],x[1]+y[1]))

1. mapValues 保持key不變
(33, 385) -->(33, (385,1) )
(33,2 ) -->(33, (22,1) )

(55,221 ) -->(55, (221,1) )


2. reduceByKey(lambda x,y: (x[0]+y[0],x[1]+y[1]))
(33,(387,2))

3. averageByAge = totalsByAge.mapValues(lambda x:x[0]/x[1])
(33,(387,2))--> (33,193.5)
```



