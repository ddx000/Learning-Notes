# K8S

# CH2 & CH3
k run nginx --image=nginx
k api-resources -o wide
--dry-run=client -o yaml
k get pod <podname> -o yaml > 1.yaml
- pod & svc | v1
- rs & deploy | apps/v1

-o name
-o wide
-o yaml
    
    
    
Yaml basic: https://ithelp.ithome.com.tw/articles/10193509

## node
Kubernetes 運作的最小硬體單位，一個 Worker Node（簡稱 Node）對應到一台機器
每個 Worker Node 中都有三個組件：kubelet、kube-proxy、Container Runtime。
Master Node (Kubernetes 運作的指揮中心)

## rs
k scale --relicas=6 rs replicasetName


## deploy
k create deploy <deployname> --image=<imagename> 
k scale deploy <deployname> --replicas=3
    

## svc
service 多個pod抽象成唯一個服務
Service 就是 Kubernetes 中用來定義「一群 Pod 要如何被連線及存取」的元件。
> 帶著相同標籤、做類似事情的一群 Pod
> Service 作為中介層，避免使用者和 Pod 進行直接連線，除了讓我們服務維持一定彈性能夠選擇不同的 Pod 來處理請求外，某種程度上亦避免裸露無謂的 Port 而導致資安問題
> 
- ClusterIP
> ClusterIP Service 是 K8s 提供的內部反向代理
> 內部服務訪問
- NodePort
> 對外暴露服務
- LoadBalancer 
>  對外暴露服務(公有雲ex AWS,GCP)

servicename.namespace.svc.cluster.local
db-service.default.svc.cluster.local

k expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
    
