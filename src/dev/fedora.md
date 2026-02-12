# fedora配置

## repo

```shell
# /etc/yum.repos.d/fedora.repo
[fedora]
name=Fedora $releasever - $basearch
failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/fedora/releases/$releasever/Everything/$basearch/os/
#metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
enabled=1
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[fedora-debuginfo]
name=Fedora $releasever - $basearch - Debug
failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/fedora/releases/$releasever/Everything/$basearch/debug/tree/
#metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-debug-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[fedora-source]
name=Fedora $releasever - Source
failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/fedora/releases/$releasever/Everything/source/tree/
#metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-source-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False
```

## base

```shell
dnf install vim-enhanced fish ripgrep fd-find git make cmake gcc gcc-c++ python3 cargo rustc go perl docker-compose docker-cli zstd alacritty -y

systemctl enable sshd
systemctl restart sshd
```

## cargo
```shell
dnf install cargo rustc -y

echo 'export RUSTUP_DIST_SERVER="https://rsproxy.cn"' >> ~/.bashrc
echo 'export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"' >> ~/.bashrc
echo 'export PATH=$PATH:/$HOME/.cargo/bin' >> ~/.bashrc

. ~/.bashrc

mkdir -p ~/.cargo
cat > ~/.cargo/config.toml << EOF
[source.crates-io]
replace-with = 'rsproxy-sparse'
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"
[net]
git-fetch-with-cli = true
EOF
```

## python3

```shell
dnf install python3-uv -y

mkdir -p ~/.config/uv
cat > ~/.config/uv/uv.toml << EOF
[[index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
default = true
EOF

mkdir -p ~/.pip
cat > ~/.pip/pip.conf << EOF
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = http://pypi.tuna.tsinghua.edu.cn" > ~/.pip/pip.conf
EOF
```

## docker

```shell
systemctl enable docker
systemctl restart docker
```