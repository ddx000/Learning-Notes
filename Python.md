# Python 

## Python List
List實現是存放pointers 的dynamic arraylist(數組)
dynamic的意思是append後會自動擴容(當然不是每次append都擴容，會有一個buffer)
在擴容時會有比較大的cost(因為要移動，所以最好強迫他一開始就分配夠的數目)
insert和delete都是O(N)


## Python List vs Numpy Array

1. 原生list比numpy array多存pointers 記憶體開銷較大
![](https://i.imgur.com/HBOqbZI.png)
![](https://i.imgur.com/eOexqwD.png)
2. 原生list是存pointers, 所以可以存不同類型, numpy是"連續"數組存放在記憶體中，相同類型
3. Element wise operation ，科學運算快

## 鏈表與數組

數組一開始需要分配好空間，擴容常常需要移動
所以如果不確定大小，鏈表或許是好選擇(不用移動元素)
![](https://i.imgur.com/Nn6Cs39.png)

但是在Random access上，數組有絕對優勢(找單一個元素 有index O(1))
![](https://i.imgur.com/eFHrAni.png)

如果說鏈表常做刪除特定元素，可以搭配哈希表存那個元素的位置，這樣會快很多


## Python Deque

python deque double ended queue是雙向鏈表
頭尾append and pop均是O(1)
list(數組)的缺陷是插入慢，
