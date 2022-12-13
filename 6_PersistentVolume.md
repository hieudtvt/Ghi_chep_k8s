# PersistentVolumeClaims, PersistentVolumes
- Là một storage ở trong k8s
- Có thời gian tồn tại độc lập với pod và không bị mất đi khi pod bị xóa hay gặp sự cố
![](https://imgur.com/9loGyrb0.png)
## Persistent Volumes (pv)
- Là một phần không gian lưu trữ dữ liệu trong cụm, được cấp phát bởi Cluster Admin
- Là một loại tài nguyên của cụm, và tồn tại độc lập với Pod
## Persistent Volume Claim
- Một người muốn sử dụng PV thì cần tạo một PersistentVolumeClaim (pvc)
## Thực hành mối quan hệ giữa PersistentVolumes và PersistentVolumeClaim
- Ở trên worker node, tạo một thư mục `/mnt/data`
```
mkdir /mnt/data
```
- Tạo một file index trong `mnt/data`
```
echo 'test' > index.html
```
- Tạo một PersistentVolume bằng cách tạo file pv-volume.yaml với nội dung sau
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
- Trong đó:
  - ReadWriteOnce: là volume có thể được đọc và ghi bởi một single node, cho phép nhiều pod có thể truy cập với volume khi pod chạy ở cùng một node
  - ReadOnlyMany: volume có thể đọc bởi nhiều node
  - ReadWriteMany: volume có thể được ghi bởi nhiều node
  - ReadWriteOncePod: volume có thể được đọc ghi bởi Pod duy nhất
  - `path: "/mnt/data"`: là đường dẫn đến volume trên worker node
- Tạo một PersistentVolumeclaim, vì Pod sử dụng PersistentVolumeClaims để yêu cầu bộ nhớ 
- Ở đây, ta tạo một PersistentVolumeClaim có dung lượng 3GB để có thể đọc/ghi cho ít nhất một nốt
- Tạo file pv-claim.yaml có nội dung sau:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
- Câu lệnh tạo PersistentVolumeClaim: `kubectl apply -f pv-claim.yaml`
- Câu lệnh kiểm tra PersistentVolume `kubectl get pv task-pv-volume`
![](https://imgur.com/1aVBQdS.png)
- Câu lệnh kiểm tra PersistentVolumeClaim: `kubectl get pvc  task-pv-claim`
![](https://imgur.com/2o8oe4o.png)
- Tạo Pod sử dụng PersistentVolumeClaim như một volume
- file pv-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```
- Câu lệnh tạo Pod: `kubectl apply -f pv-pod.yaml`
- Kiểm tra Pod: `kubectl get pod task-pv-pod`
- Chui vào container bằng câu lệnh: `kubectl exec -it task-pv-pod -- /bin/bash`
- Vào thư mục `/usr/share/nginx/html` ta thấy có file index.html mà ta tạo ở trong /mnt/data/ ở trong worker node
![](https://imgur.com/1FNEOsv.png) tức là container đã sủ dụng PersistentVolume
- Xóa Pod, PersistentVolumeClaim và PersistentVolume:
```
kubectl delete pod task-pv-pod
kubectl delete pvc task-pv-claim
kubectl delete pv task-pv-volume
```
![](https://imgur.com/6LTRo6v.png)
- Kiểm tra thư mục /mnt/data/ ta thấy index.html vẫn còn, tức là dữ liệu không mất đi sau khi xóa pod
![](https://imgur.com/xwvgEl2.png)
- Mount cùng Persistentvolume ở hai nơi
- Nội dung file `pv-duplicate.yaml` 
```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
    - name: test
      image: nginx
      volumeMounts:
        # a mount for site-data
        - name: config
          mountPath: /usr/share/nginx/html
          subPath: html
        # another mount for nginx config
        - name: config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
  volumes:
    - name: config
      persistentVolumeClaim:
        claimName: test-nfs-claim
```
- 
