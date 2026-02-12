# Dev

## opensuse/tumbleweed源配置

```shell
zypper mr -da
zypper ar -p 98 -cfg 'https://mirrors.tuna.tsinghua.edu.cn/opensuse/tumbleweed/repo/oss/' mirror-oss
zypper ar -p 98 -cfg 'https://mirrors.tuna.tsinghua.edu.cn/opensuse/tumbleweed/repo/non-oss/' mirror-non-oss
zypper ar -p 98 -cfg 'https://mirrors.tuna.tsinghua.edu.cn/opensuse/update/tumbleweed/' mirror-factory-update
zypper ref
```

## 基础命令

```shell
zypper install ripgrep fd git make gcc gcc-c++ python313 python313-devel go go-devel cmake freetype-devel fontconfig-devel libxcb-devel libxkbcommon-devel
```

## cargo

```shell
zypper install cargo rustc

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
mkdir -p ~/.config/uv/uv.toml
cat > ~/.config/uv/uv.toml << EOF
[[index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
default = true
EOF

zypper install python313 python313-devel

zypper install python313-uv

mkdir -p ~/.pip
cat > ~/.pip/pip.conf << EOF
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = http://pypi.tuna.tsinghua.edu.cn" > ~/.pip/pip.conf
EOF
```


## alacritty

```shell
zypper install alacritty
# or
cargo install alacritty
```