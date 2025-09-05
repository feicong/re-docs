# 安全研究员的Linux配置完全指南

文档最新更新请访问：[https://github.com/feicong/re-docs/edit/master/Linux-fedora-arm64.md](https://github.com/feicong/re-docs/edit/master/Linux-fedora-arm64.md)

## 选择合适的Linux发行版

这里选择的是asahi的Fedora 42 Linux。

```bash
uname -a
Linux fedora 6.14.2-401.asahi.fc42.aarch64+16k #1 SMP PREEMPT_DYNAMIC Wed Apr 16 12:06:15 UTC 2025 aarch64 GNU/Linux
cat /proc/version 
Linux version 6.14.2-401.asahi.fc42.aarch64+16k (mockbuild@844819db4c624c04819b557088679991) (gcc (GCC) 15.0.1 20250329 (Red Hat 15.0.1-0), GNU ld version 2.44-3.fc42) #1 SMP PREEMPT_DYNAMIC Wed Apr 16 12:06:15 UTC 2025
```

## 基础环境配置

首先配置系统的密码免输入与dnf源加速，参考：https://mirrors.tuna.tsinghua.edu.cn/help/fedora/

```bash
ARCH="$(uname -m)"

# 检查当前系统版本是否为 noble
OS_VERSION=$(lsb_release -cs)
if [ "$OS_VERSION" != "adams" ]; then
    echo "This script is designed for Fedora adams. Exiting."
    exit 1
fi

echo "$(whoami) ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$(whoami)
sudo visudo -cf /etc/sudoers.d/$(whoami)

sudo sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=http://download.example/pub/fedora/linux|baseurl=https://mirrors.tuna.tsinghua.edu.cn/fedora|g' \
    -i.bak \
    /etc/yum.repos.d/fedora.repo \
    /etc/yum.repos.d/fedora-updates.repo
sudo dnf makecache
```

### 更新系统

```bash
sudo dnf check-update
```

### 安装必备工具

```bash
sudo dnf install curl wget make upx vim openssl adb fastboot gh git git-lfs cmake python3 python3-pip \
    flex bison make tree net-tools ninja-build meson pkg-config libtool autoconf automake help2man llvm \
    grep plantuml aria2 bash bash-completion bc binutils brotli jq repo reprepro coreutils httpie lz4 \
    shared-mime-info dbus lzip ffmpeg file flac tcpdump fontconfig texinfo gawk ca-certificates gettext \
    unifdef sed gnupg gperf z3 zstd p7zip binwalk maven lsb-release patchelf gcc gdb tzdata socat \
    ltrace strace gnome-tweaks net-tools openssh-server dnsutils pahole protobuf node go just docker \
    wireshark qemu bridge-utils qemu-kvm libzip fzf htop java-21-openjdk -y
```

手动安装下面的工具：

```bash
# https://code.visualstudio.com/docs/setup/linux
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\nautorefresh=1\ntype=rpm-md\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" | sudo tee /etc/yum.repos.d/vscode.repo > /dev/null
sudo dnf update && sudo dnf install code -y

wget https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v2.4.1/Clash.Verge-2.4.1-1.aarch64.rpm
sudo dnf install ./Clash.Verge-*.rpm -y

wget https://dldir1v6.qq.com/qqfile/qq/QQNT/Linux/QQ_3.2.19_250904_aarch64_01.rpm # https://im.qq.com/linuxqq/index.shtml
sudo dnf install ./QQ_*.rpm -y

wget https://dldir1v6.qq.com/weixin/Universal/Linux/WeChatLinux_arm64.rpm # https://linux.weixin.qq.com/
sudo dnf install ./WeChatLinux_*.rpm -y
```

### 配置docker

```bash
sudo usermod -aG docker $USER
newgrp docker
sudo mkdir -p /etc/docker
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

### Java配置

选择Java版本。

```bash
sudo alternatives --config java
```

### 环境变量配置

添加环境变量到 `~/.bashrc`：

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk
export PATH=$PATH:$JAVA_HOME/bin
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
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.4.2_build/ghidra_11.4.2_PUBLIC_20250826.zip
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
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
pip install -U pip --break-system-packages
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
