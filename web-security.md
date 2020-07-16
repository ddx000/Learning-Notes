# 網路安全

## 基礎密碼學

### 對稱式加密(Symmetric Encryption) - DES/AES
對稱式加密就是傳送方與接收方的加解密皆使用同一把密鑰

缺點: 但對稱式加密最大的弱點是，**要是這把鑰匙在第一次傳送時，被中間人攔截且複製**，
再原封不動傳給接收方，那之後所有攔截到的加密訊息，都能被輕易破解，因此就需要更進階的非對稱式加密技術去解決這個bug。

### 非對稱式加密(Asymmetric Encryption) - RSA (速度較慢)

#### 公開金鑰(Public key)及私密金鑰(Private key)
每個使用者都擁有一對金鑰(pair)：
公開金鑰能被廣泛的發佈與流傳，而私密金鑰則必須被妥善的保存
**公鑰加密私鑰解密，也可以私鑰加密公鑰解密**

> A 跟 B 都先生成一組公私鑰的 pair，A 把 A 的公鑰傳給 B，B 把 B 的公鑰傳給 A。
> 現在 A 有他自己的私鑰跟 B 的公鑰，B 有他自己的私鑰跟 A 的公鑰。
> A 要傳東西給 B 就用 B 的公鑰加密，然後 B 拿到之後用 B 自己的私鑰解密。
> 一切搞定，即使在中途被攔截，只要 B 的私鑰沒有流出就完全不會有事。反之亦然。

中心思想就是利用公鑰可以用來加密的特性，而如果你是用公鑰加密你必須要用私鑰解密。
所以我根本不怕公鑰流出(公開給所有人也沒差)，只要我私鑰保存好就好，我私鑰根本就沒傳過，也不可能被攔截


#### 數位簽章
那既然大家都有 B 的 public key ，那任何人都可以用 B 的公鑰加密傳訊息給 B，那 B 怎麼知道哪個是 A 寫的哪個是別人偽造的呢？這裡就要引進數位簽章的概念。

數位簽章就是 A 在傳送訊息前，用 A 的私鑰加密，傳給 B。B 再用 A 的公鑰來看是不是真的是 A 簽名的(事實上是對內容的 Hash 簽名，不過為了講解方便，就先當直接對內容簽)。

這裡運用的是可以公鑰加密私鑰解密，也可以私鑰加密公鑰解密的第二個特性。

你說公鑰所有人都有怎麼辦？反正全世界都知道這訊息是A的訊息也沒關係，只要沒有 B 的私鑰就沒有其他人可以看到內容！
(msg+Bpub加密+Apri加密)--->Apub解鎖 + Bpri解鎖 +得到msg

一開始 A 先把 A 的公鑰傳給 B，B 先把 B 的公鑰傳給 A。

A 要傳給 B 之前，把要傳的內容，先用 B 的公鑰加密再用 A 的私鑰簽，然後 B 用 A 的公鑰確認簽章再用 B 的私鑰解密內容。

