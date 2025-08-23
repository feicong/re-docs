# 安全研究员的Linux配置完全指南

文档最新更新请访问：[https://github.com/feicong/re-docs/edit/master/Linux-arm64.md](https://github.com/feicong/re-docs/edit/master/Linux-arm64.md)

## 选择合适的Linux发行版

推荐使用**Ubuntu 24.04 LTS**，主要这是安卓系统与软件开发使用的官方Linux发行版本。

官方下载地址：[https://cdimage.ubuntu.com/releases/24.04/release/](https://cdimage.ubuntu.com/releases/24.04/release/)

下载链接直达：[https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04.3-desktop-arm64.iso](https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04.3-desktop-arm64.iso)

国内mirror下载链接列表：

[https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu/releases/24.04/release/ubuntu-24.04.3-desktop-arm64.iso](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu/releases/24.04/release/ubuntu-24.04.3-desktop-arm64.iso)

[https://mirrors.ustc.edu.cn/ubuntu-cdimage/releases/24.04/release/ubuntu-24.04.3-desktop-arm64.iso](https://mirrors.ustc.edu.cn/ubuntu-cdimage/releases/24.04/release/ubuntu-24.04.3-desktop-arm64.iso)


## 基础环境配置

首先配置系统的密码免输入与apt源加速：

```bash

export DEBIAN_FRONTEND=noninteractive
export TZ="Asia/Shanghai"

ARCH="$(uname -m)"

# 检查当前系统版本是否为 noble
OS_VERSION=$(lsb_release -cs)
if [ "$OS_VERSION" != "noble" ]; then
    echo "This script is designed for Ubuntu Noble. Exiting."
    exit 1
fi

echo "$(whoami) ALL=(ALL) NOPASSWD:ALL" | tee /etc/sudoers.d/$(whoami)
visudo -cf /etc/sudoers.d/$(whoami)

# Update apt sources list based on architecture
if [ "$ARCH" = "x86_64" ] || [ "$ARCH" = "i686" ] || [ "$ARCH" = "x86" ]; then
	sudo tee /etc/apt/sources.list.d/ubuntu.sources <<EOF

Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# Types: deb-src
# URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# Suites: noble noble-updates noble-backports
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# Types: deb-src
# URIs: http://security.ubuntu.com/ubuntu/
# Suites: noble-security
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 预发布软件源，不建议启用

# Types: deb
# URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# Suites: noble-proposed
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# # Types: deb-src
# # URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# # Suites: noble-proposed
# # Components: main restricted universe multiverse
# # Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

EOF
else
    sudo mv -f /etc/apt/sources.list /etc/apt/sources.list.bak || true
	sudo tee /etc/apt/sources.list.d/ubuntu.sources <<EOF

Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# Types: deb-src
# URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports
# Suites: noble noble-updates noble-backports
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
Types: deb
URIs: http://ports.ubuntu.com/ubuntu-ports/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# Types: deb-src
# URIs: http://ports.ubuntu.com/ubuntu-ports/
# Suites: noble-security
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 预发布软件源，不建议启用

# Types: deb
# URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports
# Suites: noble-proposed
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# # Types: deb-src
# # URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports
# # Suites: noble-proposed
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
sudo apt update && sudo apt install -y snapd git git-lfs curl wget axel unzip zip build-essential cmake \
    python3 python3-pip libpython3-dev openjdk-21-jdk upx flex bison make tree net-tools \
    ninja-build meson pkg-config libtool autoconf automake help2man llvm lua5.4 ruby vim \
    graphviz grep plantuml aria2 bash bash-completion bc binutils imagemagick brotli \
    jq repo reprepro quickjs coreutils libarchive-dev scrcpy httpie lz4 shared-mime-info dbus lzip \
    android-sdk-libsparse-utils libelf-dev libevent-dev ffmpeg libffi-dev file flac tcpdump flex \
    fontconfig libncurses-dev libfreetype-dev texinfo libusbmuxd-dev gawk ca-certificates libmagic-dev \
    gettext unifdef libnghttp2-dev libnghttp3-dev libnice-dev webp libpcap-dev sed libplist-dev gnupg \
    libpng-dev openssl gperf z3 libslirp-dev zstd p7zip binwalk \
    libcapstone-dev apktool smali gradle maven libzstd-dev libcurl4-openssl-dev libedit-dev lsb-release \
    software-properties-common apt-file libdwarf-dev libgirepository1.0-dev x11-apps patchelf libjson-glib-dev \
    libsoup-3.0-dev libsqlite3-dev libunwind-dev gcc gdb tzdata socat ltrace strace \
    libtool-bin p7zip-full libc6-dev gnome-tweaks net-tools openssh-server \
    neofetch dnsutils cloud-image-utils pahole gh apt-transport-https
```

一些工具使用snap下载最新版本.

```bash
sudo snap install intellij-idea-community --classic
sudo snap install clion --classic
sudo snap install go --classic
sudo snap install node --classic
sudo snap install protobuf --classic
sudo snap install postman --classic
sudo snap install just --classic
```

手动安装下面的工具：

```bash
# https://code.visualstudio.com/docs/setup/linux
echo "code code/add-microsoft-repo boolean true" | sudo debconf-set-selections
sudo apt-get install wget gpg apt-transport-https -y
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo install -D -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/microsoft.gpg
rm -f microsoft.gpg
echo  '
Types: deb
URIs: https://packages.microsoft.com/repos/code
Suites: stable
Components: main
Architectures: amd64,arm64,armhf
Signed-By: /usr/share/keyrings/microsoft.gpg
' | sudo tee -a /etc/apt/sources.list.d/vscode.sources
sudo apt update && sudo apt install code -y

wget https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v2.3.2/Clash.Verge_2.3.2_arm64.deb
sudo dpkg -i ./Clash.Verge_*.deb

wget https://dldir1v6.qq.com/qqfile/qq/QQNT/Linux/QQ_3.2.18_250724_arm64_01.AppImage # https://im.qq.com/linuxqq/index.shtml
wget https://dldir1v6.qq.com/weixin/Universal/Linux/WeChatLinux_arm64.AppImage # https://linux.weixin.qq.com/

chmod a+x *.AppImage
```

### 配置docker

```bash
sudo usermod -aG docker $USER
newgrp docker
sudo vim /etc/docker/daemon.json
{
  "registry-mirrors": [
      "https://docker.1ms.run",
      "https://docker.xuanyuan.me",
      "https://dockerhub.timeweb.cloud",
      "http://mirrors.ustc.edu.cn/",
      "http://mirror.azure.cn/",
      "https://docker.m.daocloud.io"
    ]
}
```

运行测试

```bash
sudo systemctl restart docker
docker pull ubuntu:24.04
```

### 设置Python环境

```bash
pip3 install --upgrade pip
```

## 开发环境配置

支持snapd安装方式与手动安装方式。

### 手动配置安装

下载地址：[https://developer.android.com/?hl=zh-cn](https://developer.android.com/?hl=zh-cn)

```bash
wget https://redirector.gvt1.com/edgedl/android/studio/ide-zips/2025.1.2.12/android-studio-2025.1.2.12-linux.tar.gz
sudo tar -xvzf android-studio-*.tar.gz -C /opt/
/opt/android-studio/bin/studio.sh
```

接下来安装，Android SDK和NDK。可以在Android Studio手动安装或者命令行安装。

```bash
mkdir -p $HOME/Android/Sdk
wget https://dl.google.com/android/repository/commandlinetools-linux-13114758_latest.zip -O cmdline-tools.zip
unzip cmdline-tools.zip -d $HOME/Android/Sdk
mv $HOME/Android/sdk/cmdline-tools $HOME/Android/Sdk/cmdline-tools-latest
mkdir -p $HOME/Android/Sdk/cmdline-tools
mv $HOME/Android/sdk/cmdline-tools-latest $HOME/Android/Sdk/cmdline-tools/latest
```

### 自动配置安装

```bash
sudo snap install android-studio --classic
```

### Java配置

选择Java版本。

```
sudo update-alternatives --config java
```


### 环境变量配置

添加环境变量到 `~/.bashrc`：

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-arm64
export PATH=$PATH:$JAVA_HOME/bin

export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$HOME/Android/Sdk

export NDK_ROOT=$ANDROID_HOME/ndk/28.2.13676358
export ANDROID_NDK_ROOT=$ANDROID_HOME/ndk/28.2.13676358
export ANDROID_NDK=$ANDROID_HOME/ndk/28.2.13676358
export PATH=$PATH:$ANDROID_NDK_ROOT

export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
```

生效：

```bash
source ~/.bashrc
```


## 逆向分析工具

### 反编译和调试工具

```bash
pip3 install frida-tools --break-system-packages
```

### IDA Pro / Ghidra

- 下载并安装 **Ghidra**（开源）

- Ghidra 安装：

```bash
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.4.1_build/ghidra_11.4.1_PUBLIC_20250731.zip
unzip ghidra_*.zip -d $HOME/tools
```

## 配置

一些软件需要配置登陆与设置代理。

### gh

这个是github官方的命令行工具，管理仓库贼方便。登陆后就可以使用了。

```bash
gh auth login
```

### vscode配置

参考[macOS的vscode配置](https://github.com/feicong/re-docs/blob/master/macOS.md#vscode%E9%85%8D%E7%BD%AE)


### golang

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

### pip

设置pip的mirror。

```bash
pip install -U pip --break-system-packages
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

### npm

设置npm的mirror。

```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
npm config set registry https://registry.npmmirror.com
```

安装一些js工具。

```bash
npm install -g @anthropic-ai/claude-code @google/gemini-cli typescript
```

### maven

```bash
mkdir -p ~/.m2

echo '<?xml version="1.0" encoding="UTF-8"?> 
<settings xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="settings.xsd">
    <mirrors>
        <mirror>
            <id>aliyunmaven</id>
            <mirrorOf>*</mirrorOf>
            <name>阿里云公共仓库</name>
            <url>https://maven.aliyun.com/repository/public</url>
        </mirror>

        <mirror>
            <id>huaweicloud</id>
            <mirrorOf>*</mirrorOf>
            <url>https://repo.huaweicloud.com/repository/maven/</url>
        </mirror>

        <mirror>
            <id>nexus-163</id>
            <mirrorOf>*</mirrorOf>
            <name>Nexus 163</name>
            <url>http://mirrors.163.com/maven/repository/maven-public/</url>
        </mirror>

        <mirror>
            <id>nexus-tencentyun</id>
            <mirrorOf>*</mirrorOf>
            <name>Nexus tencentyun</name>
            <url>http://mirrors.cloud.tencent.com/nexus/repository/maven-public/</url>
        </mirror>
    </mirrors>
</settings>' > ~/.m2/settings.xml
```

maven项目中，执行`mvn install`命令即可看到效果。

### gradle

依赖mirror：

```bash
mkdir -p ~/.gradle

echo 'fun RepositoryHandler.enableMirror() {
    all {
        if (this is MavenArtifactRepository) {
            val originalUrl = this.url.toString().removeSuffix("/")
            urlMappings[originalUrl]?.let {
                logger.lifecycle("Repository[$url] is mirrored to $it")
                this.setUrl(it)
            }
        }
    }
}

val urlMappings = mapOf(
    "https://repo.maven.apache.org/maven2" to "https://mirrors.tencent.com/nexus/repository/maven-public/",
    "https://dl.google.com/dl/android/maven2" to "https://mirrors.tencent.com/nexus/repository/maven-public/",
    "https://plugins.gradle.org/m2" to "https://mirrors.tencent.com/nexus/repository/gradle-plugins/"
)

gradle.allprojects {
    buildscript {
        repositories.enableMirror()
    }
    repositories.enableMirror()
}

gradle.beforeSettings { 
    pluginManagement.repositories.enableMirror()
    dependencyResolutionManagement.repositories.enableMirror()
}' > ~/.gradle/init.gradle.kts
```

设置代理：

```bash
echo '
# https://docs.gradle.org/current/userguide/networking.html
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=7890
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=7890
' ~/.gradle/gradle.properties
```

## 其他优化

### 提高文件打开数限制

```bash
echo "fs.file-max = 100000" | sudo tee -a /etc/sysctl.conf
ulimit -n 100000
```

### 设置免密码sudo

```bash
echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER
```

### 配置不同版本内核

```bash
sudo add-apt-repository ppa:cappelikan/ppa -y
sudo apt update && sudo apt install mainline -y
sudo mainline install 6.6
```

### 安装cuttlefish

```bash
tar xf debs.tar
sudo dpkg -i ./cuttlefish-common_*_*64.deb || sudo apt-get install -f
sudo dpkg -i ./cuttlefish-base_*_*64.deb || sudo apt-get install -f
sudo dpkg -i ./cuttlefish-user_*_*64.deb || sudo apt-get install -f
sudo dpkg -i ./cuttlefish-integration_*_*64.deb || sudo apt-get install -f
sudo usermod -aG kvm,cvdnetwork,render $USER
sudo reboot
```
