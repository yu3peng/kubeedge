### 1. 登录免费 Kubernetes 资源

打开两个web页面，分别登录以下网址
https://www.katacoda.com/courses/ubuntu/playground

认证后，点击 ***START SCENARIO*** 按钮

一个 Ubuntu 环境作为 Kubeedge Cloud，另外一个 Ubuntu 环境作为 Kubeedge Edge

### 2. 检查

1. 查看网络地址

```shell
ifconfig
```

输出设备网络信息, 此时记录 *ens3* 对应的 IPv4 地址, 分别为 172.17.0.C（Kubeedge Cloud）、172.17.0.E（Kubeedge Edge），待用

2. 更新并升级软件源, 安装必要工具

```shell
sudo apt-get update 
sudo apt-get upgrade
sudo apt-get install net-tools make vim openssh-server docker.io
```

3. root用户操作

因为Kubernetes和Kubeedge都需要root权限, 因此以后尽addr: ::1/128 Sco量用root用户操作.

```shell
sudo vim /etc/ssh/sshd_config
```

将 *PermitRootLogin prohibit-password* 改为 *PermitRootLogin yes*

### 3. 安装 Kubeedge Cloud

在 172.17.0.C（Kubeedge Cloud）上

1. 安装snap包管理, 通过snap安装kubernetes三件套

```shell
sudo apt-get install snap
sudo snap install kubectl --classic
sudo snap install kubelet --classic
sudo snap install kubeadm --classic
```

2. 下载Kubeedge源码

```shell
git clone https://github.com/kubeedge/kubeedge $GOPATH/src/github.com/kubeedge/kubeedge
```

3. 编译Keadm

```shell
cd $GOPATH/src/github.com/kubeedge/kubeedge
make all WHAT=keadm
```

4. 安装keadm

```shell
sudo cp ./_output/local/bin/keadm /usr/bin/
```

5. 下载kind, 部署kubernetes集群

```shell
cd /root/
GO111MODULE="on" go get sigs.k8s.io/kind@v0.7.0
kind version
```

手动下载kindest镜像
```shell
docker pull kindest/node:v1.17.2
```

创建kind配置文件
```shell
sudo tee /root/kind.yaml <<-'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "127.0.0.1"
  apiServerPort: 6443
nodes:
  - role: control-plane
    image: kindest/node:v1.17.2
EOF
```

创建集群
```shell
kind create cluster --config=/root/kind.yaml
```

检查结果
```shell
kubectl get nodes
```

输出
```shell
NAME                 STATUS     ROLES    AGE   VERSION
kind-control-plane   Ready      master   12s   v1.17.2
```

6. 创建kubeedge cloud节点

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

### 3. 安装 Kubeedge Edge

在 172.17.0.E（Kubeedge Edge）上

1. 拷贝 keadm 

```shell
scp root@172.17.0.C:/usr/bin/keadm /usr/bin/
```

2. 拷贝密钥到Edge

```shell
scp -r root@172.17.0.C:/etc/kubeedge/certs /etc/kubeedge/
scp -r root@172.17.0.C:/etc/kubeedge/ca /etc/kubeedge/
```

3. 加入Kubeedeg集群

```shell
su #切换root用户
keadm join --cloudcore-ipport=192.168.0.C:10000 --edgenode-name=test1 --kubeedge-version=1.2.1
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










