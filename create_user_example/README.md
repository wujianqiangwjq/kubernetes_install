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
# import user for LiCO
+ get pvc for namespace "wujq"
```
[root@k8smaster ~]# kubectl  get pvc -n wujq ("wujq" is my namespace)
NAME           STATUS    VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS       AGE
wujq-pvc        Bound     wujq-pv   120Gi      RWX            wujq-pv-class   130d
```
pvc is "wujq-pvc"
+ get token for namespace "wujq"
```
[root@k8smaster ~]# kubectl  get secrets -n wujq ("wujq" is my namespace)
NAME                  TYPE                                  DATA   AGE
default-token-987c9   kubernetes.io/service-account-token   3      130d
```
secret'name  is  "default-token-987c9"
```
[root@k8smaster ~]# kubectl  describe  secret default-token-987c9  -n wujq
Name:         default-token-987c9
Namespace:    wujq
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: b9c281f3-fbaa-11e9-b014-7cd30ad833b8

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1363 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJocGN1c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tOTg3YzkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImI5YzI4MWYzLWZiYWEtMTFlOS1iMDE0LTdjZDMwYWQ4MzNiOCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpocGN1c2VyOmRlZmF1bHQifQ.dZX87xCXN4uWUpvXbv87l_5RS74E89rUnbGsX_f0bTDe4ZhNm8Y_ikhDQdcWmspsdqtsPRUmNOdzJEYrDSvJR5rzqAvTLwD9gCRjQWmHfz4dCVP55PBTPUXW4CxFY-_5gaXz7HvAQ2VFov95pdmd1N9DZQ6S5H6mFjPGbMt-zbRFSPgyr1hPvCgJF0P8uHbuUzSUyvNRfqOR8fFXwQlEFIJ3qzER5J1FYXbIJpYi6mVnk3KBpVFN-_llaVu4JyJ0_z2iB3qlRfS5pbXL5hnUYCpOX34tLpHqpPQCE2HROu1x2gnBl3i0cGImtOHXPZ-Ul5Vg8b1YrYAwDhylCRNs_Q
```
the token is what you want to get
