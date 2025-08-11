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
echo "10.234.74.119 master" >> /etc/hosts
echo "192.168.10.23 node1" >> /etc/hosts
```

## 安装containerd

**如果没有源，先配置下**

```shell
cat > /etc/yum.repos.d/AnolisOS.repo << EOF
> [repo-base]
name=repo-base
baseurl=http://10.220.45.41/repos/CGSL/7/os/x86_64/os/
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CGSL-V7


[repo-update]
name=repo-update
baseurl=http://10.220.45.41/repos/CGSL/7/update/x86_64/os
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CGSL-V7
> EOF
```

### 安装
```shell
# containerd
dnf install containerd -y
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

### 修改配置

* `SystemdCgroup`: `sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml`
