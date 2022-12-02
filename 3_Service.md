# Kubernetes Services
- Là một dạng services có chức năng expose trafic cho Pod để Pod có thể giao tiếp và xử lý request từ client
- Mỗi service sẽ có một địa chỉ IP và port không đổi
- Khi client mở connection tới service, service sẽ dẫn connection đó một trong những Pod ở phía sau.
![](https://imgur.com/ZiGmeT8.png)
## Tại sao phải dùng service
- Các pod khi được tạo ra sẽ có thể bị xóa và thay thế bằng một pod khác
- Các worker node có thể bị die, Pod cũng bị die theo và Pod mới sẽ được tạo trên worker khác.
- Khi pod mới được tạo ra, nó sẽ có một IP khác, lúc này, nếu ta dùng IP của Pod để tạo connection tới client thì ta phải update lại code
## Cách quản lý connection
- Services sử dụng label  để chọn Pod mà nó quản lý connection
- Service có 4 loại cơ bản 
  - ClusterIP
  - NodePort
  - ExternalName
  - LoadBalancer
## ClusterIP
- Là loại service sẽ tạo ra một IP và local DNS dùng để truy cập ở bên trong cluster, không thể truy cập từ bên ngoài, được dùng chủ yếu cho các Pod ở bên trong cluster dễ dàng giao tiếp với nhau.
## NodePort
- Là một trong những cách để expose Pod cho client bên ngoài có thể truy cập vào được Pod bên trong cluster.
- Bên ngoài sẽ giao tiếp với Pod thông qua IP và port của NodePort
- Port có range mặc định từ 30000 - 32767
![](https://imgur.com/qJ1AO4h.png)
### Thực hành NodePort
- Ở bài trên ta đã có pod chạy rs, bây giờ để client có thể truy cập tới Pod ta cần tạo một service dạng NodePort như sau:
- Tạo một file hello-nodeport.yaml như sau:
```
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: hello-kube
  type: NodePort # type NodePort
  ports:
    - port: 80
      targetPort: 3000
      nodePort: 30123 # port of the worker node
```
- Trong đó:
  - **port: 80**: là port của service
  - **targetPort: 8080**: là port của Pod
  - **nodePort: 30123**: là port của Worker node
- Câu lệnh chạy: `kubectl apply -f hello-nodeport.yaml`
- Test truy cập:
![](https://imgur.com/MqxTK4l.png)
![](https://imgur.com/b5gJrvh.png)


