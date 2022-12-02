# Pod
![](https://imgur.com/RVw2wRq.png)
## Command
### Info pod
- `kubectl get pods`: xem một số thông tin pod như sau:

![](https://imgur.com/qne7e3L.png)
  - **name**: tên của pod mà ta định nghĩa ở **metadata** 
  - **READY**: số lượng container đã ready-ok trong pod
  - **Status**: cho biết trạng thái của pod đang là gì
    + ContainerCreating: đang tạo
	+ Running: đang chạy
  - **Restart**: số lần khởi động lại của pod
  - **Age**: thời gian tính từ lúc pod được tạo
### Chui vào pod
- `kubectl exec -it my-app -- bash`: chui vào trong pod
![](https://imgur.com/vZTGasy.png)
### Check log của pod
`kubectl logs my-app`
![](https://imgur.com/WBCYQXP.png)
### Check all info pod

`kubectl describe my-app` 
![](https://imgur.com/YqU1dm7.png)

