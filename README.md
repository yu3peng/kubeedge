| Kubeedge Cloud

| -- Ubuntu18.04 IP:172.17.0.C 
	    	    
| -- kubeedge-web-app:v2.7
	    
-- Kubeedge Edge
	   
| -- Ubuntu18.04 IP: 172.17.0.E 

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
NAME           STATUS   ROLES    AGE    VERSION
controlplane   Ready    master   140m   v1.14.0
node01         Ready    edge     57s    v1.17.1-kubeedge-v1.3.1
```

### 5. Cloud 部署kubeedge-web-app

```shell
git clone https://github.com/kubeedge/examples $GOPATH/src/github.com/kubeedge/examples
cd $GOPATH/src/github.com/kubeedge/examples

# 修改kubeedge-speaker-model.yaml文件
rm kubeedge-web-demo/kubeedge-web-app/deployments/kubeedge-speaker-model.yaml
sudo tee kubeedge-web-demo/kubeedge-web-app/deployments/kubeedge-speaker-model.yaml <<-'EOF'
apiVersion: devices.kubeedge.io/v1alpha2
kind: DeviceModel
metadata:
 name: speaker-model
 namespace: default
spec:
 properties:
  - name: track
    description: music track to play
    type:
     string:
      accessMode: ReadWrite
      defaultValue: ''
EOF

# 修改kubeedge-speaker-instance.yaml文件
rm kubeedge-web-demo/kubeedge-web-app/deployments/kubeedge-speaker-instance.yaml
sudo tee kubeedge-web-demo/kubeedge-web-app/deployments/kubeedge-speaker-instance.yaml <<-'EOF'
apiVersion: devices.kubeedge.io/v1alpha2
kind: Device
metadata:
  name: speaker-01
  labels:
    description: 'Speaker'
    manufacturer: 'test'
spec:
  deviceModelRef:
    name: speaker-model
  nodeSelector:
    nodeSelectorTerms:
    - matchExpressions:
      - key: ''
        operator: In
        values:
        - node01
EOF

# 修改kubeedge-web-app.yaml文件
rm kubeedge-web-demo/kubeedge-web-app/deployments/kubeedge-web-app.yaml
sudo tee kubeedge-web-demo/kubeedge-web-app/deployments/kubeedge-web-app.yaml <<-'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: kubeedge-web-app
  name: kubeedge-web-app
  namespace: default
spec:
  selector:
    matchLabels:
      k8s-app: kubeedge-web-app
  template:
    metadata:
      labels:
        k8s-app: kubeedge-web-app
    spec:
      nodeSelector:
        node-role.kubernetes.io/master: ""
      hostNetwork: true
      containers:
      - name: kubeedge-web-app
        image: yu3peng/kubeedge-web-app:v2.7
        imagePullPolicy: IfNotPresent
      restartPolicy: Always
EOF

# 创建资源文件
sudo tee /root/fabric8-rbac.yaml <<-'EOF'
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: fabric8-rbac
subjects:
  - kind: ServiceAccount
    # Reference to upper's `metadata.name`
    name: default
    # Reference to upper's `metadata.namespace`
    namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
EOF

# 部署资源文件
kubectl create -f /root/fabric8-rbac.yaml

# 允许master节点部署pod
kubectl taint nodes --all node-role.kubernetes.io/master-

# 部署
cd $GOPATH/src/github.com/kubeedge/examples
cd kubeedge-web-demo/kubeedge-web-app/deployments
kubectl create -f kubeedge-speaker-model.yaml
kubectl create -f kubeedge-speaker-instance.yaml
kubectl create -f kubeedge-web-app.yaml
```

### 6. 检查集群运行状态

```shell
$ kubectl get nodes
NAME           STATUS   ROLES    AGE    VERSION
controlplane   Ready    master   163m   v1.14.0
node01         Ready    edge     31m    v1.17.1-kubeedge-v1.3.1

