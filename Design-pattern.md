# Design pattern 及物件導向


## reference

https://refactoring.guru/design-patterns/python
https://python-web-guide.readthedocs.io/zh/latest/design/design.html
https://adamchang.squarespace.com/python/design-pattern
https://www.youtube.com/watch?v=CG4FUo3Oh7E

---

## 定義: 
> 每一個模式(Pattern)都是在 某些環境下(Context)下 針對某問題(Problem) 提出的解決方案 (Solution)
> Design pattern是針對程式設計的經驗法則，以物件導向為概念(再利用的解決方案)

## UML:
http://ashkandi.herokuapp.com/blog/2015/09/14/uml-notes-01/
第一行Class 第二行Attribute 第三行Methods
斜線抽象介面 / +public / -private
實線三角形: 繼承(指到的為 parent class)
虛線三角形: 實例化(指到的為 instance)
菱形實線箭頭: 聚合



## 創建型模式
### Abstract Factory

![](https://i.imgur.com/I3AKV27.png)
客戶端代碼可以通過相應的抽象接口調用工廠和產品類。 無需修改實際客戶端代碼， 就能更改傳遞給客戶端的工廠類， 也能更改客戶端代碼接收的產品變體。
假設客戶端想要工廠創建一把椅子。 客戶端無需了解工廠類， 也不用管工廠類創建出的椅子類型。 無論是現代風格， 還是維多利亞風格的椅子， 對於客戶端來說沒有分別， 它只需調用抽象接口就可以了。
最後一點說明： 如果客戶端僅接觸抽象接口， 那麼誰來創建實際的工廠對象呢？
一般情況下， 應用程序會在初始化階段創建具體工廠對象。 而在此之前，應用程序必鬚根據配置文件或環境設定選擇工廠類別。
其運作方式如下： 應用程序啟動後檢測當前操作系統。 根據該信息， 應用程序通過與該操作系統對應的類創建工廠對象。 其餘代碼使用該工廠對象創建UI元素。 這樣可以避免生成錯誤類型的元素。
使用這種方法， 客戶端代碼只需調用抽象接口， 而無需了解具體工廠類和UI元素。 此外， 客戶端代碼還支持未來添加新的工廠或UI元素

如果代碼需要與多個不同系列的相關產品交互， 但是由於無法提前獲取相關信息， 或者出於對未來擴展性的考慮， 你不希望代碼基於產品的具體類進行構建， 在這種情況下， 你可以使用抽象工廠。
在許多設計工作的初期都會使用工廠方法模式（較為簡單，而且可以更方便地通過子類進行定制），隨後演化為使用抽象工廠模式、原型模式或生成器模式（更靈活但更加複雜）。

abstract class 抽象類別:
https://openhome.cc/Gossip/Python/AbstractClass.html

抽象類別（Abstract Class）不可以被implement, 而底下必須繼承他，然後實作出有加上decorator的函式

https://refactoringguru.cn/design-patterns/abstract-factory

```
import random
from abc import ABCMeta, abstractmethod

class GuessGame(metaclass=ABCMeta):
    @abstractmethod
    def message(self, msg):
        pass

    @abstractmethod
    def guess(self):
        pass     

    def go(self):
        self.message(self.welcome)
        number = int(random.random() * 10)
        while True:
            guess = self.guess();
            if guess > number:
                self.message(self.bigger)
            elif guess < number:
                self.message(self.smaller)
            else:
                break
        self.message(self.correct)
        
class ConsoleGame(GuessGame):
    def __init__(self):
        self.welcome = "歡迎"
        self.prompt = "輸入數字："
        self.correct = "猜中了"
        self.bigger = "你猜的比較大"
        self.smaller = "你猜的比較小"
    
    def message(self, msg):
        print(msg)
    
    def guess(self):
        return int(input(self.prompt))

game = ConsoleGame()
game.go()
```

### Builder
當對象需要多個部分組合起來一步步創建，並且創建和表示分離的時候。
可以這麼理解，你要買電腦，工廠模式直接返回一個你需要型號的電腦，但是構造模式允許你自定義電腦各種配置類型，
組裝完成後給你。這個過程你可以傳入builder從而自定義創建的方式。
![](https://i.imgur.com/K2210OT.png)
![](https://i.imgur.com/lSMtwx4.png)

```

class Computer:
    def __init__(self, serial_number):
        self.serial = serial_number
        self.memory = None      # in gigabytes
        self.hdd = None         # in gigabytes
        self.gpu = None

    def __str__(self):
        info = ('Memory: {}GB'.format(self.memory),
                'Hard Disk: {}GB'.format(self.hdd),
                'Graphics Card: {}'.format(self.gpu))
        return '\n'.join(info)


class ComputerBuilder:
    def __init__(self):
        self.computer = Computer('AG23385193')

    def configure_memory(self, amount):
        self.computer.memory = amount

    def configure_hdd(self, amount):
        self.computer.hdd = amount

    def configure_gpu(self, gpu_model):
        self.computer.gpu = gpu_model


class HardwareEngineer:
    def __init__(self):
        self.builder = None

    def construct_computer(self, memory, hdd, gpu):
        self.builder = ComputerBuilder()
        [step for step in (self.builder.configure_memory(memory),
                        self.builder.configure_hdd(hdd),
                        self.builder.configure_gpu(gpu))]

    @property
    def computer(self):
        return self.builder.computer
```

使用buidler，可以创建多个builder类实现不同的组装方式
engineer = HardwareEngineer()
engineer.construct_computer(hdd=500, memory=8, gpu='GeForce GTX 650 Ti')
computer = engineer.computer
print(computer)



### Factory Method
https://python-web-guide.readthedocs.io/zh/latest/design/design.html
https://refactoringguru.cn/design-patterns/factory-method
讓所有產品都遵循同一接口。 該接口必須聲明對所有產品都有意義的方法。
在創建類中添加一個空的工廠方法。 該方法的返回類型必須遵循通用的產品接口。
在創建者代碼中找到對於產品構造函數的所有引用。 將它們依次替換為對於工廠方法的調用， 同時將創建產品的代碼移入工廠方法。 你可能需要在工廠方法中添加臨時參數來控制返回的產品類型。
工廠方法的代碼看上去可能非常糟糕。 其中可能會有復雜的 switch分支運算符， 用於選擇各種需要實例化的產品類。

工廠方法適合對象種類較少的情況，如果有多種不同類型對象需要創建，使用抽象工廠模式。以實現一個遊戲的例子說明，在一個抽象工廠類裡實現多個關聯對象的創建：

## Singleton Pattern

單例模式是一種創建型設計模式， 讓你能夠保證一個類只有一個實例， 並提供一個訪問該實例的全局節點。
單例模式同時解決了兩個問題， 所以違反了_單一職責原則_ ：
保證一個類只有一個實例。 為什麼會有人想要控制一個類所擁有的實例數量？ 最常見的原因是控制某些共享資源（例如數據庫或文件）的訪問權限。
它的運作方式是這樣的： 如果你創建了一個對象， 同時過一會兒後你決定再創建一個新對象， 此時你會獲得之前已創建的對象， 而不是一個新對象。
注意， 普通構造函數無法實現上述行為， 因為構造函數的設計決定了它必須總是返回一個新對象。

為該實例提供一個全局訪問節點。 還記得你（好吧，其實是我自己）用過的那些存儲重要對象的全局變量嗎？它們在使用上十分方便，但同時也非常不安全，因為任何代碼都有可能覆蓋掉那些變量的內容，從而引發程序崩潰。
和全局變量一樣， 單例模式也允許在程序的任何地方訪問特定對象。 但是它可以保護該實例不被其他代碼覆蓋。
還有一點： 你不會希望解決同一個問題的代碼分散在程序各處的。 因此更好的方式是將其放在同一個類中， 特別是當其他代碼已經依賴這個類時更應該如此。

## 結構性模式
### Adapter

![](https://i.imgur.com/rUT9ATo.png)
適配器模式通常在已有程序中使用，讓相互不兼容的類能很好地合作。


## 行為模式
### Command
![](https://i.imgur.com/wKrTAs2.png)
像是QT的信號與插槽
![](https://i.imgur.com/s30OuVc.png)

### Iterator
- 封裝成.next，而底下的演算法可以是DFS, BFS等等

迭代器模式是一種行為設計模式， 讓你能在不暴露集合底層表現形式（列表、棧和樹等）的情況下遍歷集合中所有的元素。

當集合背後為複雜的數據結構， 且你希望對客戶端隱藏其複雜性時（出於使用便利性或安全性的考慮），可以使用迭代器模式。

迭代器封裝了與復雜數據結構進行交互的細節， 為客戶端提供多個訪問集合元素的簡單方法。 這種方式不僅對客戶端來說非常方便， 而且能避免客戶端在直接與集合交互時執行錯誤或有害的操作， 從而起到保護集合的作用。

使用該模式可以減少程序中重複的遍歷代碼。

重要迭代算法的代碼往往體積非常龐大。 當這些代碼被放置在程序業務邏輯中時， 它會讓原始代碼的職責模糊不清， 降低其可維護性。 因此， 將遍歷代碼移到特定的迭代器中可使程序代碼更加精煉和簡潔。

如果你希望代碼能夠遍歷不同的甚至是無法預知的數據結構， 可以使用迭代器模式。

該模式為集合和迭代器提供了一些通用接口。 如果你在代碼中使用了這些接口， 那麼將其他實現了這些接口的集合和迭代器傳遞給它時， 它仍將可以正常運行。

![](https://i.imgur.com/QbVC3cl.png)

### Observer
- 訂閱與發送

![](https://i.imgur.com/GZL5IDS.png)

### Strategy
