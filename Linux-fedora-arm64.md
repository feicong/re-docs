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
    wireshark qemu bridge-utils qemu-kvm libzip fzf htop java-21-openjdk libguestfs-tools guestfs-tools \
    mesa-vulkan-drivers vulkan-loader vulkan-tools -y
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

### Android开发配置

参考：https://gist.github.com/Abyss-W4tcher/f1833623c975193446315d48c106750e

```bash
https://dl.google.com/android/repository/commandlinetools-linux-13114758_latest.zip -O commandlinetools-linux.zip
mkdir -p Android/cmdline-tools
unzip commandlinetools-linux.zip -d Android/cmdline-tools
mv Android/cmdline-tools/cmdline-tools Android/cmdline-tools/latest
export ANDROID_HOME=$(pwd)/Android
export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/cmdline-tools/latest:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin:$ANDROID_HOME/platform-tools:$PATH"

# Get emulator download link from https://ci.android.com/builds/branches/aosp-emu-master-dev/grid (needs a manual download, or get the link from your navigator downloads and paste it in the following wget parameter)
# https://ci.android.com/builds/submitted/13288691/emulator-linux_aarch64_gfxstream/latest/sdk-repo-linux_aarch64-emulator-13288691.zip
unzip -qq emulator.zip -d Android/
emulator -version | grep version
# Android emulator version 35.6.3.0 (build_id 13288691) (CL:N/A)
#  License version 2, as published by the Free Software Foundation, and

curl 'https://chromium.googlesource.com/android_tools/+/refs/heads/main/sdk/emulator/package.xml?format=TEXT' | base64 -d > Android/emulator/package.xml
# 最后一行改为如下：
# November 20, 2015</license><localPackage path="emulator" obsolete="false"><type-details xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="ns5:genericDetailsType"/><revision><major>35</major><minor>6</minor><micro>3</micro></revision><display-name>Android Emulator</display-name><uses-license ref="android-sdk-license"/><dependencies><dependency path="patcher;v4"/><dependency path="tools"><min-revision><major>25</major><minor>3</minor></min-revision></dependency></dependencies></localPackage></ns2:repository>

yes | sdkmanager --licenses
sdkmanager "build-tools;36.0.0" "platform-tools" "platforms;android-33" "tools"

# sdkmanager --list # List available packages
TARGET_IMAGE='system-images;android-33;default;arm64-v8a'
sdkmanager $TARGET_IMAGE

yes '' | avdmanager create avd -n my_avd -k $TARGET_IMAGE
emulator -avd my_avd -no-snapshot -no-window -no-audio
```

启动测试：

参考：https://www.reddit.com/r/AsahiLinux/comments/1gxzcbh/android_emulatoravd_not_working_yet_on_asahi_linux/