$ kubectl get crds
NAME                                           CREATED AT
clusterobjectsyncs.reliablesyncs.kubeedge.io   2020-07-30T02:02:12Z
devicemodels.devices.kubeedge.io               2020-07-30T02:02:12Z
devices.devices.kubeedge.io                    2020-07-30T02:02:11Z
objectsyncs.reliablesyncs.kubeedge.io          2020-07-30T02:02:12Z

$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
kubeedge-web-app   1/1     1            1           29m

$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
kubeedge-web-app-5ccb4ddf7f-5vfbf   1/1     Running   0          30m
```

点击页面上的 Guestbook

在端口输入框中输入 80，然后点击 Display Port 按钮，网页不用关:

### 7. Edge(172.17.0.E) 部署客户端程序监听mqtt消息

```shell
ssh node01
cd /root/
sudo tee /root/main.go <<-'EOF'
package main

import (
	"encoding/json"
	"fmt"
	"github.com/kubeedge/kubeedge/cloud/pkg/devicecontroller/types"
	"github.com/yosssi/gmq/mqtt"
	"github.com/yosssi/gmq/mqtt/client"
)

func main() {
	fmt.Println("Get music list successfully")

	cli := client.New(&client.Options{
		// Define the processing of the error handler.
		ErrorHandler: func(err error) {
			fmt.Println(err)
		},
	})

	fmt.Println("Create mqtt client successfully")

	stopchan := make(chan int)
	// Terminate the Client.
	defer cli.Terminate()

	// Connect to the MQTT Server.
	err := cli.Connect(&client.ConnectOptions{
		Network:  "tcp",
		Address:  "localhost:1883",
		ClientID: []byte("receive-client"),
	})
	if err != nil {
		panic(err)
	}
	fmt.Println("Connect mqtt client successfully")

	err = cli.Subscribe(&client.SubscribeOptions{
		SubReqs: []*client.SubReq{
			{
				TopicFilter: []byte(`$hw/events/device/speaker-01/twin/update/document`),
				fmt.Printf("Unmarshal error: %v\n", err)
				QoS:         mqtt.QoS0,
				// Define the processing of the message handler.
				Handler: func(topicName, message []byte) {
					Update := &types.DeviceTwinDocument{}
					err := json.Unmarshal(message, Update)
					if err != nil {
						fmt.Println("Unmarshal error", err)
						fmt.Printf("Unmarshal error: %v\n", err)
					}
					fmt.Printf("%+v", Update)
				},
			},
		},
	})
	fmt.Println("Subscribe mqtt topic successfully")

	<-stopchan
	if err != nil {
		panic(err)
	} else {
		fmt.Println("Connection successfully")
	}
}
EOF

git clone https://github.com/kubeedge/kubeedge $GOPATH/src/github.com/kubeedge/kubeedge
go get github.com/yosssi/gmq/mqtt/client
go get github.com/yosssi/gmq/mqtt
go run /root/main.go
```

输出
```shell
Get music list successfully
Create mqtt client successfully
Connect mqtt client successfully
Subscribe mqtt topic successfully
```

### 8. 查看运行效果

Cloud上:

pod日志终端

```shell
kubectl logs -f kubeedge-web-app-76d4755f59-f5zbr
```

Edge上:

Kubeedge日志

```shell
tail -f /var/log/kubeedge/edgecore.log
```
MQTT客户端程序

```shell
go run main.go
```

选中8号歌曲歌曲, 点击网页Play按钮, 依次产生如下结果:

Cloud上:
```shell
kubectl logs -f kubeedge-web-app-xxxxx


```

Edge上:
```shell
tail -f /var/log/kubeedge/edgecore.log



```

```shell
go run main.go


```

### 参考资料
1. [Kubeedge安装,配置,HelloWorld](https://github.com/JingruiLea/blogs/blob/master/%E5%AE%89%E8%A3%85kubeedge.md)

