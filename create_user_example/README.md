# create a user of the kubernetes for the lico  
+ create a namespace of wujq(master node)
```
   kubectl create ns wujq
```
+ modify the server and  path of nfs  for the hpcuser_pv.yaml (copy the directory of create_user_example to the master node)
```
cd  create_user_example
cat hpcuser_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wujq-pv
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteMany
  storageClassName: wujq-pv-class
  nfs:
    server: 10.240.208.138    # change to your server 
    path: /home/wujq          # change to your path

```
+ create user (on the master node)
```
cd  create_user_example
kubectl create -f .
```