```bash
emulator -avd my_avd -no-snapshot -no-window -no-audio -gpu swiftshader_indirect -accel on
INFO         | Android emulator version 35.6.3.0 (build_id 13288691) (CL:N/A)
INFO         | Graphics backend: gfxstream
INFO         | Found systemPath /home/android/Downloads/Android/system-images/android-33/default/arm64-v8a/
ERROR: ld.so: object '/home/android/Downloads/Android/emulator/lib64/libStubXlib.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
ERROR: ld.so: object '/home/android/Downloads/Android/emulator/lib64/libStubXlib.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
INFO         | Increasing RAM size to 2048MB
##############################################################################
##                        WARNING - ACTION REQUIRED                         ##
##  Consider using the '-metrics-collection' flag to help improve the       ##
##  emulator by sending anonymized usage data. Or use the '-no-metrics'     ##
##  flag to bypass this warning and turn off the metrics collection.        ##
##  In a future release this warning will turn into a one-time blocking     ##
##  prompt to ask for explicit user input regarding metrics collection.     ##
##                                                                          ##
##  Please see '-help-metrics-collection' for more details. You can use     ##
##  '-metrics-to-file' or '-metrics-to-console' flags to see what type of   ##
##  data is being collected by emulator as part of usage statistics.        ##
##############################################################################
INFO         | Checking system compatibility:
INFO         |   Checking: hasSufficientDiskSpace
INFO         |      Ok: Disk space requirements to run avd: `my_avd` are met
INFO         |   Checking: hasSufficientHwGpu
INFO         |      Ok: Hardware GPU compatibility checks are not required
INFO         |   Checking: hasSufficientSystem
INFO         |      Ok: System requirements to run avd: `my_avd` are met
WARNING      | File System is not ext4, disable QuickbootFileBacked feature
INFO         | Storing crashdata in: /tmp/android-android/emu-crash-35.6.3.db, detection is enabled for process: 99818
INFO         | Guest Driver: Auto (ext controls)
library_mode swiftshader_indirect gpu mode swiftshader_indirect
INFO         | Initializing hardware OpenGLES emulation support
I0905 09:46:23.009538   99818 opengles.cpp:262] android_startOpenglesRenderer: gpu info
I0905 09:46:23.009903   99818 opengles.cpp:263] 
INFO         | Raised nofile soft limit to 4096.
INFO         | HealthMonitor disabled.
INFO         | initIcdPaths: ICD set to 'swiftshader', using Swiftshader ICD
INFO         | Setting ICD filenames for the loader = /home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/vk_swiftshader_icd.json:/home/android/Downloads/Android/emulator/lib64/vulkan/vk_swiftshader_icd.json
INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so]

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so]: not found in map, open for the first time

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so] (posix): begin

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so] (posix,linux): call dlopen on [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so]

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so] failed (posix). dlerror: []

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so.1]

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so.1]: not found in map, open for the first time

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so.1] (posix): begin

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so.1] (posix,linux): call dlopen on [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so.1]

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/qemu/linux-aarch64/lib64/vulkan/libvulkan.so.1] failed (posix). dlerror: []

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so]

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so]: not found in map, open for the first time

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so] (posix): begin

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so] (posix,linux): call dlopen on [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so]

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so] failed (posix). dlerror: []

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so.1]

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so.1]: not found in map, open for the first time

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so.1] (posix): begin

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so.1] (posix,linux): call dlopen on [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so.1]

INFO         | SharedLibrary::open for [/home/android/Downloads/Android/emulator/lib64/vulkan/libvulkan.so.1] failed (posix). dlerror: []

ERROR        | Cannot add any library for Vulkan loader from the list of 4 items
ERROR        | Dispatch is invalid.
ERROR        | Failed to initialize global Vulkan emulation. Disable the Vulkan support.
INFO         | SharedLibrary::open for [libGLESv2.so]: not found in map, open for the first time

INFO         | SharedLibrary::open for [libGLESv2.so] (posix): begin

INFO         | SharedLibrary::open for [libGLESv2.so] (posix,linux): call dlopen on [libGLESv2.so]

INFO         | SharedLibrary::open for [libEGL.so]: not found in map, open for the first time

INFO         | SharedLibrary::open for [libEGL.so] (posix): begin

INFO         | SharedLibrary::open for [libEGL.so] (posix,linux): call dlopen on [libEGL.so]

INFO         | SharedLibrary::open for [libX11]

INFO         | SharedLibrary::open for [libX11]: not found in map, open for the first time

INFO         | SharedLibrary::open for [libX11] (posix): begin

INFO         | SharedLibrary::open for [libX11] (posix,linux): call dlopen on [libX11.so]

INFO         | SharedLibrary::open for [libX11] failed (posix). dlerror: []

WARNING: could not open libX11.so, try libX11.so.6
INFO         | SharedLibrary::open for [libX11.so.6]

INFO         | SharedLibrary::open for [libX11.so.6]: not found in map, open for the first time

INFO         | SharedLibrary::open for [libX11.so.6] (posix): begin

INFO         | SharedLibrary::open for [libX11.so.6] (posix,linux): call dlopen on [libX11.so.6]

INFO         | Graphics Adapter Vendor Google (Google Inc.)
INFO         | Graphics Adapter Android Emulator OpenGL ES Translator (Google SwiftShader)
INFO         | Graphics API Version OpenGL ES 3.0 (OpenGL ES 3.0 SwiftShader 4.1.0.7)
INFO         | Graphics API Extensions GL_OES_EGL_sync GL_OES_EGL_image GL_OES_EGL_image_external GL_OES_depth24 GL_OES_depth32 GL_OES_element_index_uint GL_OES_texture_float GL_OES_texture_float_linear GL_OES_compressed_paletted_texture GL_OES_compressed_ETC1_RGB8_texture GL_OES_depth_texture GL_OES_texture_half_float GL_OES_texture_half_float_linear GL_OES_packed_depth_stencil GL_OES_vertex_half_float GL_OES_standard_derivatives GL_OES_texture_npot GL_OES_rgb8_rgba8 GL_EXT_color_buffer_float GL_EXT_color_buffer_half_float GL_EXT_texture_format_BGRA8888 GL_APPLE_texture_format_BGRA8888 
INFO         | Graphics Device Extensions N/A
INFO         | Sending adb public key [QAAAADP/EAwF7Pil3htCnXm0uW1U81sBXRxD7pvQfMc0x/i2RzinuX1U4cg3uYrqwF2daidF4AahAe0+ZGcGJjlDu1pbZq81YdS9C2jCukTkZIm+Pzlz2xvk6t1Ie94ayyF6whAKr5B+szPeUOfstlVbBD10PO4kce/u9YnixMsokDHL9Wgo84pvHzzAbk1eY4/0qSrZAcqtdx54C4L4Us6H5eb5c1YLLy+2j5PbxiT+upvnVtyJH7fZVTccc3WsORuB5CcoMyxdlDFkA2WHEYcBnvd0qd41uytrckpHVQrSresJGWQTO75RCE2qL7rwhVjJRdjsVq/r+XRfMJudBL4eeuMx6tWsdhqExON6goYudgBXgc6IjgAm3HvjlwKxKpcxdIZeJaDe9Q8foWx3dkKxDrr6HVUG5srXW3mkQWj/fwwQjv96Vy+0npGzR46VqX2dTzzGBPAilBQPdaV2PUBNaXJ6nMklS+AY2dEpKd5dZYjymaoB4egSgvs47ZQx/cuMeGUSTBrG0Vp+c7jC++AcYni9MhC/3eIWzmHikIGVNF7sc03zNd62n4Uf3MXA/sp3ye0IEgWGZMUEobYNqNr/B2sYormKLMW44kaLF2PiP+mdpIuXPvcTF+Y9Z37nmn49SyqubB4bqy9IlzDGYxGOJ7v4uihSDewE4sd9qRauMwVvEgKCpAEAAQA= android@fedora]
INFO         | Userspace boot properties:
INFO         |   androidboot.boot_devices=a003600.virtio_mmio
INFO         |   androidboot.dalvik.vm.heapsize=512m
INFO         |   androidboot.debug.hwui.renderer=skiagl
INFO         |   androidboot.hardware=ranchu
INFO         |   androidboot.hardware.gltransport=pipe
INFO         |   androidboot.hardware.vulkan=ranchu
INFO         |   androidboot.logcat=*:V
INFO         |   androidboot.opengles.version=196608
INFO         |   androidboot.qemu=1
INFO         |   androidboot.qemu.adb.pubkey=QAAAADP/EAwF7Pil3htCnXm0uW1U81sBXRxD7pvQfMc0x/i2RzinuX1U4cg3uYrqwF2daidF4AahAe0+ZGcGJjlDu1pbZq81YdS9C2jCukTkZIm+Pzlz2xvk6t1Ie94ayyF6whAKr5B+szPeUOfstlVbBD10PO4kce/u9YnixMsokDHL9Wgo84pvHzzAbk1eY4/0qSrZAcqtdx54C4L4Us6H5eb5c1YLLy+2j5PbxiT+upvnVtyJH7fZVTccc3WsORuB5CcoMyxdlDFkA2WHEYcBnvd0qd41uytrckpHVQrSresJGWQTO75RCE2qL7rwhVjJRdjsVq/r+XRfMJudBL4eeuMx6tWsdhqExON6goYudgBXgc6IjgAm3HvjlwKxKpcxdIZeJaDe9Q8foWx3dkKxDrr6HVUG5srXW3mkQWj/fwwQjv96Vy+0npGzR46VqX2dTzzGBPAilBQPdaV2PUBNaXJ6nMklS+AY2dEpKd5dZYjymaoB4egSgvs47ZQx/cuMeGUSTBrG0Vp+c7jC++AcYni9MhC/3eIWzmHikIGVNF7sc03zNd62n4Uf3MXA/sp3ye0IEgWGZMUEobYNqNr/B2sYormKLMW44kaLF2PiP+mdpIuXPvcTF+Y9Z37nmn49SyqubB4bqy9IlzDGYxGOJ7v4uihSDewE4sd9qRauMwVvEgKCpAEAAQA= android@fedora
INFO         |   androidboot.qemu.avd_name=my_avd
INFO         |   androidboot.qemu.camera_hq_edge_processing=0
INFO         |   androidboot.qemu.camera_protocol_ver=1
INFO         |   androidboot.qemu.cpuvulkan.version=4202496
INFO         |   androidboot.qemu.gltransport.drawFlushInterval=800
INFO         |   androidboot.qemu.gltransport.name=pipe
INFO         |   androidboot.qemu.hwcodec.avcdec=2
INFO         |   androidboot.qemu.hwcodec.hevcdec=2
INFO         |   androidboot.qemu.hwcodec.vpxdec=2
INFO         |   androidboot.qemu.settings.system.screen_off_timeout=2147483647
INFO         |   androidboot.qemu.virtiowifi=1
INFO         |   androidboot.qemu.vsync=60
INFO         |   androidboot.serialno=EMULATOR35X6X3X0
INFO         |   androidboot.vbmeta.digest=0aa6a7fa9cc8ff3e3ee12c25c0c91e7dbf3627f0d45dc44a48cbd8d88eb72f59
INFO         |   androidboot.vbmeta.hash_alg=sha256
INFO         |   androidboot.vbmeta.size=6208
INFO         |   androidboot.veritymode=enforcing
ioctl(KVM_CREATE_VM) failed: 22 Invalid argument
qemu-system-aarch64-headless: failed to initialize KVM: Invalid argument
WARNING      | QEMU main loop exits abnormally with code 1
INFO         | Wait for emulator (pid 99818) 10 seconds to shutdown gracefully before kill;you can set environment variable ANDROID_EMULATOR_WAIT_TIME_BEFORE_KILL(in seconds) to change the default value (20 seconds)

ERROR        | Unable to spawn process  due to:, No such file or directory
INFO         | Wait for emulator (pid 99818) 20 seconds to shutdown gracefully before kill;you can set environment variable ANDROID_EMULATOR_WAIT_TIME_BEFORE_KILL(in seconds) to change the default value (20 seconds)

sudo dmesg | grep -i kvm
[    0.062036] kvm [1]: nv: 566 coarse grained trap handlers
[    0.062081] kvm [1]: IPA Size Limit: 36 bits (Reduced IPA size, limited VM/VMM compatibility)
[    0.062089] kvm [1]: Non-architectural vgic, tainting kernel
[    0.062090] kvm [1]: GICv3: no GICV resource entry
[    0.062091] kvm [1]: disabling GICv2 emulation
[    0.062092] kvm [1]: GICv3 with broken locally generated SEI
[    0.062092] kvm [1]: GICv3 sysreg trapping enabled ([G0G1D], reduced performance)
[    0.062117] kvm [1]: GIC system register CPU interface enabled
[    0.062123] kvm [1]: vgic interrupt IRQ33
[    0.062142] kvm [1]: VHE mode initialized successfully

qemu-system-aarch64 -accel help
Accelerators supported in QEMU binary:
xen
kvm
tcg

# 卒！ 参考：https://gitlab.com/libvirt/libvirt/-/issues/365
```

