# Volume 
- Là một mount point từ hệ thống file của server vào bên trong container
- Dữ liệu khi ta ghi vào filesystem chỉ tồn tại khi container còn chạy. Khi một Pod bị xóa và tạo lại, container mới sẽ được tạo ra và dữ liệu của container trước sẽ bị mất đi
- Volume có nhiệm vụ dữ lại những dữ liệu đó
## Các loại volume hay dùng
- emptyDir
- hostPath
- cloud storage
- PersistentVolumeClaim
## emptyDir
- Là loại volume đơn giản, nó sẽ tạo ra directory bên trong Pod 
- Các container ở trong Pod có thể ghi dữ liệu vào bên trong nó
### Đặc điểm
- Một emptyDir volume sẽ được khởi tạo với vùng lưu trữ rỗng (khởi tạo trên ổ cứng của K8s Node) đi theo một Pod được khởi động trên K8s Node
- Một emptyDir volume có thể được mount với bất kỳ mount point nào trong các container của Pod.
- Các containers trong cùng một 1 Pod có thể đọc/ghi với emptyDir Volume
- Volume chỉ tồn tại trong một lifecycly của pod
- Dữ liệu trong loại volume này chỉ được lưu trữ tạm thời và sẽ bị mất đi khi Pod bị xóa
- Ta dùng loại volume này khi muốn các container có thể chia sẻ dữ liệu lẫn nhau và không cần lưu trữ dữ liệu lại
- EmptyDir volume không dùng cho việc lưu trữ các dữ liệu mang có yêu cầu thời gian dài hạn và toàn vẹn dữ liệu như (database, dữ liệu ứng dụng, dữ liệu giám sát hệ thống,...)

![](https://imgur.com/undefined.png)
### Thực hành emptyDir
- Tạo 2 container trong cùng một Pod:
  - Container `myvolumes_1` sẽ mount emptyDir volume vào thư mục `/demo1`
  - Container `myvolume_2` sẽ mount emptyDir volume vào thư mục `/demo2`
```
# vi myVolumes-Pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - image: alpine
    name: test-volume-1

    command: ['sh', '-c', 'tail -f /dev/null']


    volumeMounts:
    - mountPath: /demo1
      name: demo-volume   

   
  - image: alpine
    name: test-volume-2

    command: ['sh', '-c', 'tail -f /dev/null']

    volumeMounts:
    - mountPath: /demo2
      name: demo-volume

  volumes:
  - name: demo-volume
    emptyDir: {}
```
- Sau khi tạo thành công 2 container, ta truy cập vào container test-volume-1 bằng câu lệnh
```
kubectl exec test-pod -c test-volume-1 -i -t -- sh
```
- Tạo một file trong thư mục demo1
```
echo 'test' > test.txt
```
- Sau đó thoát container test-volume-1, ta truy cập vào test-volume-2 bằng câu lệnh
```
kubectl exec test-pod -c test-volume-2 -i -t -- sh 
```
- Truy cập vào thư mục demo2 ta thấy file test.txt vừa tạo trên container test-volume-2

![](https://imgur.com/rqbf5y5.png)

## hostPath
![](https://imgur.com/YmD3yvY.png)

- Là loại volume sẽ tạo một mount point từ Pod ra ngoài filesystem của node. 
- Dữ liệu lưu trong volume này chỉ tồn tại trên một worker nod và sẽ không bị xóa đi khi Pod bị xóa.
### Thực hành 
- Tạo file như sau
```
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-volume
spec:
  containers:
    - image: nginx:alpine
      name: web-server
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
        - name: time
          mountPath: /etc/localtime
        - name: test-log # log volume
          mountPath: /var/log/nginx # mounted at /var/log/nginx in the container
      ports:
        - containerPort: 80
          protocol: TCP
  volumes:
    - name: html
      gitRepo: # gitRepo volume
        repository: https://github.com/luksa/kubia-website-example.git # The volume will clone this Git repository
        revision: master # master branch
        directory: . # cloned into the root dir of the volume.
    - name: test-log
      hostPath: # hostPath volume
        path: /var/log # folder of woker nod
    - name: time
      hostPath:
        path: /usr/share/zoneinfo/Asia/Ho_Chi_Minh
```
![](https://imgur.com/bV0Ov5p.png)
- Trong đó: 
  - `mountPath: /usr/share/nginx/html`: đường dẫn của pod mà ta sẽ mount ra ngoài filesystem của Workernode
  - `name: html`: là tên volume
  - `hostPath`: khai báo dạng volume
  - `path: /var/log`: vị trí nằm ở workernode mà volume **html** sẽ mount tới























