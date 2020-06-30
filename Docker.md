# Docker 概念


https://medium.com/unorthodox-paranoid/docker-tutorial-101-c3808b899ac6


虛擬化
- Container(打包應用程式) vs. VM(打包作業系統)
- Docker Image(唯獨 by dockerfile)
- Docker Container (可寫層 instace 類似物件導向的產生的實例)
- Docker Registry(倉庫)

client vs Server 架構
- client- /pull/run
- deamon tools 處理docker 物件
- 



# Docker 安裝步驟
[安裝步驟](https://medium.com/%E4%B8%80%E5%80%8B%E5%B0%8F%E5%B0%8F%E5%B7%A5%E7%A8%8B%E5%B8%AB%E7%9A%84%E9%9A%A8%E6%89%8B%E7%AD%86%E8%A8%98/docker-%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98-%E5%AE%89%E8%A3%9D-docker-8adb49a4c4ce)


我們可以使用 Apt 進行 Docker 的安裝。
sudo apt-get install docker.io
安裝完後，我們要先啟動 docker
sudo service docker start

sudo docker version

- 開通sudo權限
sudo usermod -aG docker <使用者名稱>



## 指令

1. 觀察

```
docker images`
docker p
```

2. Fetch (預設的registry是Docker Hub)
```
docker search <keyword> 
docker pull <image name>

docker pull ubuntu:latest
docker pull registry.hub.docker.com/ubuntu:latest
```




## 如何使用DOCKERFILE

- 一層一層layer疊上去

- 在目前目錄尋找Dockerfile或dockerfile並建立(冒號後面為Tag
docker build [--rm] [-t/--tag <tag name>] [--network <network name>]
docker build -t myimage:v1 .
docker build --rm -t node_image
(--rm = 此 Image 期間產生的中間 image 通通 remove (-rm) 掉)
[各種dockerfile的語法教學](https://www.jinnsblog.com/2018/12/docker-dockerfile-guide.html)

COPY的範例如下：

COPY file1.txt file2.js file3.json ./
COPY ["file1.txt", "file2.js", "file3.json" "./"]

```
# : 註解
FROM : 基底image
MAINTAINER : 維護者的資訊
RUN : 在container建立時執行的動作

example:
### Dockerfile
FROM python:3
RUN pip install --upgrade pip && \
    pip install --no-cache-dir  pymysql numpy pandas Flask gunicorn gevent Flask-Cors scipy
WORKDIR /
COPY requirement.txt gunicorn.conf.py web_api.py ./
## RUN pip3 install --no-cache-dir -r requirement.txt
CMD ["gunicorn", "web_api:app", "-c", "./gunicorn.conf.py"]
```



#### 注意dns for pip install
https://stackoverflow.com/questions/28668180/cant-install-pip-packages-inside-a-docker-container-with-ubuntu


## RUN啟動
docker run -it 名稱

- it 互動
-t:讓Docker分配到一個虛擬終端(pseudo-tty)，並綁定到容器的標準輸入上。
-i:則是讓容器的標準輸入(STDIN)保持開啟狀態(-i : 進入互動模式)


$ docker run -d -p 80:8888 --name docker0109 dockerfile_test
指令參數講解：
- d 背景執行
- p 將主機 8888 port 與 container的 80 port 綁定
- -p 8080:80
– name 為 container 命名

1. 檢查本機是否有指定image，若不存在則到registry下載
2. 使用image建立並啟動container
3. 分配到一個檔案系統，並在read layer(唯讀層)外掛載一層read-write layer(可讀寫層)
4. 從原電腦主機設定的網路介面橋接
5. 執行使用者指定的應用程式
6. 執行完畢後container被終止

## 終止Container/ 進入Container 



## 刪除image

使用 docker rmi <image name> 刪除指定的 image。
docker rmi <image name> / docker image rm <image name>
以剛剛產生的 new_testnode_img 為例，我們可以使用此指令刪除此 image
docker image rm new_testnode_img



## 如果container變動要commit上去

```
Usage: docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
docker commit -m "Added Git package" -a "Starter" 88400ddfbf99 ubuntu:v2
```



## 匯出/匯入 (實體IO)
()[https://ithelp.ithome.com.tw/articles/10191387]


docker save -o mytomcat.tar mytomcat
-o : 表示是寫入檔案；預設為寫入STDOUT
docker load -i ubuntu.tar
-i : 表示讀取tar檔；預設為使用STDIN。

## 上傳

docker push ubuntu:v3


## lask

https://zhuanlan.zhihu.com/p/78432719

pip install gunicorn gevent


在根目录下新建文件 /gunicorn.conf.py

workers = 5    # 定义同时开启的处理请求的进程数量，根据网站流量适当调整
worker_class = "gevent"   # 采用gevent库，支持异步处理请求，提高吞吐量
bind = "0.0.0.0:80"
gunicorn start:app -c gunicorn.conf.py
