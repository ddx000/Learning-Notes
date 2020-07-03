# Flask WEB API

```python3 

from flask import Flask,request
from flask_cors import CORS,cross_origin

app = Flask(__name__)
CORS(app)
app.config["DEBUG"] = True


@app.route('/')
@cross_origin()
def test():
    return 'hello'


@app.route('/test',methods=['GET'])
@cross_origin()
def test2():
    data = request.args.get('val')
    return data
    
    
if __name__ == '__main__':
    app.run()

```

http://127.0.0.1:5000/test?val=123

# multi-threaded & process

Flask 有支援 multi-threaded
app.run(threaded=True)
如果沒開啟這個選項，Flask 會以 single thread 並且阻斷其他 request
但是由於Python GIL限制 所以可以改用Gunicorn


# Gunicorn
基於 Python WSGI 的 HTTP Server，介於 Nginx 與 Flask 中間，可以協助安排 workers 來處理 request，來應對 Process/thread 不同的需求，並且可以附加一些套件來做追蹤。
![](https://i.imgur.com/tcgxJ1b.png)


$ pip install gunicorn
$ gunicorn web_api:app -c ./gunicorn_conf.py
啟動webapi.py裡面，app.run()起來



gunicorn參數:  
-w 4 同時開啟多少進程數
-b 127.0.0.1:4000 綁定到什麼地址端口
-c 直接定義 gunicorn_conf.py
```
loglevel = 'debug'
accesslog = "log/access.log"
errorlog = "log/debug.log"

workers = 5    
# 定義多少process，取決於CPU數量
# 建議可以設為multiprocessing.cpu_count() * 2 + 1

worker_class = "gevent" 
# gevent 支持async

bind = "0.0.0.0:5000"
```
# Nginx

如果只是單純的API，就不用額外包一層NGINX
[Nginx、Gunicorn在服务器中分别起什么作用？](https://www.zhihu.com/question/38528616)
Nginx主要功能是
1. 負載均衡
2. 靜態文件配置
3. 抗並發能力(連線限制等等)

# 正向代理與反向代理

https://kknews.cc/zh-tw/tech/k66p2gb.html

![](https://i.imgur.com/iqN3dfN.png)
![](https://i.imgur.com/Lpr8Wco.png)

不論是正向代理和反向代理，都能提高速度
正向代理，是指代理"客戶端"去送出請求，伺服器會不知道誰是真的客戶端
反向代理，是指代理"伺服器"去接收請求，客戶端會不知道誰是真的伺服器

正向代理:
突破訪問限制，提高速度

反向代理:
隱藏伺服器真實IP
負載均衡-->轉發需求給不同的真實伺服器
提供安全保障


# Gunicorn配置

[通过优化 Gunicorn 配置提高性能](https://juejin.im/post/5ce8cab8e51d4577523f22f8)

- worker就是Process數量
- Gunicorn下的worker還可以產生thread
但是由於GIL限制，這個Thread只能用在io bound


有一些 Python 库比如（gevent 和 Asyncio）可以在 Python 中启用多并发。那是基于协程实现的“伪线程”。
gunicorn --worker-class=gevent --worker-connections=1000 --workers=3 main:app
連結數最多3process*1000thread-connections


