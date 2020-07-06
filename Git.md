---
tags: Tool, Git
---
# Git 

## Git-Flow branching model
[GitFlow explanation](https://nvie.com/posts/a-successful-git-branching-model/?fbclid=IwAR0vW_2GVzlmFdDTxLi1FoRY5H-GZHl2kb_IYpw2femkH0_KRPn57lFO3Hs)  
[Chinese explanation](https://gitbook.tw/chapters/gitflow/why-need-git-flow.html)
![](https://i.imgur.com/MniG37y.png)
- Decentralized but centralized 
- Master for stable version(production-ready) and many branch for developing

### Devolop Branch type

- Feature branches 
from develop branch, merge back into develop
```
$ git checkout -b myfeature develop
# edit some functinos & feature code here

$ git checkout develop

& git merge --no-ff myfeature
# The --no-ff flag causes the merge to always create a new commit object

& git branch -d myfeature
$ git push origin develop
```

- Release branches
from develop branch, merge back to develop & master
```
$ git checkout -b release-1.2 develop
Switched to a new branch "release-1.2"

### do some little change here  

$ git commit -a -m "Bumped version number to 1.2"

$ git checkout master
$ git merge --no-ff release-1.2
$ git tag -a 1.2

$ git checkout develop
$ git merge --no-ff release-1.2

$ git branch -d release-1.2
```

- Hotfix branches 
from master branch, merge back to develop & master

```
$ git checkout -b hotfix-1.2.1 master
### do some hotfix here
$ git commit -a -m "Bumped version number to 1.2.1"
$ git commit -m "Fixed severe production problem"
$ git checkout master
$ git merge --no-ff hotfix-1.2.1
$ git tag -a 1.2.1
$ git checkout develop
$ git merge --no-ff hotfix-1.2.1
```

- common cmd
git branch (show all branch)  
git branch <name>  (create branch)  
git checkout <name> (switch branch)  
git checkout -b dev (create&switch )  
git branch -d dev(delete)  






## Git vs SVN
- Git分散在local，每個電腦都可以有一個repo, SVN則是集中
- Git的branch更常被用到, 不提交就不會影響到master
- Git Structure
![](https://i.imgur.com/0av1M1l.png)



## config
git config --global user.name ""  
git config --global user.email ""  
git config user.name  
## general  
git init (創建git)  

**加入STAGE**  
git add 1.py  
git add . (加入所有)  
git add *.c  
git add -A (一次加入所有變更)  

**STAGE**  
git status (看目前stage的情況)  
git status -s (簡化)  
git diff (看差距)  
git diff --cached(看STAGE裡面的差距)  
git commit -m “First Commit”  
git commit -a 忽略git add到stage情況，直接提交所有變更(新增/刪除不算)  
git commit -am "change3 in dev" 在管理庫有文件的時候 自動添加和commit  
git commit --amend --no-edit  
(為上一筆commit加一筆資料)
git reset 1.py (**UNSTAGE 1.py**)  

## 回覆上一版本reset

git reset --hard HEAD^^ (將版本回到上上版本)  
git reset --hard HEAD~2 (將版本回到上上版本)  
git reset --hard 2a17846 (直接打號碼，最推薦此方法)  

## 關於reset hard/mixed/soft

假如更改
1111  
2222  
3333  
4444  
從4回2，  
**基本上reset hard就是完全回復到過去那樣子 並把這段時間修改刪掉**  
(3333 4444會完全消失)  
預設的reset mixed並不會更改3333 4444  
只是從版控上會顯示modified  
+3333  
+4444  
(看你要再commit 還是discard change)  
soft和mixed很像，差在mixed會把暫存區的東西丟掉

[不小心使用 hard 模式 Reset 了某個 Commit-->reflog](https://gitbook.tw/chapters/using-git/restore-hard-reset-commit.html)





## 單獨切換某個版本

git checkout c6762a1 -- 1.py (切換單獨1.py到特定的版本，只對單檔版本控制)

## 檢視commit紀錄

git reflog (如果不小心切到過去，想要查看未來的commit的話)  
git log 過去commit的紀錄  
git log --oneline  
git log --oneline graph  

![](https://i.imgur.com/WvvXNE2.png)

## Branch
![](https://i.imgur.com/kI2P6dc.png)

> master 假設是穩定版，會創一個開發版的branch進行開發 然後後續在merge進來

git branch dev (創立分支)    
git branch (星號代表在哪個分支)  
git checkout dev **(切換分支)**  
git checkout -b dev (創立並切換分支)  
git branch -d dev(刪除分支)  

![](https://i.imgur.com/xs3XBWM.png)

> BRANCH只是一個指標，指向commitment (HEAD為目前在哪個分支)
砍掉branch其實也只是把cursor砍掉，實際上可以再接回去
git branch "分支名" "HASH值" (可以用reflog查詢)


## remove rm  
$ git rm welcome.html : 刪除檔案並從Git移除
$ git rm welcome.html --cached  : 只是不再被控管



## merge CONFLICT

git merge "分支名稱"  

git merge --no-ff -m "keep merge info" dev  
> --no--ff 不要使用快轉模式(fast-forward)合併  
![](https://i.imgur.com/GeMdYM2.png)

## rebase  

![](https://i.imgur.com/b16CdTN.png)


把之前修改的主版本merge進來 (rebase很危險 會複寫)  
git rebase dev  


## merge and rebase

**rebase is danger**
使用merge後會保留原始提交的記錄，在git tree中可以查看到完整的提交路線；  
而使用rebase則會使我們的git tree看起來更加清爽，更方便去維護。      
使用merge和rebase都能達到合併分支的作用  

---

接著為了順利完成 develop 分支接續的開發工作，因此需把 master 剛剛修復共用模組的變更合併到 develop 分支中，好讓 develop 分支中的錯誤可以被排除，方便繼續開發功能二。此時有兩種選擇，第一種就是使用 merge 做反向合併(develop merge master)，為何說反向是因為通常都是主幹分支來合併其他分支，而目前情況是需要讓分支來合併主幹；接著在完成功能二後，我們又需要執行正向合併(master merge develop)，讓 develop 的變更回歸到 master 主要分支中，而若以此情況繼續演進下去，不難想像最後 master 與 develop 分支會有互相交織的複雜網狀演進圖存在(如下圖所示)。

![](https://dotblogsfile.blob.core.windows.net/user/chris%20chen/a969cffc-9a36-4e1b-9ba3-e455e5cec4bb/1461946135_26128.png)

因此我們可以考慮選擇另一種方式進行，就是使用 rebase 來將 develop 分支重新接枝到指定的 master 分支節點，讓節點演進線圖比較單純好懂，但此舉會造成 develop 上舊 commit 節點被異動(因為長出分支的節點位置已經變動)，如果目前分支是有與其他人共用的話，就不適合使用 rebase 指令。執行 rebase 後節點演進圖如下所示，很單純的移動 develop 分支節點至 Bug 修復完成的 master 節點上，從演進圖來看是相當好理解。

![](https://i.imgur.com/N3fYIJX.png)


Merge 和 rebase 都是合併歷史記錄，但是結果不同。

Merge
修改內容的歷史記錄會維持原狀，但是合併後的歷史紀錄會變得更複雜。
Rebase
修改內容的歷史記錄會接在要合併的分支後面，合併後的歷史記錄會比較清楚簡單，但是，會比使用 merge 更容易發生衝突。

[分支/合併 參考圖示](https://backlog.com/git-tutorial/tw/stepup/stepup1_4.html


## revert / rebase / reset 

revert是額外開一個commit 會有點亂  
大多時候reset就能搞定版本切換  
rebase大多是整理commit  

![](https://i.imgur.com/ITGp8nm.png)


## 增加tag
可以用來當作註記release版號等等功能
