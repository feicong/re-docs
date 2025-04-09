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
```

### 安装必备工具

```bash
sudo apt install -y git curl wget unzip zip build-essential cmake python3 python3-pip openjdk-17-jdk upx
```

### 设置Python环境

```bash
pip3 install --upgrade pip
```

## Android 开发环境配置

### 安装Android Studio

下载地址：[https://developer.android.com/?hl=zh-cn](https://developer.android.com/?hl=zh-cn)

```bash
wget https://xxx/android-studio-xxx-linux.tar.gz
sudo tar -xvzf android-studio-*.tar.gz -C /opt/
/opt/android-studio/bin/studio.sh
```

### 安装 Android SDK和NDK

可以在Android Studio手动安装或者命令行安装。

```bash
mkdir -p $HOME/Android/sdk
wget https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip -O cmdline-tools.zip
unzip cmdline-tools.zip -d $HOME/Android/sdk
mv $HOME/Android/sdk/cmdline-tools $HOME/Android/sdk/cmdline-tools-latest
mkdir -p $HOME/Android/sdk/cmdline-tools
mv $HOME/Android/sdk/cmdline-tools-latest $HOME/Android/sdk/cmdline-tools/latest
```

添加环境变量到 `~/.bashrc` 或 `~/.zshrc`：

```bash
export ANDROID_HOME=$HOME/Android/sdk
export PATH=$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$PATH
```

应用环境变量：

```bash
source ~/.bashrc
```

安装SDK组件：

```bash
sdkmanager --install "platform-tools" "platforms;android-33" "build-tools;33.0.2"
```

### 3.2 安装adb和fastboot

```bash
sudo apt install -y adb fastboot
```

## 逆向分析工具

### 反编译和调试工具

```bash
sudo apt install -y apktool jadx
pip3 install frida-tools
```

### IDA Pro / Ghidra

- 下载并安装 **IDA Pro**（商业软件）或 **Ghidra**（开源）

- Ghidra 安装：

```bash
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.3.1_build/ghidra_11.3.1_PUBLIC_20250219.zip
unzip ghidra_11.3.1_PUBLIC_20250219.zip -d $HOME/tools
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
