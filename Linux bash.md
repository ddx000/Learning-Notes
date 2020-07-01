# Linux bash command

## Linux command
apt 是用來取代 apt-get的  
sudo apt-get update  
sudo apt-get upgrade  
sudo apt-get install python3-pip  
sudo apt-get install virtualenv  
virtualenv --python= (target)  
pip install -r requirements.txt  
source (target directory)/bin/activate  

## 關於路徑
- windows"\" linux"/"
- 而在輸入程式path時，\通常代表轉義詞，所以通常用雙斜\\
- "C:\\Windows\\System" 應該用 C:/Windows/System來代替 比較不會錯
- Windows可以接受「/」、「\\」，Linux只接受「/」
- .指的是當前目錄(windows)
- ./指的是當前目錄 (unix/linux)


## Linux on Windows
- WSL: WS1 可能少些東西 不過方便快速 / Docker可能會有問題 (安裝WSL2)
[WSL2](https://www.huanlintalk.com/2020/02/wsl-2-installation.html)
- VM: 很完整內核 