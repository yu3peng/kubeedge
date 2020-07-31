-- Kubeedge Cloud

| -- Ubuntu18.04 IP:172.17.0.C 
	    	    
| -- web-controller-app
	    
-- Kubeedge Edge
	   
| -- Ubuntu18.04 IP: 172.17.0.E 

| -- pi-counter-app

### 1. 登录免费 Kubernetes 资源

登录以下网址
https://www.katacoda.com/courses/kubernetes/guestbook

认证后，点击 ***START SCENARIO*** 按钮

### 2. 检查

查看网络地址

```shell
ifconfig
```

输出设备网络信息, 此时记录 *ens3* 对应的 IPv4 地址, 分别为 172.17.0.C（Kubeedge Cloud）

### 3. 安装 Kubeedge Cloud

在 172.17.0.C（Kubeedge Cloud）上

1. 下载Kubeedge源码

```shell
git clone https://github.com/kubeedge/kubeedge $GOPATH/src/github.com/kubeedge/kubeedge
```

2. 编译Keadm

```shell
cd $GOPATH/src/github.com/kubeedge/kubeedge
make all WHAT=keadm
```

3. 安装keadm

```shell
sudo cp ./_output/local/bin/keadm /usr/bin/
```

4. 创建kubeedge cloud节点

```shell
keadm init --kubeedge-version=1.3.1 --advertise-address=172.17.0.C
```

输出
```shell
Kubernetes version verification passed, KubeEdge installation will start...
Expected or Default KubeEdge version 1.3.1 is already downloaded
...
KubeEdge cloudcore is running, For logs visit:  /var/log/kubeedge/cloudcore.log
CloudCore started
```

查看日志 
```shell
tail -f /var/log/kubeedge/cloudcore.log
```

记录token
```shell
keadm gettoken
b603ecab6707991922dca354116c0b440fbf26eead9fbb919665f7769fa7be67.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTYxNzY5ODZ9.zw3vNXGATeFYP0ErBITbHnNNFw6vpZO0N8h
```

5. 删除节点
```shell
kubectl delete node node01
```

### 4. 安装 Kubeedge Edge

1. 登录到 node01 节点，停止 kubelet 进程

```shell
ssh node01
rm /usr/bin/kubelet
```

```shell
ps -aux | grep kubelet
```
获得进程号之后
```shell
kill -9 进程号
```

2. 拷贝 keadm 

```shell
scp root@172.17.0.C:/usr/bin/keadm /usr/bin/
```

3. 将/etc/docker/daemon.json中的对应参数修改为cgroupfs

```shell
vi /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

重启docker

```shell
systemctl restart docker
```

4. 加入Kubeedeg集群

在kubeedge v1.2.1版本中，采用的是证书，需要从cloud拷贝到edge
```shell
mkdir -p /etc/kubeedge/
scp -r root@172.17.0.C:/etc/kubeedge/certs /etc/kubeedge/
scp -r root@172.17.0.C:/etc/kubeedge/ca /etc/kubeedge/
```

在kubeedge v1.3.1版本中，采用的是token，
```shell
keadm join --cloudcore-ipport=172.17.0.C:10000 --edgenode-name=node01 --kubeedge-version=1.3.1 --token=b603ecab6707991922dca354116c0b440fbf26eead9fbb919665f7769fa7be67.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTYxNzY5ODZ9.zw3vNXGATeFYP0ErBITbHnNNFw6vpZO0N8h
```

输出
```shell
...
KubeEdge edgecore is running, For logs visit:  /var/log/kubeedge/edgecore.log
```

查看日志 
```shell
tail -f /var/log/kubeedge/edgecore.log
```

查看网络地址

```shell
ifconfig
```

输出设备网络信息, 此时记录 *ens3* 对应的 IPv4 地址, 分别为 172.17.0.E（Kubeedge Edge）

3. 退出 node01，查看 KubeEdge 集群节点状态

```shell
kubectl get nodes
```

输出
```shell
NAME           STATUS   ROLES        AGE   VERSION
controlplane   Ready    master       72m   v1.14.0
node01         Ready    agent,edge   18s   v1.17.1-kubeedge-v1.3.1
```

### 5. 部署 app
```shell
git clone https://github.com/yu3peng/examples.git $GOPATH/src/github.com/kubeedge/examples

# 创建device model
cd $GOPATH/src/github.com/kubeedge/examples/kubeedge-counter-demo/crds
kubectl create -f kubeedge-counter-model.yaml

# 创建device
cd $GOPATH/src/github.com/kubeedge/examples/kubeedge-counter-demo/crds
kubectl create -f kubeedge-counter-instance.yaml

# 部署云端应用
cd $GOPATH/src/github.com/kubeedge/examples/kubeedge-counter-demo/crds
kubectl apply -f kubeedge-web-controller-app.yaml

# 部署边缘端应用
cd $GOPATH/src/github.com/kubeedge/examples/kubeedge-counter-demo/crds
kubectl apply -f kubeedge-pi-counter-app.yaml

```

### 6. 体验Demo


### 参考资料
1. [Kubeedge安装,配置,HelloWorld](https://github.com/JingruiLea/blogs/blob/master/%E5%AE%89%E8%A3%85kubeedge.md)
2. [KubeEdge v1.3部署指南！](https://kubeedge.io/zh/blog/kubeedge-deployment-manual/)
