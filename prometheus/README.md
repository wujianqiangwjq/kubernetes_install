# prometheus 的安装有多种方式
+ 二进制安装
+ prometheus operator(coreos出的),operator只是管理prometheus, prometheus还需要另外安装的。
+ kube-prometheus (其中包含有grafana, prometheus ,alter-manager, prometheus operator,...)

这里介绍kube-prometheus安装,当前环境
+ kubectl  1.15.0
+ kube-prometheus 0.3

##  步骤:
+ *wget  https://github.com/coreos/kube-prometheus/archive/v0.3.0.tar.gz*
+ *tar -xf v0.3.0.tar.gz*
+ *cd  kube-prometheus-0.3.0*
+ *创建CRD: kubectl create -f manifests/setup*
+ *kubectl create -f manifests/setup -f manifests*
+ *这时会建一个monitoring的namespace,检查pod运行状态,一切正常时，将一些type改为NodePort*
```
将granfana nodeport 改为30030
将alertmanager-main nodeport 改为30093
将prometheus-k8s nodeport 改为30090
[root@master kube-prometheus-0.3.0]# kubectl  get svc -n monitoring
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
alertmanager-main       NodePort    10.1.112.27    <none>        9093:30093/TCP               37m
alertmanager-operated   ClusterIP   None           <none>        9093/TCP,9094/TCP,9094/UDP   37m
grafana                 NodePort    10.1.194.139   <none>        3000:30030/TCP               37m
kube-state-metrics      ClusterIP   None           <none>        8443/TCP,9443/TCP            37m
node-exporter           ClusterIP   None           <none>        9100/TCP                     37m
prometheus-adapter      ClusterIP   10.1.87.98     <none>        443/TCP                      37m
prometheus-k8s          NodePort    10.1.10.166    <none>        9090:30090/TCP               37m
prometheus-operated     ClusterIP   None           <none>        9090/TCP                     37m
prometheus-operator     ClusterIP   None           <none>        8080/TCP                     38m
```
# 登录grafana， 默认用户名密码:  admin/admin


### 如何添加serviceMonitor，添加GPU为例
> GPU监控官方推荐使用DCGM(datacenter gpu manager)
+ *download: https://github.com/NVIDIA/gpu-monitoring-tools.git*
## 添加步骤：
+ 为gpu节点打label
```
 kubectl label nodes wjqmaster hardware-type=NVIDIAGPU
```
+ 我选择在gpu-monitoring-tools/exporters/prometheus-dcgm/k8s/node-exporter下的gpu-only-node-exporter-daemonset.yaml
> 可能需要修改一点
+ 为其编写service, 如gpu-service.yaml
+ 为其编写serviceMonitor 如gpu-serviceMonitor.yaml
