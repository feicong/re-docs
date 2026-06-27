# 安全研究员的Armbian配置完全指南

文档最新更新请访问：[https://github.com/feicong/re-docs/edit/master/Linux-armbian.md](https://github.com/feicong/re-docs/edit/master/Linux-armbian.md)

## 选择合适的Linux发行版

推荐使用**Ubuntu 26.04 LTS**，主要这是安卓系统与软件开发使用的官方Linux发行版本。

官方下载地址：[https://armbian.com/boards/rock-5b](https://armbian.com/boards/rock-5b)

下载链接直达：[https://dl.armbian.com/nightly/rock-5b/Resolute_vendor_gnome](https://dl.armbian.com/nightly/rock-5b/Resolute_vendor_gnome)

## 基础环境配置

首先配置系统的密码免输入与apt源加速：

```bash

export DEBIAN_FRONTEND=noninteractive
export TZ="Asia/Shanghai"

ARCH="$(uname -m)"

# 检查当前系统版本是否为 resolute
OS_VERSION=$(lsb_release -cs)
if [ "$OS_VERSION" != "resolute" ]; then
    echo "This script is designed for Ubuntu resolute. Exiting."
    exit 1
fi

echo "$(whoami) ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$(whoami)
sudo visudo -cf /etc/sudoers.d/$(whoami)

# Update apt sources list based on architecture
if [ "$ARCH" = "aarch64" ]; then
    sudo mv -f /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.bak
	sudo tee /etc/apt/sources.list.d/ubuntu.sources <<EOF

Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
Suites: resolute resolute-updates resolute-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# Types: deb-src
# URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# Suites: resolute resolute-updates resolute-backports
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 以下安全更新软件源为官方源配置
Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: resolute-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# Types: deb-src
# URIs: http://security.ubuntu.com/ubuntu/
# Suites: resolute-security
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 预发布软件源，不建议启用

# Types: deb
# URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# Suites: resolute-proposed
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# # Types: deb-src
# # URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# # Suites: resolute-proposed
# # Components: main restricted universe multiverse
# # Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

EOF
fi
```

### 更新系统

```bash
sudo apt update && sudo apt upgrade -y
```

### 安装必备工具

```bash
sudo apt update && sudo apt install -y git git-lfs curl wget axel unzip zip build-essential cmake \
    python3 python3-pip libpython3-dev upx bison make tree net-tools \
    ninja-build meson pkg-config libtool autoconf automake help2man llvm ruby vim \
    graphviz grep plantuml aria2 bash bash-completion bc binutils imagemagick brotli \
    jq repo reprepro coreutils libarchive-dev httpie lz4 shared-mime-info dbus lzip \
    android-sdk-libsparse-utils libelf-dev libevent-dev ffmpeg libffi-dev file flac tcpdump flex \
    fontconfig libncurses-dev texinfo libusbmuxd-dev gawk ca-certificates libmagic-dev \
    gettext unifdef libpcap-dev sed libplist-dev gnupg openssl z3 libslirp-dev zstd \
    lsb-release software-properties-common apt-file libdwarf-dev patchelf \
    libsqlite3-dev libunwind-dev gcc gdb tzdata socat strace libtool-bin 7zip libc6-dev \
    cloud-image-utils pahole apt-transport-https libguestfs-tools \
    gh glab witr fastfetch golang nodejs npm just gradle docker.io openjdk-21-jdk openjdk-25-jdk
```

### 配置docker

```bash
sudo usermod -aG docker $USER
newgrp docker
sudo mkdir -p /etc/docker
sudo vim /etc/docker/daemon.json

{
  "registry-mirrors": [
      "https://docker.m.daocloud.io",
      "https://docker.xuanyuan.me",
      "https://dockerhub.timeweb.cloud"
    ]
}
```

运行测试

```bash
sudo systemctl restart docker
docker pull ubuntu:26.04
```

### 设置Python环境

```bash
pip3 install --upgrade pip
```

## 开发环境配置

### Java配置

选择Java版本。

```bash
sudo update-alternatives --config java
```

### 环境变量配置

添加环境变量到 `~/.profile`：

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-arm64
export PATH=$PATH:$JAVA_HOME/bin

export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$HOME/Android/Sdk

export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator

export PATH=$PATH:$HOME/.npm/bin

alias codexx='codex --dangerously-bypass-approvals-and-sandbox'
alias claudex='claude --allowedTools=code,python,bash,browser,search,web,image,sql,http,git --dangerously-skip-permissions --permission-mode bypassPermissions'

source ~/.profile
```

## 配置

一些软件需要配置登陆与设置代理。

### gh

这个是github官方的命令行工具，管理仓库贼方便。登陆后就可以使用了。

```bash
gh auth login
```

### golang

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

### pip

```bash
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
pip install -U pip --break-system-packages
```

### npm

```bash
npm config set registry https://registry.npmmirror.com
```

安装一些工具。

```bash
npm install -g @anthropic-ai/claude-code @google/gemini-cli @openai/codex
```
