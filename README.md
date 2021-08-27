# ceph-csi-deploy
一个cephrbd和kubernetes StorageClass集成示例
## 创建命名空间
```bash
kubectl create namespace ceph
```
## 修改csi-config-map.yaml
将clusterID和monitors修改为你的ceph集群信息
## 修改csi-rbd-secret.yaml
将userID和userKey修改为您的ceph集群认证信息
## 修改csi-rbd-sc.yaml
将clusterID修改为您的ceph集群fsid
## 创建StorageClass
```bash
kubectl -n ceph create -f .
``` 
## 测试pvc
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-rbd-sc
```
## 测试pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: csi-rbd-demo-pod
spec:
  containers:
    - name: web-server
      image: docker.io/library/nginx:latest
      volumeMounts:
        - name: mypvc
          mountPath: /var/lib/www/html
  volumes:
    - name: mypvc
      persistentVolumeClaim:
        claimName: rbd-pvc
        readOnly: false
```
