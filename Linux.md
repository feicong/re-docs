# 安全研究员的Linux配置完全指南

文档最新更新请访问：[https://github.com/feicong/re-docs/edit/master/Linux.md](https://github.com/feicong/re-docs/edit/master/Linux.md)

## 选择合适的Linux发行版

推荐使用**Ubuntu 24.04 LTS**，主要这是安卓系统与软件开发使用的官方Linux发行版本。

官方下载地址：[https://releases.ubuntu.com/24.04/](https://releases.ubuntu.com/24.04/)

下载链接直达：[https://releases.ubuntu.com/24.04/ubuntu-24.04.3-desktop-amd64.iso](https://releases.ubuntu.com/24.04/ubuntu-24.04.3-desktop-amd64.iso)

国内mirror下载链接列表：

[https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.3-desktop-amd64.iso](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.3-desktop-amd64.iso)

[https://mirrors.ustc.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.3-desktop-amd64.iso](https://mirrors.ustc.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.3-desktop-amd64.iso)


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
	tee /etc/apt/sources.list.d/ubuntu.sources <<EOF

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
	tee /etc/apt/sources.list.d/ubuntu.sources <<EOF

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
sudo add-apt-repository ppa:wireshark-dev/stable -y
```

### 安装必备工具

```bash
sudo apt update && sudo apt install -y snapd git git-lfs curl wget axel unzip zip build-essential cmake \
    python3 python3-pip libpython3-dev openjdk-21-jdk openjdk-17-jdk upx wireshark flex bison make tree net-tools \
    ninja-build meson pkg-config libtool autoconf automake help2man llvm lua5.4 ruby valac kotlin vim \
    graphviz grep plantuml aria2 bash bash-completion bc binutils imagemagick brotli qemu-user qemu-system \
    jq repo reprepro quickjs coreutils libarchive-dev scrcpy httpie lz4 shared-mime-info dbus lzip \
    android-sdk-libsparse-utils libelf-dev libevent-dev ffmpeg libffi-dev file flac mpg123 tcpdump flex \
    fontconfig libncurses-dev libfreetype-dev texinfo libusbmuxd-dev gawk ca-certificates libmagic-dev \
    gettext unifdef libnghttp2-dev libnghttp3-dev libnice-dev webp libpcap-dev sed libplist-dev gnupg \
    libpng-dev openssl x264 x265 mpv gobject-introspection googletest gperf z3 libslirp-dev zstd p7zip binwalk \
    libcapstone-dev apktool smali gradle maven libzstd-dev libcurl4-openssl-dev libedit-dev lsb-release \
    software-properties-common apt-file libdwarf-dev libgirepository1.0-dev x11-apps patchelf libjson-glib-dev \
    libjsonrpc-glib-1.0-dev libsoup-3.0-dev libsqlite3-dev libunwind-dev gcc gdb libglib2.0-dev libgraphviz-dev \
    libgee-0.8-dev libsoup2.4-dev libgstreamerd-3-dev gtk-3-examples libgtk-3-bin libgtk-3-common libgtk-3-dev \
    gtk-4-examples libgtk-4-bin libgtk-4-common libgtk-4-dev libvala-*dev tzdata iputils-ping socat ltrace strace \
    libtool-bin p7zip-full libc6-dev fuse docker.io docker-buildx docker-compose gnome-tweaks net-tools openssh-server \
    neofetch libfuse2 libfuse3-dev dnsutils cloud-image-utils qemu-utils genisoimage pahole
```

一些工具使用snap下载最新版本.

```bash
sudo snap install intellij-idea-community --classic
sudo snap install pycharm-community --classic
sudo snap install clion --classic
sudo snap install gh --classic
sudo snap install go --classic
sudo snap install node --classic
sudo snap install code --classic
sudo snap install protobuf --classic
sudo snap install postman --classic
sudo snap install just --classic
```

手动安装下面的工具：

```bash
wget https://github.com/balena-io/etcher/releases/download/v2.1.4/balenaEtcher-linux-x64-2.1.4.zip
unzip balenaEtcher-linux-*.zip

wget https://www.sweetscape.com/download/010EditorLinux64Installer.tar.gz
tar xvf 010EditorLinux64Installer.tar.gz
chmod +x 010EditorLinux64Installer
./010EditorLinux64Installer  --prefix 010editor --mode silent
chmod a+x 010editor/010editor

wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb

wget https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v2.3.2/Clash.Verge_2.3.2_amd64.deb
sudo dpkg -i ./Clash.Verge_*.deb

wget https://github.com/shiftkey/desktop/releases/download/release-3.4.13-linux1/GitHubDesktop-linux-x86_64-3.4.13-linux1.AppImage
wget https://dldir1v6.qq.com/qqfile/qq/QQNT/Linux/QQ_3.2.18_250724_x86_64_01.AppImage   # https://im.qq.com/linuxqq/index.shtml
wget https://dldir1v6.qq.com/weixin/Universal/Linux/WeChatLinux_x86_64.AppImage # https://linux.weixin.qq.com/

chmod a+x *.AppImage
```

### 配置x11

```bash
export DISPLAY=:0
xhost +
# 运行xclock看效果
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
docker run hello-world
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
wget https://redirector.gvt1.com/edgedl/android/studio/ide-zips/2024.3.1.14/android-studio-2024.3.1.14-linux.tar.gz
sudo tar -xvzf android-studio-*.tar.gz -C /opt/
/opt/android-studio/bin/studio.sh
```

接下来安装，Android SDK和NDK。可以在Android Studio手动安装或者命令行安装。

```bash
mkdir -p $HOME/Android/Sdk
wget https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip -O cmdline-tools.zip
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

选择17版本的。

```
sudo update-alternatives --config java
```


### 环境变量配置

添加环境变量到 `~/.bashrc`：

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin

export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$HOME/Android/Sdk

export NDK_ROOT=$ANDROID_HOME/ndk/28.1.13356709
export ANDROID_NDK_ROOT=$ANDROID_HOME/ndk/28.1.13356709
export ANDROID_NDK=$ANDROID_HOME/ndk/28.1.13356709
export PATH=$PATH:$ANDROID_NDK_ROOT

export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
```

生效：

```
source ~/.bashrc
```


## 逆向分析工具

### 反编译和调试工具

```bash
pip3 install frida-tools --break-system-packages
```

### IDA Pro / Ghidra

- 下载并安装 **IDA Pro**（商业软件）或 **Ghidra**（开源）

- Ghidra 安装：

```bash
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.3.1_build/ghidra_11.3.1_PUBLIC_20250219.zip
unzip ghidra_11.3.1_PUBLIC_20250219.zip -d $HOME/tools
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
code ~/.gradle/init.gradle.kts

fun RepositoryHandler.enableMirror() {
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
}
```

设置代理：

```bash
code ~/.gradle/gradle.properties

# https://docs.gradle.org/current/userguide/networking.html
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=7890
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=7890
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

### 设置显示分辨率

```bash
xrandr
xrandr --output HDMI-1 --mode 1920x1080
```