(use the pod's labels as selectors)
    
k expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml


k create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
    
k create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
    
(This will not use the pods labels as selectors)
    
    
    
## NS

k config set-context $(k config current-context) -n=dev
    
-n namespace
-A --all-namespace all-namespace

## switch context
    
A context is the connection to a specific cluster (username/apiserver host) used by kubectl. You can manage multiple clusters that way. Namespace is a logical partition inside a specific cluster to manage resources and constraints.    
    
    
k config current-context
k config use-context CONTEXT_NAME
k config get-contexts
    
## docker & k8s args

FROM ubuntu
CMD sleep 5
    
docker run ubuntu-sleeper sleep 10

FROM ubuntu
ENTRYPOINT ["sleep"]

docker run ubuntu-sleeper 10
    
entrypoint is something in default and you append something
    
```
spec:
  containers:
    - name: sleeper
      command: ["sleep2.0"] # overwrite entrypoint
      args: ["10"] # 塞進去container的args
```
    
ENTRYPOINT => command
CMD => args
    

## ENV
    
```
spec:
  containers:
    - name: sleeper
      ports:
        - containerPort: 8080
      env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_FAREWELL
          valueFrom:
            configMapKeyRef:
              name: special-config
              # Specify the key associated with the value
              key: special.how
```

    
## CM & secrets
kc cm app-config --from-literal=COLOR=BLUE
![](https://i.imgur.com/PDGqTX3.png)

ke pods --recursive | grep envFrom -A3
-A3：顯示匹配行之後的 3 行內容，數字可自由調整。
-B3：顯示匹配行之前的 3 行內容，數字可自由調整。
-C3：顯示匹配行前後的 3 行內容，數字可自由調整。
若希望 grep 在搜尋時可以不區分英文大小寫，可以加上 -i 參數：

kc create secret generic <secret-name> --from-literal=<key>=<va>
    
secret 裡面的yaml 必須base64 encoded
echo -n "mysql" | base64
echo -n "fsdfds" | base64 --decode
    
kg secrets
kd secrets
kg secret app-secret -o yaml
### get 配上 -o yaml!

![](https://i.imgur.com/cfwQpvo.png)

## security-context

![](https://i.imgur.com/oinJmsF.png)


## enter a pod of k8s
k exec -it shell-demo -- /bin/bash

## multi-container pods
SIDECAR : log server


    
## service account
machine use svc account to connect to k8s  
```kubectl create sa <name>```
when you create a sa, you also create a secret, and the secret mount inside sa  
在創serviceaccount時 會自動創建一個secret給外部service call api時帶的bearer token  

為了指定serviceAccountName in Pod
automountServiceAccountToken: false

## Kubectl explain

```kubectl explain <resource name> --recursive | less``` 
    
## Taint & Tolerations

k taint nodes node-name key=value:taint-effect
k taint nodes node1 app=blue:NoSchedule
    
taint is for node
toleration is for pods
- spec.tolerations

NoSchedule | NoExecute-> evict
這概念是跟node說不要接受某種特定的pod 但是不是確保pod一定會進去node
    

## Node Selectors
- pod上定義 spec.nodeSelector
- label nodes
k label nodes <node-name> <label-key>=<label-value>
- Node Affinity
處理一些像是not nodeSelector

# CH5. Observability
    
## Readiness & Liveness Probeds
    
## logs
Examining pod logs
First, look at the logs of the affected container:

kubectl logs ${POD_NAME} ${CONTAINER_NAME}
If your container has previously crashed, you can access the previous container's crash log with:

kubectl logs --previous ${POD_NAME} ${CONTAINER_NAME}

k top nodes
    
# CH6. Pod Design 
    
## selectors

k get pods --selector app=App1
-l, --selector="": Selector (label query) to filter on
kubectl get all --selector name1=value1,name2=value2,name3=value3


## rollout

k edit ...
k rollout status deployment/app
k rollout history deployment/app
k rollout undo deployment/app

    
## Updating a deployment

k create deployment nginx --image=nginx:1.16
k rollout status deployment nginx
k rollout history deployment nginx
    
### check the status of each revision individually
k rollout history deployment nginx --revision=1
    

### We can use the --record flag to save the command used to create/update   
k set image deployment nginx nginx=nginx:1.17 --record
k edit ... --record
    
### undo 
k rollout undo deployment nginx

    
## Jobs & CronJobs
- 如果只用pods 去計算的話 做完一個task 就會shutdown 然後不斷重啟(restartPolicy) 這不是我們要的
apiVersion: batch/v1
kind: Job
metadata:
spec:
  completions: 3 # 先1再2再3 成功三次就可以
  parallelism: 3 # 並行三個
  template:
    spec:
      
- apiVersion: batch/v1beta1
kind: CronJob
metadata:
spec:
  schedule: "*/1 * * * *"
  jobTemplate"
     spec:
     
# CH7. Service & Networking
  
- NodePort
公開被外部訪問的port 從30000-32767
- Port 
port是暴露在cluster ip上的埠
port = cluster port
<cluster ip>:port 是提供给集群内部客户访问service的入口。
- TargetPort
targetPort是pod上的埠
- containerPort
containerPort是在pod控制器中定義的
https://iter01.com/523068.html

kind: Service
spec:
  type: NodePort
  ports:
    - targetPort: 80 # 如果不提供 假設跟port一下
      port: 80 #就是clusterIPport
      nodePort: 30008 # 不提供則random
  selector:
    app: myapp # 選一群pod


## INGRESS
  k8s內建的load balancer & routing
  - Ingress controller (Nginx)
  - Ingress resource
  
  ingress controller (use nginx as default) 是一個deployment 需要有configmap 與serviceaccount
  然後需要nodeport 把controller expose出去
  nodeport(Svc) -> ingress controller (with cm+sa) --> ingress resoucres
  configmap for ing controller 是為了config nginx其他參數方便
  ingress controller 自己有智慧 可以去抓ingress resource

![](https://i.imgur.com/yCSRHQu.png)
![](https://i.imgur.com/N63RaW5.png)
![](https://i.imgur.com/88lGYpJ.png)
![](https://i.imgur.com/4KVGSOL.png)
use host or paths
- if didn't specify host, accept all host

Now, in k8s version 1.20+ we can create an Ingress resource from the imperative way like this:-

Format - k create ingress <ingress-name> --rule="host/path=service:port"

Example - k create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"

kd ing <ingress-name>
  
## Network policy
![](https://i.imgur.com/cZ4WVNH.png)
![](https://i.imgur.com/f6b0V4Y.png)


# State Persistence
- pv
  accessModes:
    - readwriteOnce
  capacity:
    - storage: 1Gi
  awsElas....
  
- pvc
Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim
  
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim

volume種類
https://godleon.github.io/blog/Kubernetes/k8s-Volume-Overview/
- emptyDir: emptyDir 會在 pod 被分配到特定的 node 後產生，一開始永遠是一個空的目錄
- emptyDir 的 life cycle 是跟著 pod + node 一起走，只要該 pod 持續的在原本的 node 上運行，emptyDir 就會保留著；相反的，只要兩個條件有任一消失，emptyDir 的資料也會跟著消失
- 注意: 容器異常時並不會導致 Pod 服務被移除，故 emptyDir 的數據是安全的，需等到 Pod 完全被移除才會一併刪除 emptyDir 資料。
- 當服務創立後，該掛載空間也會一併被建立，並賦予讀寫的權限，但當服務被移除後該空間一併會被移除，適合存放不重要資料。
  
```
  spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

  
hostPath
- 若是希望 volume 新增後裏面就存放一些 node 上已經存在的資料，可以使用 hostPath。  
- 負責將機器上「特定路徑」的文件掛載到 Container 內，且當服務因某些原因重新啟動，或者需要重新建置，檔案仍保存在 Node 上，直到該文件被移除或者該 Node 被移除 Kubernetes Cluster。

## kubectl 內建 縮寫
    
ns = namespace
sa = serviceaccount
cm = configmap
-n = --namespace
sc = storage class

## kubectl expose

```kubectl expose deploy hello-deployment --type=NodePort --name=my-deployment-service ```