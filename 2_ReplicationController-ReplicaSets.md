# ReplicationControllers and RelicaSets
## ReplicationControllers
- Là một resource dùng để tạo và quản lý pod
- Thực hiện đảm bảo số lượng pod nó quản lý không thay đổi và kept running
- Số lượng pod được tạo ra theo chỉ định và được quản lý thông qua labels của pod
![](https://imgur.com/jYxLbex.png)
### Thực hành ReplicationControllers
- Tạo file hello-rc.yaml với nội dung sau
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: hello-rc
spec:
  replicas: 2 # number of the pod
  selector: # The pod selector determining what pods the RC is operating on
    app: hello-kube # label value that ReplicationController match to create pod replicas
  template: # pod template
    metadata:
      labels:
        app: hello-kube # label value of pod
    spec:
      containers:
      - image: 080196/hello-kube # image used to run container
        name: hello-kube # name of the container
        ports:
          - containerPort: 3000 # pod of the container
```
- Trong đó:
  - **replicas: 2**: chỉ định số lượng pod được tạo ra
  - **selector:**: chỉ định pod được RC quản lý thông qua nhãn
  - **template**: pod được tạo ra sẽ có config như khai báo ở template
- Câu lệnh tạo RC: `kubectl apply -f hello-rc.yaml`
![](https://imgur.com/pBW8FiL.png)
## Thay đổi template của pod
- Khi thay đổi template và cập nhật lại RC, nhưng nó sẽ không apply cho những pod hiện tại
- Muốn pod hiện tại cập nhật template mới thì phải xóa hết pod để RC tạo ra pod mới hoặc xóa RC và tạo lại
- Câu lệnh xóa RC: `kubectl delete rc hello-rc`
- Tuy nhiên nếu xóa RC thì các pod mà nó quản lý cũng bị xóa theo
## ReplicaSets
- Là resource tương tự như RC, nhưng là phiên bản mới hơn của RC
### Thực hành ReplicaSets
- Tạo một file hello-rs.yaml có nội dung sau:
```
apiVersion: apps/v1 # change version API
kind: ReplicaSet # change resource name
metadata:
  name: hello-rs
spec:
  replicas: 2
  selector:
    matchLabels: # change here 
      app: hello-kube
  template:
    metadata:
      labels:
        app: hello-kube
    spec:
      containers:
      - image: 080196/hello-kube
        name: hello-kube
        ports:
          - containerPort: 3000
```
- Câu lệnh tạo rs:`kubectl apply -f hello-rs.yaml`
- Câu lệnh kiểm tra rs:`kubectl get rs`
## DaemonSets 
- Là một resource giám sát và quản lý pod theo labels
![](https://imgur.com/26MbfwI.png)
- Tuy nhiên, nếu như sử dụng rs thì pod có thể deploy ở bất kỳ node nào và trong một node có thể chạy mấy pod cũng được.
- Sử dụng daemonset thì pod sẽ chỉ được deploy tới mỗi node duy nhất, tức là có bao nhiêu node thì sẽ có bấy nhiêu pod, sẽ không có thuộc tính replicas
- Ứng dụng của daemonset là dùng cho monitoring và logging

