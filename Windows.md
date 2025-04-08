# Windows安全研究员配置完全指南

Windows 11 Pro版本的Ser 7迷你主机为演示配置的环境。

## 操作系统配置

### 选择合适的Windows版本

推荐使用 **Windows 11 Pro 或 Enterprise** 版本，提供 Hyper-V 和组策略管理功能。我个人选择Windows 11 Pro，也就是11的专业版本。

### 系统下载

Windows 11专业版本下载：[https://www.microsoft.com/zh-cn/software-download/windows11](https://www.microsoft.com/zh-cn/software-download/windows11)

### 系统激活

Windows 11系统安装后激活（管理员权限运行PowerShell)：

```powershell
irm massgrave.dev/get.ps1 | iex
```

### 关闭不必要的系统功能

关闭 Windows Defender

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

### 关闭 UAC（用户账户控制）

```powershell
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
```

### 禁用Windows自动更新

```powershell
sc config wuauserv start= disabled
sc stop wuauserv
```

## 安全研究常用工具

首先安装软件管理器chocolatey。

```powershell
Set-ExecutionPolicy Bypass -Scope Process

Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### 常用软件

安装软件：

```powershell
choco install motrix listen1 git git-lfs wget curl aria2 putty winscp fiddler ghidra \
    python3 python311 meson which sed grep awk 7zip 7zip-zstd wireshark fastcopy googlechrome \
    vscode etcher docker-desktop winrar peazip cpu-z gpu-z tree unzip zip diskgenius adobereader \
    androidstudio openjdk openjdk17 notepadplusplus vlc nodejs nvm qemu angryip cmake dd fd xxd \
    sysinternals ninja vim openssl jq powertoys go everything checksum typescript tesseract \
    ffmpeg msys2 vnc-viewer sqlite drawio autoruns llvm directx github-desktop gnupg z3 \
    plantuml scrcpy f.lux sudo base64 zstandad apktool jadx patch gzip dos2unix \
    upx usbimager logseq snipaste gradle maven libjpeg-turbo file flac gh webp httpie \
    intellijidea-community pycharm ruby lua make kotlinc graalvm protoc pup imagemagick -y
```

更新软件：

```
choco upgrade all -y 
```

使用这个下载软件：https://neatdownload.com/

### 软件配置

输入法使用系统自带的五笔或者拼音输入法。

[ImTip](https://imtip.aardio.com/update/ImTip.7z) 使用这个工具来跟随查看输入法状态

### 安装配置wsl

```powershell
wsl --install
wsl --update
wsl --list --online
wsl --set-default Ubuntu-22.04
```

### 逆向分析工具

- **IDA Pro**（反汇编器）
- **Ghidra**（开源反编译工具）
- **Apktool**（安卓反编译工具）

### 动态调试工具

- **Frida**（动态插桩框架）

### 网络安全工具

- **Wireshark**（网络抓包工具）
- **Fiddler**（HTTP 调试代理）
- **[reqable](https://github.com/reqable/reqable-app/releases/download/2.33.8/reqable-app-windows-x86_64.zip)**（网络抓包测试工具）

## 开发环境配置

### 编程语言环境

安装常见的安全研究库：

```powershell
pip install pwntools frida frida-tools requests scapy capstone unicorn
```

### maven

```powershell
# 创建 .m2 目录（如果不存在）
$env:USERPROFILE + "\.m2" | ForEach-Object { if (-not (Test-Path $_)) { New-Item -ItemType Directory -Path $_ } }

# 写入 settings.xml 文件
@'
<?xml version="1.0" encoding="UTF-8"?> 
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
</settings>
'@ | Set-Content -Path (Join-Path $env:USERPROFILE ".m2\settings.xml")
```

### gradle

```powershell
@'
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
'@ | Set-Content -Path "C:\Users\android\.gradle\init.gradle.kts"
```

### 安装 Visual Studio

- 选择 **Desktop development with C++** 组件。
- 安装 **Windows SDK** 和 **调试工具**。

### 安装虚拟机环境

- **VMware Workstation** 或 **VirtualBox**。
- 
- 创建 **Windows 沙盒环境** 进行动态分析。
