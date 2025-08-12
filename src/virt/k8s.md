k8s
===

**基于龙蜥23.1**

# 部署

## 禁用安全

* 禁用SElinux: `setenforce 0 && sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config`
* 禁用防火墙: `systemctl stop firewalld && systemctl disable firewalld`
* 禁用swap: `swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab`

## 启用ip转发

```shell
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay && modprobe br_netfilter
```

## 设置sysctl

```bash
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
vm.swappiness=0
EOF
sysctl --system
```

## 配置主机名

* `master`: `hostnamectl set-hostname master`
* `worker`: `hostnamectl set-hostname node1`

## 配置/etc/hosts

```shell
echo "$master_ip master" >> /etc/hosts
echo "$node1_ip node1" >> /etc/hosts
```

## 安装containerd

[安装containerd](./docker.md/#containerd)

### 修改配置

```shell
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
systemctl enable containerd && systemctl restart containerd

# 可选，将该行替换
# sandbox_image = "registry.k8s.io/pause:3.6"
sandbox_image = "$registry/pause:3.9"
```

## 安装kubernetes

```shell
wget https://mirrors.openanolis.cn/anolis/23.3/os/x86_64/os/Packages/kubernetes-1.27.8-3.an23.x86_64.rpm
wget https://mirrors.openanolis.cn/anolis/23.3/os/x86_64/os/Packages/kubernetes-client-1.27.8-3.an23.x86_64.rpm
wget https://mirrors.openanolis.cn/anolis/23.3/os/x86_64/os/Packages/kubernetes-master-1.27.8-3.an23.x86_64.rpm
wget https://mirrors.openanolis.cn/anolis/23.3/os/x86_64/os/Packages/kubernetes-kubeadm-1.27.8-3.an23.x86_64.rpm
wget https://mirrors.openanolis.cn/anolis/23.3/os/x86_64/os/Packages/kubernetes-node-1.27.8-3.an23.x86_64.rpm
```

```shell
dnf install ./kubernetes*.rpm
systemctl enable kubelet
```

## 依赖containerd

```shell
echo 'KUBELET_EXTRA_ARGS="--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock --cgroup-driver=systemd"' > /etc/sysconfig/kubelet
```

## 安装cni-plugin

```shell
wget https://github.com/containernetworking/plugins/releases/download/v1.7.1/cni-plugins-linux-amd64-v1.7.1.tgz
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.7.1.tgz
```

## 初始化server

```shell
# ip: server服务器ip
# registry: 容器镜像仓库
# 如何reset: kubeadm reset --force
# kubeadm存在一些镜像依赖, 通过搜索`kubeadm config images list`得到, 将它们放到镜像仓库里
kubeadm init --apiserver-advertise-address=$ip --image-repository $registry --kubernetes-version v1.27.8 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=Swap,NumCPU

mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
```


```shell
curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.5/manifests/calico.yaml -O
# registry: 容器镜像仓库
# calico存在一些镜像依赖, 通过搜索'docker.io'字符串搜索得到, 将它们放到镜像仓库里
sed -i 's|docker.io|$registry|g' calico.yaml > calico_modified.yaml
kubectl apply -f calico.yaml
``` 

## 注册worker

```
kubeadm join $master_ip:6443 --token $token --discovery-token-ca-cert-hash sha256:$sha256
```

# 常用命令

* `kubectl get pods -n kube-system -w`
* `kubectl apply -f python-app.yaml` | `kubectl delete -f python-app.yaml`
* `kubectl get pods` =>  `kubectl describe pod python-app-67b8945c4f-n64qx`

# 资料

* 查找calico和k8s的版本关系: https://docs.tigera.io/calico/3.28/getting-started/kubernetes/requirements
* https://www.linuxtechi.com/how-to-install-kubernetes-cluster-rhel/
* 解决cgroup子系统无cpu.*文件的问题. `The cpu controller in cgroup v2 can not be used in Red Hat Enterprise Linux 8`: https://access.redhat.com/solutions/6582021

* python-app.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
    spec:
      containers:
      - name: python
        image: $registry/python:3.11  # 使用你的私有 registry 地址
        command: ["python", "-c", "print('Hello from private registry!')"]
```


