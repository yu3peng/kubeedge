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

输出设备网络信息, 此时记录 *ens3* 对应的 IPv4 地址, 分别为 172.17.0.C、172.17.0.E，待用

2. root用户操作

因为Kubernetes和Kubeedge都需要root权限, 因此以后尽量用root用户操作.

```shell
sudo vim /etc/ssh/sshd_config
```

将 *PermitRootLogin ???* 改为 *PermitRootLogin yes*

### 3. 安装

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





















