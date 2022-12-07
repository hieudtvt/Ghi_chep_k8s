# Cập nhật một ứng dụng đang chạy trong pods
- Ta có một ứng dụng đang chạy, được deploy bằng ReplicaSet, và có một service để expose traffic của nó cho client ra bên ngoài.
![](https://imgur.com/UbF2e2W.png)
- Có hai trường hợp muốn update lại các pod để chạy image mới 
## Recreate
- Cơ chế deploy:
  - Đầu tiên xóa toàn bộ phiên bản cũ của ứng dụng trước, sau đó deploy một version mới lên.
  - Quy trình sẽ là:
    - 1. Cập nhật lại Pod template của ReplicaSet
	- 2. Xóa toàn bộ Pod hiện tại để ReplicaSet tạo ra Pod với Image mới.
- Đặc điểm:
  - Quá trình deploy quá dễ dàng
  - Ứng dụng sẽ bị downtime với client trong quá trình deploy
## RollingUpdate
- Cơ chế:
  - Deploy từng version mới của ứng dụng
  - Kiểm tra trạng thái của pod, nếu đã chạy ta dẫn request tới version mới của ứng dụng
  - Lặp lại quá trình này cho tới khi toàn bộ version mới của ứng dụng được deploy và version mới bị xóa đi
- Đặc điểm:
  - Giảm quá trình downtime của ứng dụng khi deploy.
  - Điểm yếu là sẽ có version mới và version cũ của ứng dụng chạy chung một lúc.
  - Khó khăn trong việc phải viết script dể thực hiện.
# Deployment in K8s
- Là một resource của kurbernetes giúp ta trong việc cập nhật một version mới của ứng dụng một cách dẽ dàng.
## Thực hành deployment
- Tạo một file hello-deploy.yaml như sau:
```
apiVersion: apps/v1
kind: Deployment # change here
metadata:
  name: hello-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: 080196/hello-app:v1
        name: hello-app
        ports:
          - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: hello-app
spec:
  type: NodePort
  selector:
    app: hello-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 31000
```
- Kết quả tạo ra như sau:
![](https://imgur.com/xfbZW1k.png)
## Test deployment
- Ta có một image mới với source code sau
```
const http = require("http");

const server = http.createServer((req, res) => {
  res.end("Hello application v2\n") // change v1 to v2
});

server.listen(3000, () => {
  console.log("Server listen on port 3000")
})
```
- Câu lệnh để update lại ứng dụng trong Pod với Deployment
```
kubectl set image deployment hello-app hello-app=080196/hello-app:v2
```
- Ta có thể chọn phương thức Deploy khác bằng cách chỉ định trong thuộc tính **stategy**(mặc định nếu không khai báo sẽ chọn RollingUpdate).
```
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: hello-app
spec:
  replicas: 3
  strategy: # change here
    type: Recreate
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: 080196/hello-app:v1
        name: hello-app
        ports:
          - containerPort: 3000
```
# Sự khác biệt giữa RollingUpdate và Recreate
- Ta có ứng dụng đang chạy replicas trên 3 pod
![](https://imgur.com/84wkkvD.png)
## TH RollingUpdates
- File config yaml
```
apiVersion: apps/v1
kind: Deployment # change here
metadata:
  name: hello-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: 080196/hello-app:v1
        name: hello-app
        ports:
          - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: hello-app
spec:
  type: NodePort
  selector:
    app: hello-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 31000
```
- Khi cập nhật lại Deployment bằng câu lệnh
```
kubectl set image deployment hello-app hello-app=080196/hello-app:v2
```
![](https://imgur.com/kHwhwEU.png)
## TH Recreate
- File config yaml
```
apiVersion: apps/v1
kind: Deployment # change here
metadata:
  name: hello-app
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: 080196/hello-app:v1
        name: hello-app
        ports:
          - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: hello-app
spec:
  type: NodePort
  selector:
    app: hello-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 31000
```
- Trạng thái pod đang chạy
![](https://imgur.com/L6XCAyT.png)
- Khi cập nhật lại Deployment bằng câu lệnh
```
kubectl set image deployment hello-app hello-app=080196/hello-app:v2
```
- Lúc này các pod sẽ bị xóa đi để cập nhật lại (lúc này ứng dụng bị downtime)
![](https://imgur.com/IRkNeF2.png)
- Sau một lúc khi các pod cập nhật thành công lúc này pod lại quay trở về trạng thái running bình thường
![](https://imgur.com/mYQdtY4.png)
## Rollback lại ứng dụng về phiên bản trước
- Câu lệnh kiểm tra các phiên bản
```
kubectl rollout history deploy hello-app
```
- Câu lệnh quay lại phiên bản cũ
```
kubectl rollout undo deployment hello-app --to-revision=1
```
![](https://imgur.com/eqGtYjn.png)