![](https://i.imgur.com/8pivA0o.png)

#### 非對稱加密- 中間人攻擊
![](https://i.imgur.com/LORBxkr.png)

解法就是只要在互傳公鑰的時候，有一個可信的第三方可以證明說一開始你拿到的公鑰是真的就可以


### 雜湊函數

MD5，SHA-1

## HTTPS(SSL)

client把所有支援的加密方式告訴Server 再給一個隨機數R1

Server從所有的加密方式中選一種 然後把Server的身份訊息以證書的形式告訴Client
(證書裡面包含了是哪個機構給Server發證書的 還有Server的公鑰) 
自己保留私鑰 然後給client另一個隨機數R2
client拿到證書之後 就去問那個第三方機構這個證書是否可靠

如果可以信任的話 client會再生成一個隨機數R3 用剛剛Server給的公鑰加密變成C 把C丟給Server

Server用私鑰對C解密得到R3
雙方都有都有R1 R2 R3了 生成秘鑰 K = R1+R2+R3 可喜可賀 之後就可以兩邊用對稱式加密來說悄悄話了

至於為什麼不直接非對稱式用到底就好 因為對稱式的效率比非對稱式的高很多 所以只要在一開始確認身份跟確認秘鑰的時候用非對稱式就可以了

HTTPS:
Identity 保證你跟她對話
Confidentiality 保證沒有人看到這對話
Integrity 保證沒有人修改過這對話

## SSH

> ssh username@hostname
SSH後 CMD會跳一個那個主機的公鑰的hash
這就是那個主機的公鑰，要去確認是不是，不然容易被中間人攻擊
輸入密碼後，系統會將密碼和**隨機字串**使用該主機的公鑰加密
該主機就可以透過私鑰解密拿到其**隨機字串** 其隨機字串就是對稱式加密的KEY


### SSH設定公鑰私鑰(加速登入 不用密碼 更安全)
https://noob.tw/ssh-key/
我們在local端可以存hostname的公鑰，這樣就可以很快知道host是不是host

要怎麼讓這個host知道你真的是你呢，上一個方法是用**密碼+隨機字串**驗證

這邊可以反向，一樣將私鑰存在自己電腦，把公鑰給對方，local私鑰加密**主機給的隨機字串**，
他用公鑰解密是不是主機自己當初給出來的隨機字串
是的話就允許通行 所以之後不再需要輸入密碼


## Web Application
來源: OWASP
https://owasp.org/www-project-top-ten/
https://www.jyt0532.com/2017/03/19/owasp/

1. SQL Injection
不要亂用字串拼接，改用ORM或者Parameterized query(SQL和傳進去的參數分開驗證)
就是code有code的syntax跟允許的character data有data的syntax
2. Broken Authentication and Session Management
就是被盜帳號，例如密碼沒hash或申請帳號時未加密(HTTPS)，Sessionid被其他人知道或完全不會timeout
3. Cross-Site Scripting(XSS)
例如放一段html腳本在留言區，預防就是要過濾user input!!!
應用於Django: django自己的Templates 就有內帶 HTML escaping
4. Insecure Direct Object Reference
如果server直接用user給的變數去access檔案 就可以去猜server存其他檔案的檔名or資料夾 
舉個例子 url http://example.com/showimage?image=img1 這是網站要給你看的image1的link
那你想看img2就直接把後面改成img2 如果可以看得到那就是A4
5. Security Misconfiguration
像是你安裝一個database 會有default的帳密比如說user: admin 密碼: admin 你沒有把它拿掉
或是server error的時候你把所有error message全部傳給client
6. Sensitive Data Exposure
傳輸未加密或密碼沒有用hash或加salt 老問題
7. Missing Function Level Access Control
網頁沒有做好access control 比如說一些webserver的default path:
/log
/phpmyadmin
/admin等等 這些權限沒關就屬於這類
8. CSRF
cookie/session還有效的時候(比如你現在開facebook不用輸入密碼 因為你cookie還有效) 讓你在你不知情的情況下送http request 
- CSRF 解法
- 解法: TOKEN(最常見，亂數TOKEN)，POST(會被破解)，驗證碼(煩人)，檢查refer url(會被破解)


## Security in Django
https://ithelp.ithome.com.tw/articles/10230384
https://docs.djangoproject.com/en/3.0/topics/security/

1. XSS Cross-site scripting
Templates  automatic HTML escaping
2. CSRF 
Django 會透過 CsrfViewMiddleware，
來檢查每個 POST request 裡面的 csrfmiddlewaretoken 
(這個 token 不是只是一個值這麼簡單)，還有檢查 referrer 欄位是不是同一個 domain。
3. Injection
ORM
4. Clickjacking
按鈕是飛機的機長，而攻擊者利用 Clickjacking 的技術，強迫機長改飛另一個方向( 強迫按鈕改作另一件事情 )的感覺
簡單來說，ClickJacking 就是 你以為的按鈕 不是 你以為的按鈕，你以為你點了一個下一頁的按鈕，可是它可能是一個打開筆電前鏡頭的按鈕，也可能是一個載入木馬程式的按鈕，利用的是把 frame 設為透明的，變成在頁面顯示你以為的網頁，但背後卻隱藏真實的網頁。現在的瀏覽器會有 x-frame-options 的防禦機制，來保護使用者意外執行惡意的 frame
Django 透過 X-Frame-Options middleware，把 X-Frame-Options 的 header 改成 DENY，也就是不給嵌入 frame，直接杜絕 Clickjacking 的機會。



## 如何儲存密碼
https://www.jyt0532.com/2017/02/19/password-security/
- Hash your password(無法逆向)
但是還是可能被破解
- 加Salt，增加難度，加random的salt會更讓人難以破解
這個時候就需要加點隨機的東西增加亂度了 解法就是在每個人創帳號密碼的時候
隨機生成一個字串 加在明碼後面或是穿插明碼其中 Verify的時候也一樣 只要server記得每個人的隨機字串就可以
HASH(plain_text + salt) == saved_string
最好連salt的長度也是random(當然要越長越好) 這樣的話hacker就必須所有的長度都有個表
那其實更快的方式是直接把username或是email加進來hash 基本上就是增加原本字串的難度 畢竟所有md5的output是2^128
沒有一個machine可以記得了所有的input 而增加難度最簡單的方式 就是增加長度 長度每加一 難度變36倍
- 既然salt是存在database的明碼 那遲早有一天記憶體大到可以cover長度10的密碼 那再把它減去salt不就得到user的密碼了嗎

A:所以salt有兩個目的 第一個是讓hash前的字串總長度拉長 這樣即使使用者給的長度不夠 還是可以增加暴力破解難度
舉個例子 如果user密碼長度是10 salt長度是5 那總長度是15的密碼就不太可能用建表暴力解再減salt的方式
Q:總長度15的不好建 10的總可以了吧 他不就可以把所有排列組合+salt建表嗎
A:答對了！這就是為什麼salt要random
salt的第二個目的是可以把損失降到最低 那個hacker一次還是只能破解一個人的密碼 因為他必須對每個新的salt 重新建表 這樣可以幫security部門爭取時間叫人家改密碼




---
## REF



https://www.jyt0532.com/2017/03/09/ssh/

https://medium.com/@RiverChan/%E5%9F%BA%E7%A4%8E%E5%AF%86%E7%A2%BC%E5%AD%B8-%E5%B0%8D%E7%A8%B1%E5%BC%8F%E8%88%87%E9%9D%9E%E5%B0%8D%E7%A8%B1%E5%BC%8F%E5%8A%A0%E5%AF%86%E6%8A%80%E8%A1%93-de25fd5fa537

https://blog.techbridge.cc/2017/04/16/simple-cryptography/