### Ubuntu虚拟化

配置：

```bash
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-arm64.img
qemu-img resize noble-server-cloudimg-arm64.img 120G
qemu-img create -f qcow2 noble-server-cloudimg-arm64-120G.img 120G
virt-resize --expand /dev/sda1 noble-server-cloudimg-arm64.img noble-server-cloudimg-arm64-120G.img

virt-customize -a noble-server-cloudimg-arm64-120G.img --root-password password:android
virt-customize -a noble-server-cloudimg-arm64-120G.img \
    --run-command "useradd -m -s /bin/bash android" \
  --run-command "echo 'android:android' | chpasswd" \
  --run-command "usermod -aG sudo android"

```

启动：

参考： https://gist.github.com/itzurabhi/a760155c28c0e34ebb14ccf10f08d47b

```bash
sudo modprobe virtio_net
qemu-system-aarch64 -accel kvm  -m 4500M -cpu max -smp 4 -M virt -nographic -pflash flash1.img -drive if=none,file=noble-server-cloudimg-arm64-120G.img,format=qcow2,id=hd0 -device virtio-blk-device,drive=hd0 -netdev user,id=net0,hostfwd=tcp::5555-:22 -device virtio-net-device,netdev=net0,mac=00:16:3e:68:02:5d
...
android@ sudo chmod 0600 /etc/netplan/00-installer-config.yaml
android$ cat /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
sudo netplan apply
```
