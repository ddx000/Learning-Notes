# Operation System

### Thread & Process
- 變數共享與否、開銷

### Concurrency & Parallel
- Concurrency: 一個CPU利用切換的方式同時處理多個任務
- Parallel: 多CPU，各自獨立做自己的事情

# CH6. Synchronization

## Race Condition
- 發生時機: 多個Thread or CPU同時取用Critical Section時要保護Critical Section
- 想要解決:
-  Mutual Exclusion / Progress / Bounded Waiting
-  一次一個人 / 沒有人想要進廁所時，另一個人可以自由上 / 不可以Starving

-  系統解法:
-  On single core-cpu, Disable interrupt or no-preemptive CPU scheduling
-  軟體解決:
-  1. Peterson's Solution turn+flag
-  2. Bakery Algorithm(同時領號碼牌)
-  硬體解法:
-  利用硬體ATOMIC不可分割特徵 
-  1. test and set(取前值)
-  2. swap(丟球)

## Conditional Variable
- wait()& signal() &broadbast()

## Semaphore
- 停車場例子 對一個counter++/--，0就代表無法再執行
- 分成binary semaphore or counting semaphore
- wait成功後cnt-- 
- signal後cnt++
- semaphore 又分成busy waiting(spinlock): 一直在門口等車位
- or non-busy waiting: 等待的車領號碼牌加到QUEUE裡面
- wait和signal必須要atomic(ensure的話 我們又要回到low-level 的實踐方法 軟體Peter+Bakery 硬體testset +swap)

## Deadlocks


## 常見同步問題
- Bounded Buffer
- Reader-Writers
- Dining Philosopher

## Monitors
- 更高階的API，用OO的概念避免programmer 自己implement 底下semaphore or algo發生錯誤的機會
- 在monitor結構裡的變數，只有monitor裡的function能取用跟改變
- 一次只能有一個process進入monitor，如此一來，就天然地形成了mutual exclusion，不需要程式設計師再去加鎖解鎖，可以減少人為的錯誤。

![](https://i.imgur.com/yTn36Uc.png)



## ATOMIC TRANSACTION

## 備註: 資料庫的ACID
1. Atomicity (不可被分割，如帳戶問題，要一次執行完 不然Rollback)
2. Consistency (一致性，在交易中會產生資料或者驗證狀態，然而當錯誤發生，所有已更改的資料或狀態將會恢復至交易之前。)
3. Isolation (可並發，資料庫允許多筆交易同時進行，交易進行時未完成的交易資料並不會被其他交易使用，直到此筆交易完成。)
4. Durability (永久寫入，交易完成後對資料的修改是永久性的，資料不會因為系統重啟或錯誤而改變。)



# CH7.Memory

- Disk 和 Memory : Swapping
- Virtual Memory(虛擬記憶體)

## Address Binding
- Compile time
一開始編譯直接決定變數的地址在記憶體的哪裡(寫死的)
完全不彈性(recomplie)

- Load time 
-- 由 linking loader 在load的當下改相對位置
不需要重新complie, 但是在runtime時還是不行改位置
(Swap到disk後，swap回來必須在同一個位置 容易出問題)  
-- 還是要一次load所有模組進來(即使用不到也要)

- Execution time(Runtime)
-- 需要MMU 硬體支援 
-- complie 時 只是轉成logical(virtual), MMU會轉成physical
-- OS比較常用技術
-- 缺點: 較慢(需MMU轉換)


** 先做linking再做loading，所以loading前必先有linking

- static linking
Compile 時 library 就加入程式碼
**會重複linking lib**

- Dynamic linking
程式執行期間，當某個模組被真正呼叫到時才將其載入到 main memory 中
Windows .dll (Dynamic Linking Libraries)
Linux .so (Shared Object)。
**目標是防止重複load lib**

- Dynamic loading
讓 programmer 在程式執行的過程中，動態決定要載入的 libraries
**函式沒有用到就不要load，有需要這個FUNC我才LOAD到記憶體，其實很多error handling的是用不到的

## Swapping
把process 透過midterm scheduling swap到disk去 free memory
不可以在I/O 時Swapping(不然會出問題，記憶體位置會跑掉)
可以用OS Buffer解決這個問題(共用記憶體)

## Contiguoes Memory Allocation
找取連續空間可用-->Available list
記憶體可以Variable-Size分配，但是就要考慮有洞(碎片化)
First-Fit(就快 常用)... Best-fit其實有點greedy但是未必更好

### External Fragmentation 外部碎裂
- 發生在Variable size partition  
所有可用空間總和大於某個 process 所需要，但因為這些空間不連續造成零碎的洞

解決方法
- Compaction：類似磁碟重組
- Page memory management

### Internal Fragmentation 內部碎裂
- 發生在Fixed size partition  
作業系統配置給 process 的 memory 空間大於 process 真正所需的空間
這些多出來的空間該 process 用不到，而且也沒辦法供其他 process 使用，形成浪費  
  
解決方法
- 減少Paging最小單位(但是會造成paging table爆大)

## Non-Contiguoes Memory Allocation (Page)

切Logical Memory--> **Pages**
切Physical Memory--> **Frames**
會切成一樣大，對應表叫做page table
中間是MMU進行mapping
- 沒有external fragmentation(因為不連續)
- 可以share memory(可以放pointer)
![](https://i.imgur.com/B1XuDB6.png)
- 每一個process有自己的page table，且可以記憶體protection
- 一個page size-->4kb/8kb  (太大 內部碎裂多，太小 page table大)

### Page Table 的製作 
MMU需要把page table 的memory load到physical register

1. 使用 register 保存page table每個項目的內容
-- 太大就不行

2. page table 保存在 memory 中，OS 利用 PTBR(Page Table Base Register) 記錄起始位址
-- 2hit，MMU先存取page table，再地址轉換

3. 使用 TLB(Transaction Lookaside Buffer) 來保存部份常用的 page table，完整的 page table 在 memory 中，TLB 是 full associate cache。
-- 有點像是快取加速，TLB硬體快，但容量不能太大，只能放常用的
-- TLB硬體有加速(Associative Memory) 並行查詢 o(n)-->o(1)
-- TLB在context switch時 會直接flush掉

- 會多加一個invalid -valid bit

![](https://i.imgur.com/1G077AL.png)

### Page Table如果太大，會不好找到連續記憶體放
- 找到可以拆成小page table的方法
1. Hierarchical Paging
2. Hashtable+Linked list
3. Inverted Page Table(其實是frame table)
- 存PID+Page Number 這樣其實可以不用存page table，就存physical的就一樣大
- 但是無法支援shared page/memory

## Segmentation
-- User View ，各Segment大小可以都不一樣  
段與段可以任意非連續，但是每一段必須是連續的任意長度，所以有external fragmentation，無internal  
-- logical address 為兩個值 位置+長度(s+d)  
-- 比較容易memory sharing / protection 
-- 可以同時和paging使用(Segmentation靠近使用者，先segment(中間層)後page)

# Virtual Memory
-- Demand Paging(一次只load一個page 要用在load)
-- valid/invalid判斷，再判斷到底在不在memory，還是在disk
-- pure demand paging(我只放page table 到memory 其他在disk)
-- swapper(midterm)-->處理整個process
-- pager-->處理單個單個的page
 
- Page-fault-trap
- 