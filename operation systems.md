---
tags: 基礎學科, 作業系統
---

# Operation System

# CH1. OS

作業系統(Operating system, OS)是**管理電腦硬體與軟體資源的電腦程式**，同時也是電腦系統的核心與基石。
OS 主要有以下兩個功能：
- 資源分配者
- 監控使用者程式的執行，以防止不正常的運作造成對系統的危害


更進一步來說，OS必須被期待做到:
**行程管理(Processing management)**
**記憶體管理(Memory management)**
檔案系統（File system）網路通訊（Networking）
安全機制（Security）使用者介面（User interface）驅動程式（Device drivers）...


- Multiprogramming System
系統中存在多組行程同時(concurrent)執行，**避免 CPU 閒置，提升 CPU 利用度**
指系統內所存在的等待執行 process 的數目，Multiprogramming Degree 愈高，
則 CPU 使用度可能愈高(非必定的原因是**可能產生 Thrashing 現象**)  
Thrashing: 每個程式**分配到的記憶體不足，發生 page fault**，所有 process 皆忙於 swap in/out，造成 CPU idle。(CPU idle 時 OS 會企圖引進更多的 process 進入系統讓 Thrashing 現象更嚴重。)

- Time Sharing System   
是Multiprogramming System的延伸
CPU切換速度極高
反應時間(Response Time)通常是一秒內
**使得每個使用者都認為有一套專屬的系統存在**
**行程排程採用 RR 排程**
- Distributed System
透過LAN相連，透過Message Passing進行IPC


- Real Time System  
分成HARD(絕對不超時)和SOFT
定義嚴謹的固定時間限制，電腦在處理工作時必須在這個定義的時間內完成，否則工作就算失效
**不用傳統硬碟** 及 Program 皆存在ROM或RAM中
**不使用虛擬記憶體**，因為 Page Fault 的處理時間過長
HARD的話通常要特殊設計
SOFT的話，再排程上**要支持preemptive priority**

- Clustered System



# CH2. 中斷，I / O，系統調用，OS結構設計，虛擬機

## 中斷 (Interrupt)
中斷是指處理器接收到外圍硬體或軟體的信號，**導致處理器執行 context switch** 並處理該事件。

- 外部中斷(外部IO滑鼠鍵盤等)
- 內部中斷(不合法的Debug或出錯)
- 軟體中斷(像是system call)

中斷步驟-->**保存此 process 當時執行狀況(PCB)**-->之後回來再重新載入

## 中斷檢查機制

- polling: cpu不斷去檢查io(同步，輪巡)
- Interrupt-driven I/O: cpu等待訊號(非同步)
- DMA


## Dual Mode & System call
特權指令 (Privileged Instruction)，**以保護重要的資源**，需要 Kernel mode 才能執行
Kernal Mode = modebit == 0 
特權指令例如修改記憶體，操作IO等等
透過API分層，**讓使用者基本上就是用最上層的Win32 or POSIX API**
舉例: 不可能寫隻程式就能隨便修改讀取其他process的資料

