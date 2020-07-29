# 演算法與資料結構

備註：資結演算法大多是實作才會比較了解的東西，
所以此篇筆記不常維護，希望後續是能透過實作演算法的題目來作筆記



## 演算法 - 排序
https://medium.com/@jimmy_huang/python-%E5%90%84%E7%A8%AE%E5%B8%B8%E8%A6%8Bsort-21fedc01ccb

### 排序演算法
https://medium.com/@ChYuan/%E6%8E%92%E5%BA%8F%E6%BC%94%E7%AE%97%E6%B3%95-sorting-cc521986ad5f


### 氣泡排序法
http://notepad.yehyeh.net/Content/Algorithm/Sort/Bubble/1.php
從第一個開始，和第二個倆倆比較並排序，再來是第二個和第三個比較，以此類推。每一次從頭比到尾排序會讓最大的值到最後面。
第一次走N 第二次走N-1 一直到1 所以複雜度約為N^2
最差: N^2(逆序)
平均: N^2
最好: N^1(正序)
- 穩定排序

### 選擇排序法
http://notepad.yehyeh.net/Content/Algorithm/Sort/Selection/1.php
將資料分成已排序、未排序兩部份
依序由未排序中找最小值(or 最大值)，加入到已排序部份的末端
很簡單的思維，就是每次都找到當前最小的值，然後和當前要排序的位置交換值。
Best Case：Ο(n2)
Worst Case：Ο(n2)
Average Case：Ο(n2)


