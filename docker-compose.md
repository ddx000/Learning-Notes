---
tags: Tool
---

# docker-compose


## install

```
 $ sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
 $ sudo chmod +x /usr/local/bin/docker-compose
 $ docker-compose --version
```

## reduce size
- alpine linux


## Dockerfile
ENV是在build的時候，可以定義一些變數，讓後面指令在執行時候可以參考...
```
ENV NODE_VER=node-v5.9.1-linux-armv7l
ADD ./${NODE_VER} /
```
上面Dockerfile的第二行，定義NODE_VER變數，讓第三行中的ADD參考，因此ADD會在執行時間，使用ENV所定義的值來做ADD的動作。



## volume
利用volume儲存container 的data, 並不用每次都commit
當你使用 volume 時，docker 會在你的本機上隨機新增一個資料夾（Local storage area），
大部分會在 /var 底下，然後讓這個資料夾跟 container 裡面的某個資料夾互通。
![](https://i.imgur.com/Bys65Mr.png)
```
docker volume create --name db-data
docker volume ls
```

## command

```
docker-compose up <-d> # 啟動 detached(background)
# docker-compose up 包含了 build 、 create 與 start 指令


docker-compose ps # list all container
docker-compose logs <container_name> # show logs all or certain containers
docker-compose stop # 停止，但不刪除container
docker-compose down # 停止，並移除container(-v 刪除volume)
# docker-compose down 包含了 stop 與 rm 指令
```


## compose.yml example

```

 version: "3"
 
 volumes: # 自定義volume，位於host /var/lib/docker/volumes内
  myproject_db_vol: # 這邊都有點像是變數先命名
  myproject_redis_vol: 
  myproject_media_vol: 
 
 services:
  redis:
    image: redis:5
    command: redis-server /etc/redis/redis.conf # 容器啟動後要下什麼command
    volumes:
       - myproject_redis_vol:/data # 前面是host的位置 / 後面是container的位置<host:container>
       - ./compose/redis/redis.conf:/etc/redis/redis.conf # 把配置文件mount進去
    ports:
       - "6379:6379"
       # 請務必記得port需要使用雙引號當成字串處理，以免docker在解析上出問題。
       # 前面<host port>後面container的port
       # ports:能連到主機的這些 port 都能夠使用 // expose: 僅能在此 docker-compose 內的 container 們使用。
     restart: always
 
  db:
    image: mysql:5.7
    environment:
       - MYSQL_ROOT_PASSWORD=123456 
       - MYSQL_DATABASE=myproject
       - MYSQL_USER=dbuser
       - MYSQL_PASSWORD=password 
    # 注意 建議可以用 env_file:的方式寫，保護環境變數，後面會介紹
    volumes: # 一樣<host: container>
       - myproject_db_vol:/var/lib/mysql:rw # 掛載host的db_vol進去container, 冒號後面給權限
       - ./compose/mysql/conf/my.cnf:/etc/mysql/my.cnf 
       - ./compose/mysql/init:/docker-entrypoint-initdb.d/ 
    ports:
       - "3306:3306" 
     restart: always
     
  web:
    build: ./myproject # 使用myproject目錄底下的Dockerfile
    expose:
       - "8000" # expose僅為對內部開放
    volumes:
       - ./myproject:/var/www/html/myproject # 放進去項目代碼
       - myproject_media_vol:/var/www/html/myproject/media # 存取媒體vol
       - ./compose/uwsgi:/tmp 
    links:
       - db
       - redis
    depends_on: # 依賴關係 先啟動db和redis才啟動web
       - db
       - redis
    environment:
       - DEBUG=False
     restart: always
    tty: true
    stdin_open: true
 
  nginx:
    build: ./compose/nginx
    ports:
       - "80:80"
       - "443:443"
    expose:
       - "80"
    volumes:
       - ./myproject/static:/usr/share/nginx/html/static # 挂载静态文件
       - ./compose/nginx/ssl:/usr/share/nginx/ssl # 挂载ssl证书目录
       - ./compose/nginx/log:/var/log/nginx # 挂载日志
       - myproject_media_vol:/usr/share/nginx/html/media # 挂载用户上传媒体文件
    links:
       - web
    depends_on:
       - web
     restart: always
```
 



## link 
[docker 學習筆記](https://peihsinsu.gitbooks.io/docker-note-book/content/)
[djagno部屬](https://zhuanlan.zhihu.com/p/145364353)





- sudo apt-get upgrade & update
https://askubuntu.com/questions/1109982/e-could-not-get-lock-var-lib-dpkg-lock-frontend-open-11-resource-temporari


# 從頭到尾的Flow
https://testdriven.io/blog/dockerizing-django-with-postgres-gunicorn-and-nginx/



```
# pull official base image
FROM python:3.8.3-alpine

# set work directory
WORKDIR /home/ddx/Desktop/django/myresume

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# copy project
COPY . .
```
