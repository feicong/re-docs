# 安全研究员的Windows配置完全指南

文档最新更新请访问：[https://github.com/feicong/re-docs/edit/master/Windows.md](https://github.com/feicong/re-docs/edit/master/Windows.md)

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

### 关闭Windows Defender

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

### 启动系统保护

```bash
Enable-ComputerRestore -Drive "C:\"
```

## 安全研究常用工具

Windows系统在安装其它软件前，先配置一个Clash方便代理下载与更新软件.

我是建议使用这个：https://github.com/clash-verge-rev/clash-verge-rev/releases

机场配置大家自己想办法，一般是购买或者有服务器的自建。

[Clash.Verge](https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v2.2.2/Clash.Verge_2.2.2_x64-setup.exe)

接下来，安装软件管理器chocolatey。

```powershell
Set-ExecutionPolicy Bypass -Scope Process

Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### 常用软件

安装软件：

```powershell
choco install motrix listen1 git git-lfs wget curl aria2 putty winscp fiddler postman ghidra `
    python3 python311 meson which sed grep awk 7zip 7zip-zstd wireshark fastcopy googlechrome `
    vscode etcher docker-desktop winrar peazip cpu-z gpu-z tree unzip zip diskgenius adobereader `
    androidstudio openjdk openjdk17 oracle17jdk notepadplusplus vlc nodejs nvm qemu angryip cmake `
    sysinternals ninja vim openssl jq powertoys go everything checksum typescript tesseract `
    ffmpeg msys2 vnc-viewer sqlite drawio autoruns llvm directx github-desktop gnupg z3 `
    plantuml scrcpy f.lux sudo base64 zstandard apktool jadx patch gzip dos2unix dd fd xxd `
    upx usbimager logseq snipaste gradle maven libjpeg-turbo file flac gh webp httpie imagemagick `
    intellijidea-community pycharm clion-ide ruby lua make kotlinc protoc pup cursoride claude -y
```

> Wireshark需要单独安装一下npcap： https://npcap.com/dist/npcap-1.81.exe

更新软件：

```
choco upgrade all -y 
```

使用这个下载软件：https://neatdownload.com/

###环境变量配置

使用setx自动配置部分环境变量。

```powershell
setx ANDROID_HOME "C:\Users\android\AppData\Local\Android\Sdk"
setx ANDROID_NDK "%ANDROID_HOME%\ndk\28.0.12916984"
setx JAVA_HOME "C:\Program Files\Java\jdk-17.0.2"
```

下面的手动加到Path中去，如果使用setx设置错误，后果非常严重！

```powershell
%ANDROID_HOME%\build-tools\36.0.0
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\emulator
%ANDROID_HOME%\cmdline-tools\latest\bin
%JAVA_HOME%\bin
```

### 软件配置

输入法使用系统自带的五笔或者拼音输入法。

[ImTip](https://imtip.aardio.com/update/ImTip.7z) 使用这个工具来跟随查看输入法状态

### 安装配置WSL

安装：

```powershell
wsl --install
wsl --update
wsl --list --online
wsl --set-default Ubuntu-22.04
```

WSL配置代理：

```powershell
cat C:\Users\android\.wslconfig

[experimental]
autoMemoryReclaim=gradual
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
```

### Docker配置

见[macOS版本的Docker配置](https://github.com/feicong/re-docs/blob/master/macOS.md#orbstack%E9%85%8D%E7%BD%AE)


### 动态调试工具

- **Frida**（动态插桩框架）

### 网络安全工具

- **Wireshark**（网络抓包工具）
  
- **Fiddler**（HTTP 调试代理）
  
- **[reqable](https://github.com/reqable/reqable-app/releases/download/2.33.8/reqable-app-windows-x86_64.zip)**（网络抓包测试工具）

## 开发环境配置

一些软件需要配置登陆与设置代理。

### vscode配置

```powershell
$extensions = @(
  "redhat.java",
  "vscjava.vscode-gradle",
  "vscjava.vscode-java-pack",
  "vscjava.vscode-java-debug",
  "adelphes.android-dev-ext",
  "OmarDulaimi.breakpoint-bookmarks",
  "ms-dotnettools.csharp",
  "bito.bito",
  "bpfdeploy.bpftrace",
  "codezombiech.gitignore",
  "cornell3110sp20.rml-highlighter",
  "dbaeumer.vscode-eslint",
  "eriklynd.json-tools",
  "formulahendry.code-runner",
  "foxundermoon.shell-format",
  "genieai.chatgpt-vscode",
  "github.codespaces",
  "github.copilot",
  "github.copilot-chat",
  "github.github-vscode-theme",
  "github.remotehub",
  "github.vscode-github-actions",
  "github.vscode-pull-request-github",
  "golang.go",
  "google.aidl-language",
  "gruntfuggly.todo-tree",
  "jebbs.plantuml",
  "josetr.cmake-language-support-vscode",
  "jrieken.md-navigate",
  "mesonbuild.mesonbuild",
  "ms-azuretools.vscode-docker",
  "ms-ceintl.vscode-language-pack-zh-hans",
  "ms-dotnettools.vscode-dotnet-runtime",
  "ms-python.autopep8",
  "ms-python.debugpy",
  "ms-python.isort",
  "ms-python.python",
  "ms-python.vscode-pylance",
  "ms-vscode-remote.remote-containers",
  "ms-vscode-remote.remote-ssh",
  "ms-vscode-remote.remote-ssh-edit",
  "ms-vscode-remote.remote-wsl",
  "ms-vscode-remote.vscode-remote-extensionpack",
  "ms-vscode.cmake-tools",
  "ms-vscode.cpptools",
  "ms-vscode.cpptools-extension-pack",
  "ms-vscode.cpptools-themes",
  "ms-vscode.hexeditor",
  "ms-vscode.makefile-tools",
  "ms-vscode.powershell",
  "ms-vscode.remote-explorer",
  "ms-vscode.remote-repositories",
  "ms-vscode.remote-server",
  "ms-vscode.vscode-typescript-next",
  "osstekz.vala-code",
  "prince781.vala",
  "redhat.vscode-commons",
  "redhat.vscode-xml",
  "redhat.vscode-yaml",
  "rogalmic.bash-debug",
  "twxs.cmake",
  "vadimcn.vscode-lldb",
  "vscjava.vscode-maven",
  "vscode-icons-team.vscode-icons",
  "xaver.clang-format"
  "shd101wyy.markdown-preview-enhanced"
  "saoudrizwan.claude-dev"
  "Continue.continue"
)

foreach ($extension in $extensions) {
  Write-Host "正在安装扩展: $extension"
  code --install-extension $extension
}
```

### gh

这个是github官方的命令行工具，管理仓库贼方便。登陆后就可以使用了。

```powershell
gh auth login
```

### golang

```powershell
go env -w GOPROXY=https://goproxy.cn,direct
```

### pip

设置pip的mirror。

```powershell
python.exe -m pip install --upgrade pip
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

安装常见的安全研究库：

```powershell
pip install pwntools frida frida-tools requests scapy capstone unicorn
```

### npm

设置npm的mirror。

```powershell
npm.cmd config set registry https://registry.npmmirror.com
```

安装一些npm工具。

```powershell
npm.cmd install -g frida
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
            <name>aliyun</name>
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
            <url>https://mirrors.163.com/maven/repository/maven-public/</url>
        </mirror>

        <mirror>
            <id>nexus-tencentyun</id>
            <mirrorOf>*</mirrorOf>
            <name>Nexus tencentyun</name>
            <url>https://mirrors.cloud.tencent.com/nexus/repository/maven-public/</url>
        </mirror>
    </mirrors>
</settings>
'@ | Set-Content -Path (Join-Path $env:USERPROFILE ".m2\settings.xml")
```

### gradle

设置依赖mirror：

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

设置代理：

```powershell
$gradleDir = Join-Path $env:USERPROFILE ".gradle"
if (-not (Test-Path $gradleDir)) {
    New-Item -ItemType Directory -Path $gradleDir
}

# 写入 gradle.properties 文件
@'
# https://docs.gradle.org/current/userguide/networking.html
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=7890
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=7890
'@ | Set-Content -Path (Join-Path $gradleDir "gradle.properties")
```


### 安装 Visual Studio

- 选择 **Desktop development with C++** 组件。
- 安装 **Windows SDK** 和 **调试工具**。

### 安装虚拟机环境

- **VMware Workstation** 或 **VirtualBox**。
- 
- 创建 **Windows 沙盒环境** 进行动态分析。
