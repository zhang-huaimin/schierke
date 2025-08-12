# docker

## 保存镜像

`docker save -o calico-v3.28.5-x86.tar calico/kube-controllers:v3.28.5 calico/cni:v3.28.5 calico/node:v3.28.5`

## containerd

### 安装

**如果没有源，先配置下**

```shell
cat > /etc/yum.repos.d/AnolisOS.repo << EOF
[os]
name=os
baseurl=http://$ip/repos/CGSL/7/os/x86_64/os/
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CGSL-V7


[update]
name=update
baseurl=http://$ip/repos/CGSL/7/update/x86_64/os
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CGSL-V7
EOF
```

### 安装
```shell
# containerd
dnf install containerd nerdctl -y
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

## registry

### 安装containerd

[containerd](docker.md#containerd)

### 安装registry

```shell
docker pull registry:2.8.3
# nerctl import registry.2.8.3.tar
```

```shell
nerdctl run -d -p 5000:5000 --restart=always --name registry registry:2.8.3
# 确认启动
curl http://$ip:5000/v2/_catalog
```

### 客户端如何使用http registry

```toml
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."$ip:5000"]
          endpoint = ["http://$ip:5000"]
      
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."$ip:5000"]
        insecure_skip_verify = true  # 跳过 TLS 验证
      [plugins."io.containerd.grpc.v1.cri".registry.configs."$ip:5000".http]
        skip_cert_verify = true  # 强制使用 HTTP
```