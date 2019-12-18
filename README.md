# install kubernetes with kubeadm
### the design of cluster
```
k8s-master	192.168.0.2	
k8s-node1	192.168.0.3
k8s-node2	192.168.0.4
```
### prepare installation environment  (k8s-master, k8s-node1, k8s-node2)
+ close swap, firewalld and selinux
```
systemctl stop firewalld
systemctl disable firewalld
sed -i 's/enforcing/disabled/' /etc/selinux/config 
setenforce 0
swapoff -a

edit /etc/fstab to commont the code of swap
```
+ configure the hosts and sync to all nodes
```
cat << EOF > /etc/hosts
192.168.0.2 k8s-master
192.168.0.3 k8s-node
192.168.0.4 k8s-node2
EOF
```
+ open traffic forwarding
```
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sysctl --system
```
# install kubernetes
+ prepare the resources of docker and kubernetes (k8s-master, k8s-node1, k8s-node2)
```
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum makecache
```
+ install  docker and kubernetes (k8s-master, k8s-node1, k8s-node2)
```
yum -y install docker-ce
systemctl start docker
yum install -y kubelet-1.15.0 kubeadm-1.15.0 kubectl-1.15.0
```
# configure the cluster of kubernetes
+ on the master node(k8s-master)
```
kubeadm init \
  --apiserver-advertise-address=192.168.0.2 \
  --kubernetes-version v1.15.0 \
  --service-cidr=10.1.0.0/16 \
  --pod-network-cidr=10.244.0.0/16
  
 If you can't access foreign websites:
  kubeadm init \
  --apiserver-advertise-address=192.168.0.2 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.15.0 \
  --service-cidr=10.1.0.0/16 \
  --pod-network-cidr=10.244.0.0/16

the step will output:
  kubeadm join 192.168.0.2:6443 --token japatq.5vib0jhpgmeeqsb2 \
    --discovery-token-ca-cert-hash sha256:c08f2729dbe9e2b1ce9f44e6d3159c493cc686b2e93dc252f7658cb26b87d726
  
```
+ make the configuration of kubernetes for client (k8s-master)
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
+ install flannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```
+ add new node to the cluster on all nodes
```
(k8s-node1, k8s-node2)
kubeadm join 192.168.56.12:6443 --token japatq.5vib0jhpgmeeqsb2 \
    --discovery-token-ca-cert-hash sha256:c08f2729dbe9e2b1ce9f44e6d3159c493cc686b2e93dc252f7658cb26b87d726
```
# check the cluster
+ check the status of nodes
```
kubectl get nodes
output:
      NAME     STATUS   ROLES    AGE     VERSION
      k8s-master  Ready    master   6m34s   v1.15.0
      k8s-node1   Ready    <none>   2m33s   v1.15.0
      k8s-node2   Ready    <none>   2m11s   v1.15.0
      
```
+ check the network of flannel
```
kubectl  run redis1  --image redis
kubectl get pod -o wide
output:
    NAME                      READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE ...
    redis1-5bb687c47b-q72pd   1/1     Running   0          3m46s   10.244.2.2   k8s-node2   <none>
on the node of k8s-master:   ping   10.244.2.2
on the node of k8s-node1:    ping   10.244.2.2
on the node of k8s-node2:    ping   10.244.2.2
```
# if you want to make the master node to be schedulered
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```
# set service (k8s-master, k8s-node1, k8s-node2)
```
systemctl enabel kubelet
```
# install the plugin of GPU(nvidia)
+ install nvidia driver(k8s-node2)
```
yum install -y https://us.download.nvidia.cn/tesla/418.67/nvidia-diag-driver-local- repo-rhel7-418.67-1.0-1.x86_64.rpm
yum install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r) gcc gcc-c++
yum install -y cuda-drivers
```
+ install the plugin of gpu
```
kubectl apply -f nvidia-device-plugin.yml
```
# install the kubernetes-dashboard
+ prepare secrets (k8s-master)
```
mkdir  dashboard
cd dashboard
    openssl req -nodes -newkey rsa:2048 -keyout dashboard.key -out dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"
    openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt 
cd ..
 kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard -n kube-system
```
+ install kubernetes-dashboard
```
kubectl apply -f kubernetes-dashboard.yaml
```
# check kubernetes-dashboard
+ get secret for bashboard
```
kubectl describe secret $(kubectl  describe sa kubernetes-dashboard  -n kube-system|grep Tokens|awk '{print $NF}') -n kube-system
output:
.....
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi05cXg4biIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjRhNTE2ZGIzLWJiMmQtNDZlZS04NmZlLWE5ODBiOGQxNDFiYyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.UxUPydr4nmU0Yr7dXbbqjYNV8bRuBA2hWN3Fe3DW9viUSrjbGn_iqBWGIh6MFS_GRyfxOiH1l9WFatu52iJf5GzjQnaCSGCqFI0jA9iytnOUYw68iAl9krOuo1O9MocwV-uwPWK8ULSWloRZODnftiwH_gT9cF-dt8s9nCDO7lxcIjcDCbNdW9TVArDEdMgajWUqOQjhvnY5KalGbk2X1qyv4rwTF82GSQIslvhdW3TDrfEv7whcaUfPCvDIeVKUMJU93aLMSCWqDYsMLurY7jk536ezulyGn3e1WyUXD8ax1r20QiI2P-vwIMYfoT0miMAt3dTjEqsV_DAsjZzT7Q
```
+ get port for dashboard
```
kubectl  get svc -n kube-system
output:

NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.1.0.10     <none>        53/UDP,53/TCP,9153/TCP   164m
kubernetes-dashboard   NodePort    10.1.11.190   <none>        443:32459/TCP            10m
```
+ log in the dashboard
![image](https://github.com/wujianqiangwjq/kubernetes_installl/blob/master/images/kubernetes-dashboard.JPG)
![image](https://github.com/wujianqiangwjq/kubernetes_installl/blob/master/images/dashboard1.JPG)

# install  ingress
```
kubectl apply -f ingress-nginx.yaml
```
# check ingress
```
kubectl  get pod -n ingress-nginx
  output:
      NAME                                        READY   STATUS    RESTARTS   AGE
      nginx-ingress-controller-79f6884cf6-q9sh2   1/1     Running   0          7m17s

kubectl  logs nginx-ingress-controller-79f6884cf6-q9sh2  -n ingress-nginx
```
+ check the port of ingress
```
kubectl  get svc -n ingress-nginx
     output:
         NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
         ingress-nginx   NodePort   10.1.25.188   <none>        80:31796/TCP,443:32313/TCP   9m4s

```

