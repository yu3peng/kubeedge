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
keadm init --kubeedge-version=1.2.1  --kube-config=/root/.kube/config
```

输出
```shell
Kubernetes version verification passed, KubeEdge installation will start...
Expected or Default KubeEdge version 1.2.1 is already downloaded
...
KubeEdge cloudcore is running, For logs visit:  /var/log/kubeedge/cloudcore.log
CloudCore started
```

查看日志 
```shell
tail -f /var/log/kubeedge/cloudcore.log
```

5. 删除节点
```shell
kubectl delete node node01
```

### 3. 安装 Kubeedge Edge

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

2. 拷贝密钥到Edge

```shell
mkdir -p /etc/kubeedge/
scp -r root@172.17.0.C:/etc/kubeedge/certs /etc/kubeedge/
scp -r root@172.17.0.C:/etc/kubeedge/ca /etc/kubeedge/
```

3. 将/etc/docker/daemon.json中的对应参数修改为cgroupfs

```shell
cat /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

重启docker

```shell
systemctl restart docker
```

3. 加入Kubeedeg集群

```shell
keadm join --cloudcore-ipport=172.17.0.C:10000 --edgenode-name=test1 --kubeedge-version=1.2.1
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

3. 退出 node01，查看 KubeEdge 集群节点状态

```shell
kubectl get nodes
```