- Win32 API for Windows
- POSIX API
- Java API 
![](https://i.imgur.com/yxwQp1h.png)

![](https://i.imgur.com/9s62mRT.png)



# CH3. 行程Process

## Process
- Process是OS分配**資源**(CPU使用時間、記憶體、檔案以及I/O裝置)的單位
- Thread是OS**分配CPU時間**的單位

flow
![](https://i.imgur.com/0sDhkSF.png)



- Ready和 Running之間就是CPU排程算法
- waiting：等待某事發生，例如等待使用者輸入完成。亦稱「阻塞」（blocked）

![](https://i.imgur.com/wk2qbpq.png)
process 在記憶體中的配置圖
- stack(區域變數 / 參數)
- heap(程式設計師分配malloc/free)
- data : 全域變數、靜態變數

## Context Switch
讓多個 process 可以分享單一個 CPU 資源的計算過程。當 CPU Context Switch 到另一個 process，系統會儲存舊 process 的狀態並載入新 process 的狀態。
- 恢復狀態的話 靠PCB
- 時間是 overhead
- 受限於硬體，如何加快Context Switch: 獨佔REGISTER

## Process Control Block (PCB)
作業系統核心中一種資料結構，當在切換 process 時，會把未做完的 process 相關資訊記錄在 PCB 裡。
紀錄如下
- PID
- Process State
- Programming Counter


## Scheduler
### Long-Term Scheduler [new—ready]
- 執行頻率最低，可以調控io/cpu bound的ˊ比例
- 從 job queue 中挑選合適的 jobs，並將之載入到記憶體內準備執行
- 可調控 Multiprogramming Degree
- 在VM章節會提到**Thrashing**的現象

### Short-Term Scheduler [ready—>running]
從 Ready Queue 挑選高優先度的 Process，使之獲得 CPU 控制權執行
- ch5排程演算法: FCFS SJF(短的先) RR Multi-level feedback queue(例如第一層RR 第二層比較久的RR 之類的)


### Medium-Term Scheduler [virtual memory]
記憶體空間不足，且又有其它 Process 欲進入記憶體執行，此時該 Scheduler 必須挑選某些 Process 將其 Swap Out 到磁碟中以空出記憶體空間，待記憶體有足夠空間再將其 swap in 記憶體中繼續執行。
- 可以調整 Multiprogramming Degree

### 加上VM的STD
![](https://i.imgur.com/gKafhI3.png)
- 當記憶體空間不足時，會先從等待IO的Blocked process中swap掉一些process
- 還是不足的話，接下來就是swap ready queue裡面的precess
- 等到記憶體空間夠了 就可以swap in了

### process的操作
- 樹狀結構
![](https://i.imgur.com/nUFtomY.png)
- 工作類型：與 parent 相同、與 parent 不同
- Resource sharing : 全部分享、部分分享、全部不分享
- Execution : parent child 同時執行、parent 等 child
- system call
![](https://i.imgur.com/dTGClEu.png)

### TODO
補充VORK用法



## Inter-Process Communication, IPC
兩個常見的 IPC。
- shared memory(Producer Consumer Problem )
- message Passing
    - Direct (一丟一收)
    - Indirect (郵箱)
![](https://i.imgur.com/YyajRn2.png)


# CH4. 執行緒 Thread
thread 是 OS 能夠進行運算排程的最小單位(lightweight)
優點: 輕量化的 process，可以分享code, data
(context switch 一個 thread 比 process 快約 5 倍，建立一個 thread 比 process 快約 13 倍。)

- user thread
- 在 user mode 下進行，OS 不知道有這些 thread 存在不需要 OS 介入管理
- kernal thread
- 在 kernel mode 下進行，OS 知道有這些 thread 存在，由 OS 介入管理。

user thread創建較快更輕量，缺點是一個process中只要有個user thread被system call卡死就整個卡死
所以因此演變出manytoone(kt太少) one to one(kt太多),或者many to many等的架構

> 程式設計師寫 multithread 程式是呼叫 thread API 來撰寫 (也就是 thread libraries)，而該 API 內部實作是用哪一種模型 (Many-to-one, One-to-one 或 Many-to-Many) 在規範中是沒有規定的，所以程式設計師不應該預設內部是某一種模型，以避免預期之外的錯誤。

# CH5. 排程 CPU Scheduling
用於ready-->run 的short-term scheduler
Preemptive scheduling (當前主流)
排程演算法評分標準:
1. CPU utilization(CPU使用時間)
2. Throughput(單位時間完成的個數 SJF會最高)
3. Response Time(UI, RR不錯)
4. Waiting Time

- FCFS:
先到先做 這樣可能有convey effect，第一個人做很久就GG了
- SJF:
短得先做，但是長的會starving需要提高優先權(aging)
- RR:
輪流 但是要掌握好切換時間(quantum)，短反應快但是response就很長

- Mutilevel Queue:
![](https://i.imgur.com/HRXCSiv.png)




### Thread & Process
- 變數共享與否、開銷

### Concurrency & Parallel
- Concurrency: 一個CPU利用切換的方式同時處理多個任務
- Parallel: 多CPU，各自獨立做自己的事情

# CH6. 同步問題 Synchronization
為何會有同步問題，因為有共享變數，同步問題出狀況就叫做Race Condition
我們為了避免RC，就要定義CS，CS需要符合三個條件才叫做好CS
## Race Condition
- 發生時機: 多個Thread or CPU同時取用Critical Section時要保護Critical Section
- 要避免Race Condition必須要符合Critical Section:
1. Mutual Exclusion 一次一個人
2. Progress 沒有人想要進廁所時 另一個人可以自由上(不可以AB交換，B不做時A被卡住)
3. Bounded Waiting 有限等待 不可以Starving


## 如何保護CS?

- 系統解法:  
 On single core-cpu, Disable interrupt or no-preemptive CPU scheduling
- 軟體解決:  
 1. Peterson's Solution turn+flag  
    turn：整數，存放的値為 i 或 j，誰擁有此值誰就有資格進入。  
    flag[i]：布林陣列，表示 i 想不想進去。  
 2. Bakery Algorithm(同時領號碼牌)
- 硬體解法:  
利用硬體ATOMIC不可分割特徵 只有kernel mode 能用
 1. test and set(取前值)  
檢查並設置是一種不可中斷的基本 (原子) 運算 (atomically)，它會寫值到某個記憶體位置並傳回其舊值。
 2. swap(丟球)  

## Conditional Variable
- wait()& signal() &broadbast()

## Semaphore (wait-- signal++ 0阻塞)
- 停車場例子 對一個counter++/--，0就代表無法再執行
- 分成binary semaphore or counting semaphore
- wait成功後cnt-- (要資源)
- signal後cnt++ (釋放資源)
- semaphore 又分成busy waiting(spinlock): 一直在門口等車位
- or non-busy waiting: 等待的車領號碼牌加到QUEUE裡面
- wait和signal必須要atomic(ensure的話 我們又要回到low-level 的實踐方法 軟體Peter+Bakery 硬體testset +swap)

## 常見同步問題
- Bounded Buffer
- Reader-Writers
- Dining Philosopher

## Monitors
- 更高階的API，用OO的概念避免programmer 自己implement 底下semaphore or algo發生錯誤的機會
- 在monitor結構裡的變數，只有monitor裡的function能取用跟改變
- 一次只能有一個process進入monitor，如此一來，就天然地形成了mutual exclusion，不需要程式設計師再去加鎖解鎖，可以減少人為的錯誤。

![](https://i.imgur.com/yTn36Uc.png)

## Deadlocks
Deadlock 意思是系統中存在一組 process 陷入互相等待對方所擁有的資源的情況，造成所有的 process 無法往下執行，使得 CPU 利用度大幅降低。  
同步與死鎖的關係:  
CS的同步只是死鎖資源裡面的一小部分(死鎖的資源可能是CPU RAM其他等等)  

定義: 存在一組process要進行時互相等待對方  
![](https://i.imgur.com/JhfqsDb.png)


四個必要條件:
1. Mutual Exclusion 有只能被獨佔的資源
2. Hold and Wait 我搶了資源A 還在等資源B
3. No preemption 不可以搶資源
4. Circular Waiting 循環等待

三個解決方法:
1. Prevention: 一次拿所有資源，可以Preemption搶資源(造成starvation)
2. Avoidance: Banker's Algorithm, 分成兩區(safe和unsafe)
![](https://i.imgur.com/BkZoM71.png)

3. Detect and Recovery(耗資源)


## 備註: 資料庫的ACID
1. Atomicity (不可被分割，如帳戶問題，要一次執行完 不然Rollback)
2. Consistency (一致性，在交易中會產生資料或者驗證狀態，然而當錯誤發生，所有已更改的資料或狀態將會恢復至交易之前。)
3. Isolation (可並發，資料庫允許多筆交易同時進行，交易進行時未完成的交易資料並不會被其他交易使用，直到此筆交易完成。)
4. Durability (永久寫入，交易完成後對資料的修改是永久性的，資料不會因為系統重啟或錯誤而改變。)



# CH7.Memory 記憶體管理


## Address Binding
決定程式起始位置，即程式要在記憶體的哪個地方開始執行

### Compile time
一開始編譯直接決定變數的地址在記憶體的哪裡(寫死的)
完全不彈性(recomplie)

### Load time 
由 linking loader 在load的當下改相對位置
不需要重新complie, 但是在runtime時還是不行改位置
(Swap到disk後，swap回來必須在同一個位置 容易出問題)  
還是要一次load所有模組進來(即使用不到也要)

### Execution time(Runtime)
- 需要MMU 硬體支援 
complie 時 只是轉成logical(virtual), MMU會轉成physical
- OS比較常用技術
- 缺點: 較慢(需MMU轉換)

## 系統重複使用程式碼的機制
- Static linking  
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
- Register有限，太大就不行

2. page table 保存在 memory 中，OS 利用 PTBR(Page Table Base Register) 記錄起始位址
- 慢，2 hit，先Register後Memory

3. 使用 TLB(Transaction Lookaside Buffer)  
來保存部份常用的 page table，完整的 page table 在 memory 中，TLB 是 full associate cache。
- 有點像是快取加速，TLB硬體快，但容量不能太大，只能放常用的
- TLB硬體有加速(Associative Memory) 並行查詢 o(n)-->o(1)
- TLB在context switch時 會直接flush掉


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

# CH8. Virtual Memory
虛擬記憶體 (Virtual Memory) 會讓應用程式認為其擁有連續可用的記憶體（一個連續完整的位址空間），而實際上，其通常是被分隔成多個實體記憶體碎片，還有部分暫時儲存在外部磁碟記憶體上，需要時才進行資料交換。  

## 優點
- 可以允許開很大的記憶體程式
- 提高Mutiprogramming程度 因為大家都可以進來
- i/o速度變快，因為 lazy swapper(一次只load一個page 要用在load)

## 原理 Demand Paging
- 程式執行之初不將全部的 pages 載入 memory，僅載入執行所須的 pages
- valid/invalid判斷，再判斷到底在不在memory，還是在disk
- swapper(midterm)-->處理整個process

## 舉例
- 假設舞台上擠滿各個國家的人
- 每個國家只有一個代表，其他人都先去休息
- 需要時找不到人-->page fault
- page fault就代表必須把人抓回到memory
- 但是誰是victim page?

## 算法 Page Replacement Algorithm

- 難以知道預知未來 只能近似效果
- FIFO 簡單 效果差
- OPT 將來不使用的剔掉-->誰知道?
- LRU 效果好，需要硬體Counter或stack支援
- LRU近似 SECOND CHANCE

## Thrashing
- Process分配的Frame不足-->常常發生page fault -->執行replacement
- 如果replacement是global的 代表大家可以搶來搶去
- 我搶你，你搶我，大家都在忙著swap in/out
- cpu idle, 然後long term排程器就假好心讓大家都擠進來-->爆掉
![](https://i.imgur.com/rAEQ43V.png)
- 解法
1. 降低 multiprogramming degree
2. 利用 page fault ratio 控制來防止 thrashing
3. 利用 working set model(嘗試預知未來)