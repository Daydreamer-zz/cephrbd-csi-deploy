# ceph-csi-deploy
一个cephrbd和kubernetes StorageClass集成示例，驱动镜像已经同步至阿里云镜像仓库，直连拉取无压力，需要ceph集群提前创建供kubernetes使用的资源池
## 创建命名空间
```bash
kubectl create namespace ceph
```
## 修改csi-config-map.yaml
将clusterID和monitors修改为你的ceph集群信息
## 修改csi-rbd-secret.yaml
将userID和userKey修改为您的ceph集群认证信息
## 修改csi-rbd-sc.yaml
将clusterID修改为您的ceph集群fsid，pool为您的ceph资源池名称，imageFeatures根据您的内核版本选择，一般为layering无需更改
## 创建StorageClass
```bash
kubectl -n ceph create -f .
``` 
## 设置为默认StorageClass（可选）
如果需要配置为默认的StorageClass，请执行
```bash
kubectl patch storageclass csi-rbd-sc -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
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
