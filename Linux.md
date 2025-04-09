# Linux安全研究员配置完全指南

## 选择合适的Linux发行版

推荐使用**Ubuntu 24.04 LTS**，主要这是安卓系统与软件开发使用的官方Linux发行版本。

官方下载地址：[https://releases.ubuntu.com/24.04/](https://releases.ubuntu.com/24.04/)

下载链接直达：[https://releases.ubuntu.com/24.04/ubuntu-24.04.2-desktop-amd64.iso](https://releases.ubuntu.com/24.04/ubuntu-24.04.2-desktop-amd64.iso)

国内mirror下载链接列表：

[https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.2-desktop-amd64.iso](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.2-desktop-amd64.iso)

[https://mirrors.ustc.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.2-desktop-amd64.iso](https://mirrors.ustc.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.2-desktop-amd64.iso)

[https://mirrors.aliyun.com/ubuntu-releases/24.04.2/ubuntu-24.04.2-desktop-amd64.iso](https://mirrors.aliyun.com/ubuntu-releases/24.04.2/ubuntu-24.04.2-desktop-amd64.iso)

[https://mirrors.sdu.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.2-desktop-amd64.iso](https://mirrors.sdu.edu.cn/ubuntu-releases/24.04/ubuntu-24.04.2-desktop-amd64.iso)

## 基础环境配置

### 更新系统

```bash
sudo apt update && sudo apt upgrade -y
sudo add-apt-repository ppa:wireshark-dev/stable -y
```

### 安装必备工具

```bash
sudo apt update && sudo apt install -y snapd git git-lfs curl wget unzip zip build-essential cmake \
    python3 python3-pip libpython3-dev openjdk-17-jdk upx wireshark flex bison make tree net-tools \
    ninja-build meson pkg-config libtool autoconf automake help2man llvm lua5.4 ruby valac kotlin vim \
    graphviz grep plantuml aria2 bash bash-completion bc binutils imagemagick brotli qemu-user qemu-system \
    jq repo reprepro quickjs coreutils libarchive-dev scrcpy httpie lz4 shared-mime-info dbus lzip \
    android-sdk-libsparse-utils libelf-dev libevent-dev ffmpeg libffi-dev file flac mpg123 tcpdump flex \
    fontconfig libncurses-dev libfreetype-dev texinfo libusbmuxd-dev gawk ca-certificates libmagic-dev \
    gettext unifdef libnghttp2-dev libnghttp3-dev libnice-dev webp libpcap-dev sed libplist-dev gnupg \
    libpng-dev openssl x264 x265 gobject-introspection googletest gperf z3 libslirp-dev zstd p7zip binwalk \
    libcapstone-dev apktool jadx smali gradle maven
```

一些gui工具

```bash
sudo snap install intellij-idea-community --classic
sudo snap install pycharm-community --classic
sudo snap install chromium --classic
sudo snap install gh --classic
sudo snap install go --classic
sudo snap install node --classic
sudo snap install code --classic
sudo snap install --edge bytecode-viewer
sudo snap install ghidra
sudo snap install tesseract
sudo snap install protobuf --classic
```

```bash
wget https://github.com/balena-io/etcher/releases/download/v2.1.0/balenaEtcher-2.1.0-x64.AppImage
chmod a+x balenaEtcher-2.1.0-x64.AppImage

wget https://www.sweetscape.com/download/010EditorLinux64Installer.tar.gz
tar xvf 010EditorLinux64Installer.tar.gz
chmod +x 010EditorLinux64Installer
./010EditorLinux64Installer  --prefix 010editor --mode silent
chmod a+x 010editor/010editor

wget https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v2.2.2/Clash.Verge_2.2.2_amd64.deb
sudo apt install ./Clash.Verge_*.deb


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
mkdir -p $HOME/Android/sdk
wget https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip -O cmdline-tools.zip
unzip cmdline-tools.zip -d $HOME/Android/sdk
mv $HOME/Android/sdk/cmdline-tools $HOME/Android/sdk/cmdline-tools-latest
mkdir -p $HOME/Android/sdk/cmdline-tools
mv $HOME/Android/sdk/cmdline-tools-latest $HOME/Android/sdk/cmdline-tools/latest
```

添加环境变量到 `~/.bashrc`：

```bash
export ANDROID_HOME=$HOME/Android/sdk
export PATH=$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$PATH

source ~/.bashrc
sdkmanager --install "platform-tools" "platforms;android-33" "build-tools;33.0.2"
```

### 自动配置安装

```bash
sudo snap install android-studio --classic
```


## 逆向分析工具

### 反编译和调试工具

```bash
pip3 install frida-tools
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

### golang

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

### pip

设置pip的mirror。

```bash
export HOMEBREW_PIP_INDEX_URL="https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
python -m pip install --upgrade pip
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

### npm

设置npm的mirror。

```bash
npm config set registry https://registry.npmmirror.com
```

安装一些npm工具。

```bash
npm install -g typescript
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
