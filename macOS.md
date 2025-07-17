# 安全研究员的macOS配置完全指南

文档最新更新请访问：[https://github.com/feicong/re-docs/edit/master/macOS.md](https://github.com/feicong/re-docs/edit/master/macOS.md)

Apple M4 Pro为演示配置的环境，收到机器后，开机然后开始设置。

## 环境准备

首先，打开`App Store`下载安装Xcode。

启动`Xcode`，安装好命令行工具。或者终端执行```xcode-select --install```也行。

终端执行```sudo spctl --master-disable```来开启第三方或ad-hoc签名的程序运行。在macOS15以上，执行这条命令后，过需要在设置->隐私与安全性->安全性中，改为任何来源。

接下来，安装`HomeBrew`。下载地址需要代理一下，执行如下命令安装：

```bash
/bin/bash -c "$(curl -fsSL https://github.com/Homebrew/install/raw/master/install.sh)"
```

下载软件可能有时候不能成功，执行如下命令设置一下代理。

```bash
echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"' >> ~/.zprofile
echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"' >> ~/.zprofile
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
```

## 常用的命令行工具与库

下面这些工具与库建议都安装一下。


### 编程语言与开发工具

```bash
brew install pkg-config poetry gtk+3 gtk4 libtool autoconf automake help2man pygobject3 python3 python@3.11 qt \
    llvm llvm@18 ruby lua make meson ninja node npm gcc nvm vala vala-language-server git git-lfs openjdk openjdk@17 \
    kotlin kotlin-native go perl gradle maven pipx vim cmake oracle-jdk oracle-jdk@17
```


### 工具与库

```bash
brew install graphviz libsoup grep libsoup@2 plantuml gstreamer libssh2 aria2 axel libtiff protobuf psutils libunibreak pup libusb \
    pycparser bash libusbmuxd bash-completion libusrsctp bc binutils imagemagick bison libx11 brotli qemu u-boot-tools jpeg-turbo \
    jq json-glib readline jsoncpp jsonrpc-glib libyaml repo reprepro colima quickjs coreutils cryptography libarchive scrcpy curl grpcurl \
    sdl2 lz4 shared-mime-info dbus lzip simg2img dbus-glib docker-completion dtc dwarf mbedtls dwarfutils libedit sqlite libelf \
    libevent mitmproxy ffmpeg libffi file-formula libgcrypt flac libgee mpg123 tcpdump flex tesseract fontconfig ncurses tesseract-lang \
    freetype texinfo libimobiledevice gawk tree ca-certificates gdbm libmagic nspr ucl gettext gh unifdef libnghttp2 unzip libnghttp3 \
    libnice glib glib-networking webp libpcap openjpeg wget gnu-sed libplist gnupg libpng openssl openssl@1.1 gnutls libpsl x265 x264 \
    gobject-introspection xz googletest pcre2 gperf z3 libslirp zip libsodium zstd p7zip jtool2 ios-deploy binwalk upx XQuartz \
    apktool jadx capstone neofetch mpv windsurf websocat
```

### 配置x11

打开XQuartz，在设置的安全性中打开允许连接。关闭后，重新打开它。

```bash
echo "export DISPLAY=:0" >> ~/.zshrc
source ~/.zshrc

export DISPLAY=:0
xhost +
# 运行xclock看效果
```

### 配置环境变量

修改：~/.bash_profile
或者修改：~/.zprofile

```bash
export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"

export LANG="en_US.UTF-8"
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
export LC_COLLATE="en_US.UTF-8"

export PATH="/opt/homebrew/opt/llvm/bin:~/Library/Python/3.9/bin:~/go/bin:$PATH"

export ANDROID_HOME=~/Library/Android/sdk
export ANDROID_SDK_ROOT=~/Library/Android/sdk

export ANDROID_NDK_ROOT=$ANDROID_HOME/ndk/25.2.9519653
export ANDROID_NDK=$ANDROID_HOME/ndk/25.2.9519653

export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
export JAVA_HOME=/Library/Java/JavaVirtualMachines/openjdk-17.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"
export PATH="$PATH:/Applications/010 Editor.app/Contents/CmdLine:/opt/homebrew/bin"
```

### 软件功能说明

`graphviz`与`plantuml`画图必备。

`apktool`与`smali`反编译APK要用到。

`aria2`与`wget`、`curl`下载软件用到。

`yt-dlp`与`youtube-dl`是下载youtube视频的命令行工具。

`llvm`与`gcc`编译器套件用于软件开发。

`lima`与`colima`采用命令行方法管理虚拟机。

`python`、`node`等是运行大量第三方软件的基础环境。

`scrcpy`用于安卓设备投屏。

`tesseract`用于OCR识别。

`ffmpeg`用于视频编解与转码，这一个就够了。

其中，还有大量的库是这些软件用到的依赖，还有一些是开发`vala`程序用到的，这里不一一介绍了。

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
python3 -m pip install --upgrade pip
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

### npm

设置npm的mirror。

```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
npm config set registry https://registry.npmmirror.com
```

安装一些npm工具。

```bash
npm install -g @anthropic-ai/claude-code typescript
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
            <url>https://mirrors.163.com/maven/repository/maven-public/</url>
        </mirror>

        <mirror>
            <id>nexus-tencentyun</id>
            <mirrorOf>*</mirrorOf>
            <name>Nexus tencentyun</name>
            <url>https://mirrors.cloud.tencent.com/nexus/repository/maven-public/</url>
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

## 常用的GUI工具

首先是常用的IDE，根据个人开发需要安装。```brew install android-studio intellij-idea pycharm clion goland```。

JB系列有免费社区版本的，专业版本收费可以弄开源项目申请免费使用，可以用一年。也可以其它渠道购买或和谐。有条件建议购买，生产力工具真的好用。

### 通过brew安装

```bash
brew install --cask wireshark charles reqable postman httpie 1password disk-drill windows-app displays \
    flux utm balenaetcher phantomjs vienna beyond-compare visual-studio-code github chatgpt raycast \
    ghidra vnc-viewer hiddenbar crescendo iina cryptomator itraffic iterm2 tabby termius orbstack obs \
    microsoft-office google-chrome xmind localsend 010-editor wechat qq bilibili showyedge angry-ip-scanner \
    battery microsoft-auto-update viz pearcleaner pdf-expert hex-fiend cherry-studio claude cursor lulu \
    ios-app-signer motrix listen1 bit-slicer clash-verge-rev qingg logseq Snipaste licecap drawio syncthing \
    db-browser-for-sqlite macs-fan-control keyboardcleantool karabiner-elements pika gemini-cli

brew install alienator88/homebrew-cask/sentinel-app
```

`wireshark`，`charles`还有`reqable`是抓包必备的。

`1password`是最好用的密码管理软件。

`disk-drill`是不错的磁盘管理工具。

<img width="1174" alt="image" src="https://github.com/user-attachments/assets/fa02c064-3630-4c50-908a-bffc150007e8" />

`windows-app`是微软件出品的3389远程连接软件，免费好用，基本杀死了苹果官方出品的同类收费软件。

<img width="1168" alt="image" src="https://github.com/user-attachments/assets/cf870f0b-f4e0-4a29-8559-8936848bef47" />

`displays`是桌面分辨率管理软件。

`orbstack`是Docker与虚拟机一体的免费软件，必装。

<img width="967" alt="image" src="https://github.com/user-attachments/assets/22b7ff6a-aa58-4489-a8b9-8d9e053d3022" />

`obs`是开源的录屏与推流软件，最好用，没有之一。

<img width="1091" alt="image" src="https://github.com/user-attachments/assets/39608db7-c608-41bb-9aec-0c05954bb8ae" />

`usbimager`、`balenaetcher`、`raspberry-pi-imager`这三个是烧硬盘镜像的工具，我一般用中间这个。

<img width="794" alt="image" src="https://github.com/user-attachments/assets/7d26dab8-0f9b-41e2-8c1b-6c02390d78a8" />

`flux`是护眼神器，自动根据时间与时区调整显示器亮度与色温。

<img width="436" alt="image" src="https://github.com/user-attachments/assets/44ecb591-087c-4d82-98e7-64c4167d9fa1" />

`vienna`是免费的RSS订阅器，我每天用它来看博客与技术仓库更新。

<img width="966" alt="image" src="https://github.com/user-attachments/assets/eb032c75-178c-424f-822c-7d695031ac61" />

`beyond-compare`与`010-editor`是文件编辑与比较的利器。

`visual-studio-code`是每天必用的编辑器，也可以说是IDE了，微软出品的全球开发人员都喜爱的工具。

`ghidra`是开源的反编译工具，用于替换商业软件`IDA Pro`，但目前无法动摇其地位。

`github`是官方的仓库代码管理工具，也怪好用的。`vnc-viewer`是VNC远程连接工具，有总比没有强。

<img width="961" alt="image" src="https://github.com/user-attachments/assets/471c4ce1-ad0e-4eae-8eed-0f38e889c7db" />

`hiddenbar`是用于管理状态栏的图标们的显示与隐藏，免费的用起来也还可以。

<img width="670" alt="image" src="https://github.com/user-attachments/assets/a42e9feb-98ea-47da-a296-759a528c79f9" />

`crescendo`是一个内核扩展，用于行为与网络分析，安全分析人员利器。

<img width="1402" alt="image" src="https://github.com/user-attachments/assets/efb99b4b-93b1-421e-9a40-2713cd89ddca" />

`iina`是开源的最好用的视频播放器，没有之一。

<img width="1622" alt="image" src="https://github.com/user-attachments/assets/3cda34d2-4c9b-49ff-b7aa-b3322b3eb60c" />

`cryptomator`是一个加密工具，管理小秘密必备。

<img width="651" alt="image" src="https://github.com/user-attachments/assets/452ba405-a926-4456-b360-c01991acc14f" />

`itraffic`看本机进程的流量，抓偸传流量的软件，恶意上传一目了然。

<img width="347" alt="image" src="https://github.com/user-attachments/assets/1ade7f94-dce5-4bf4-ac7c-746032345121" />

`tabby`与`termius`是一个开源免费一个收费的终端工具，也都不错，还有个免费的`iterm2`我也很喜欢。

<img width="1335" alt="image" src="https://github.com/user-attachments/assets/9bd9c284-9b26-47a2-b3e1-1134ba7de71c" />

`microsoft-office`是办公套件，微软出品，全球人都在用。

<img width="1374" alt="image" src="https://github.com/user-attachments/assets/11bc4fb5-b471-4f48-ad3f-2df0b9f273e6" />

`google-chrome`是谷歌的chrome浏览器，也是天天要用的。

`xmind`画图工具算是国光了。

<img width="963" alt="image" src="https://github.com/user-attachments/assets/d8667ac0-78d1-4f4b-961c-1b10fab86462" />

`localsend`跨平台传文件，我全靠它。

<img width="903" alt="image" src="https://github.com/user-attachments/assets/63d49817-93be-41bc-9930-4bb25dde179a" />

`wechat`与`qq`不用说。

`telegram`是全球最好用的IM。

`bilibili`是二次元的天堂。

`showyedge`对于来说必不可少，它把输入法不同的状态在状态栏用不同颜色的一条线显示，让我知道当前用的是什么输入法，不用梗着脖子看右上角状态栏。

<img width="895" alt="image" src="https://github.com/user-attachments/assets/fff7add0-7c52-4d73-8192-63ea837459a8" />

`angry-ip-scanner`用于扫描本地网络IP与设备信息，设备多的网络环境必不可少。

<img width="1298" alt="image" src="https://github.com/user-attachments/assets/927f8c4a-2a52-403c-bbeb-f5718a755d3a" />

`snip`免费的截图工具，好用。

`sentinel`对于苹果用户也是必不可少的，经常下载的第三方工具没签名或者ADHOC签名，提示损坏删除，用这个工具抹抹文件的附加信息就可以了。

<img width="547" alt="image" src="https://github.com/user-attachments/assets/def09fa0-8451-42ea-a2bc-4fd25f4d7e53" />

`battery`是免费的电池管理软件，苹果的电池老金贵了，用它观察准没错。

`viz`是开源的截图与屏幕二维码扫描工具，比`snip`还好用的样子。

<img width="333" alt="image" src="https://github.com/user-attachments/assets/7ea7f432-100c-4077-a447-c368f15c6962" />

`pearcleaner`是开源的软件卸载管理软件。

<img width="896" alt="image" src="https://github.com/user-attachments/assets/001c602a-2622-4fae-a602-cdd9678d461c" />

`pdf-expert`是看PDF必备的。

<img width="1510" alt="image" src="https://github.com/user-attachments/assets/13c6263c-3845-479d-8670-5778a5b1932c" />

`hex-fiend`是免费的十六进制编辑工具。

<img width="382" alt="image" src="https://github.com/user-attachments/assets/13ab4414-7f25-4e0e-9e21-5ea9e301bebd" />

`utm`与`vmware-fusion`都是个人免费使用的虚拟机软件，但我现在用`orbstack`了。

<img width="1090" alt="image" src="https://github.com/user-attachments/assets/c9cc9623-b001-4e92-a70e-e9b196a02729" />

`raycast`用于替代官方的聚焦搜索，怪好用的。

`reqable`是跨平台的网络抓包与API测试工具。

`ios-app-signer`是IPA签名工具，只是自己买证书，真的贵。

<img width="543" alt="image" src="https://github.com/user-attachments/assets/3fca946b-c8da-4940-a804-965477a73d34" />

`motrix`是开源的下载软件，我这里就不推荐迅雷了。

<img width="1028" alt="image" src="https://github.com/user-attachments/assets/1dc1a27a-1552-4c30-8fab-f118b95a145c" />

`listen1`是免费的跨平台音乐播放器的，搜歌与听歌于一体，我就不推荐国内的音乐播放器了。

<img width="983" alt="image" src="https://github.com/user-attachments/assets/4cab9436-bcfb-40a9-8315-b3f6b4c2bb27" />

`bit-slicer`是搜内存的工具，懂的都懂。

<img width="779" alt="image" src="https://github.com/user-attachments/assets/1c0d70ad-57c7-4d7b-803a-eafe29128a7e" />

`clash-verge-rev`是`clash-for-windows`的继任者。

<img width="884" alt="image" src="https://github.com/user-attachments/assets/89d0ebea-5fa1-45ad-92df-8623aabbf441" />

`qingg`清歌是最好用的免费输入法了。

<img width="358" alt="image" src="https://github.com/user-attachments/assets/27ae5d46-d9bd-44c6-8bcf-216a181ddf0d" />

`logseq`是一款开源的笔记管理软件，支持markdown，好用，是typora的平替。

<img width="981" alt="image" src="https://github.com/user-attachments/assets/3b00b7ff-6e56-4ad7-a931-45524a0cace2" />

`ImageOptim`是一款开源免费的图片压缩工具。

<img width="322" alt="image" src="https://github.com/user-attachments/assets/e8b86c11-2846-4d60-b54a-a8ad6ff072dc" />

`Snipaste`是一款跨平台功能强大的截图与贴图工具，提供了个人免费版本与付费版。

`licecap`是一款跨平台开源的gif录制工具。

<img width="934" alt="image" src="https://github.com/user-attachments/assets/df10b133-3fb7-4e76-92b3-dd4128e4d63b" />

`drawio`是一款开源的画图工具。跨平台免费真好用。

<img width="1355" alt="image" src="https://github.com/user-attachments/assets/335c4601-cec1-4529-a0f2-d800b9795b0a" />

`syncthing`是一款开源的跨平台的文件同步工具。


## alias与bash函数

弄别名与bash函数可以加速处理命令行操作。

```bash
cc() {
    git rev-list --count $1
}

rebase() {
    git rebase -i HEAD~$1 && git push -f
}

status() {
    git status
}

log() {
    git log
}

push() {
    git add --all && git commit -m "$1" && git push
}

# squash 4 "update"
squash() {
    git reset --soft HEAD~$1 && git add --all && git commit -m "$2." && git push -f
}

touchm() {
    find $1 -type f -exec touch -m {} \;
}


# rmall ~/ ".DS_Store"
rmall() {
    find $1 -name $2 -exec rm {} \;
}

rmdsstore() {
    find . -name ".DS_Store" -exec rm {} \;
}

# renameext jpg zip
renameext() {
    rename "s/$1/$2/" *
}

cls() {
    /usr/bin/osascript -e 'tell application "System Events" to tell process "Terminal" to keystroke "k" using command down'
}

# xattrd ~/Downloads/1.app
xattrd() {
    /usr/bin/xattr -r -d com.apple.quarantine $1
    # or
    /usr/bin/xattr -c $1
}

# random 32
random() {
    echo $RANDOM | md5sum | head -c $1; echo;
}

xattrd() {
    if [[ -z $1 ]]; then
        echo xattrd path
        return
    fi
    xattr -r -d com.apple.quarantine "$1"
}

alias clean=cls

alias clear="printf '\33c\e[3J'"

alias ll="ls -al"

alias simlog="tail -f ~/Library/Logs/CoreSimulator/*/system.log"
```

## 配置

一些常用工具还需要额外的配置一下。

### vscode配置

使用下面的命令安装插件。

```bash
extensions=(
  "redhat.java"
  "vscjava.vscode-gradle"
  "vscjava.vscode-java-pack"
  "vscjava.vscode-java-debug"
  "adelphes.android-dev-ext"
  "OmarDulaimi.breakpoint-bookmarks"
  "ms-dotnettools.csharp"
  "bito.bito"
  "bpfdeploy.bpftrace"
  "codezombiech.gitignore"
  "cornell3110sp20.rml-highlighter"
  "dbaeumer.vscode-eslint"
  "eriklynd.json-tools"
  "formulahendry.code-runner"
  "foxundermoon.shell-format"
  "genieai.chatgpt-vscode"
  "github.codespaces"
  "github.copilot"
  "github.copilot-chat"
  "github.github-vscode-theme"
  "github.remotehub"
  "github.vscode-github-actions"
  "github.vscode-pull-request-github"
  "golang.go"
  "google.aidl-language"
  "gruntfuggly.todo-tree"
  "jebbs.plantuml"
  "josetr.cmake-language-support-vscode"
  "jrieken.md-navigate"
  "mesonbuild.mesonbuild"
  "ms-azuretools.vscode-docker"
  "ms-ceintl.vscode-language-pack-zh-hans"
  "ms-dotnettools.vscode-dotnet-runtime"
  "ms-python.autopep8"
  "ms-python.debugpy"
  "ms-python.isort"
  "ms-python.python"
  "ms-python.vscode-pylance"
  "ms-vscode-remote.remote-containers"
  "ms-vscode-remote.remote-ssh"
  "ms-vscode-remote.remote-ssh-edit"
  "ms-vscode-remote.remote-wsl"
  "ms-vscode-remote.vscode-remote-extensionpack"
  "ms-vscode.cmake-tools"
  "ms-vscode.cpptools"
  "ms-vscode.cpptools-extension-pack"
  "ms-vscode.cpptools-themes"
  "ms-vscode.hexeditor"
  "ms-vscode.makefile-tools"
  "ms-vscode.powershell"
  "ms-vscode.remote-explorer"
  "ms-vscode.remote-repositories"
  "ms-vscode.remote-server"
  "ms-vscode.vscode-typescript-next"
  "osstekz.vala-code"
  "prince781.vala"
  "redhat.vscode-commons"
  "redhat.vscode-xml"
  "redhat.vscode-yaml"
  "rogalmic.bash-debug"
  "twxs.cmake"
  "vadimcn.vscode-lldb"
  "vscjava.vscode-maven"
  "vscode-icons-team.vscode-icons"
  "xaver.clang-format"
  "shd101wyy.markdown-preview-enhanced"
  "saoudrizwan.claude-dev"
  "Continue.continue"
)

for extension in "${extensions[@]}"; do
  echo "正在安装扩展: $extension"
  code --install-extension "$extension"
done
```

<img width="619" alt="image" src="https://github.com/user-attachments/assets/07a0db37-8898-41d8-942e-fee8a7856d56" />


### orbstack配置

启动`orbstack`一次。然后命令行配置它。

```bash
orb config docker
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

Ctrl+O，Ctrl+X保存并退出。下载镜像测试，效果如下：

<img width="1015" alt="image" src="https://github.com/user-attachments/assets/96e5b88b-28cb-4914-94ba-3bad4453cd5b" />


安装一个arm64版本的Ubuntu系统日常使用。

```bash
orb create ubuntu:jammy ubuntu
```

安装一个x86_64版本的Ubuntu系统解决一些x86_64使用场景。

```bash
softwareupdate --install-rosetta --agree-to-license
orb create --arch amd64 ubuntu:jammy ubuntu64
```

安装好这些虚拟机系统后，进去后docker配置使用下面的命令。

```bash
sudo tee /etc/docker/daemon.json <<EOF
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
EOF
```

### UTM配置

安装好程序后，下载Ubuntu镜像。地址可以点击界面的浏览UTM库，然后选择Ubuntu。
目前最新下载地址是：[https://archive.org/download/ubuntu-20.04-arm64-utm/ubuntu-20.04-arm64-utm.zip](https://archive.org/download/ubuntu-20.04-arm64-utm/ubuntu-20.04-arm64-utm.zip)

下载解压，然后双击Ubuntu22.04.utm，会自动导入。右链编译，改内存8g。
<img width="805" alt="image" src="https://github.com/user-attachments/assets/dd09d6e5-da8c-4c8f-a85c-bf59bba38a2e" />

改共享，添加主机的下载目录，方便虚拟机与主机共享文件。
<img width="799" alt="image" src="https://github.com/user-attachments/assets/9e3c817b-8dec-429c-8b6d-2d28b9026d14" />

改网络，虚拟vlan方式，可以添加端口转发，方便ssh连接

<img width="798" alt="image" src="https://github.com/user-attachments/assets/749615f6-f601-4973-a5bb-714b5afc60f5" />

完事后，启动虚拟机，输入用户名与密码：`ubuntu`。打开终端安装`openssh-server`与其它需要用到的软件。

<img width="798" alt="image" src="https://github.com/user-attachments/assets/05ceb9e0-fcdb-4345-98a8-da94ab2d5763" />


### Chrome设置

主要是扩展程序。访问页面是：https://chromewebstore.google.com/detail/xxx
xxx为扩展的ID。

我安装的一些扩展如下。

```bash
cfhdojbkjhnklbpkdaibdccddilifddb
anlikcnbgdeidpacdbdljnabclhahhmd
dodmmooeoklaejobgleioelladacbeki
ghbmnnjooekpmoecnnnilnnbdlolhkhi
mclkkofklkfljcocdinagocijmpgbhab
cpcifbdmkopohnnofedkjghjiclmhdah
legbfeljfbjgfifnkmpoajgpgejojooj
padekgcemlokbadohgkifijomclgjgif
kpdjmbiefanbdgnkcikhllpmjnnllbbc
lieodnapokbjkkdkhdljlllmgkmdokcm
ikhdkkncnoglghljlkmcimlnlhkeamad
bciglihaegkdhoogebcdblfhppoilclp
bpoadfkcbjbfhfodiogcnhhhpibjhbnh
dhdgffkkebhmkfjojejmpbldmpobfkfo
```

### iTerm2配置

导入主题：

```
git clone https://github.com/mbadolato/iTerm2-Color-Schemes --depth 1
cd iTerm2-Color-Schemes
tools/import-scheme.sh schemes/*
```

重启iTerm2，cmd+,打开设置，切换到Profiles->Default->Colors，Color Presets里面选主题，比如Ubuntu主题。

然后执行：```touch ~/.hushlogin```禁止显示最后登陆信息。

打开 Preferences > Profiles > Window。找到 Style 设置，选择No Title Bar，禁止显示标题。

<img width="1360" alt="image" src="https://github.com/user-attachments/assets/6f616495-dbe7-40de-bb6a-ae40f431a883" />

解决中文件乱码：

在.bash_profile或.zshrc中加入：

```
export LANG="en_US.UTF-8"
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
export LC_COLLATE="en_US.UTF-8"
```

### 科学软件

2024比较常用的有`ClashX Pro`与`clash-for-windows`。
2025使用`clash-verge-rev`的好像更多一些。

这里就不展开安装与配置了。

### 收费软件

根据个人使用习惯，还有一些收费的软件是要使用到的。比如`CleanMyMac X`、`Parallels Desktop`、`IDA Pro`、`Typora`、`Beyond Compare`、`paragon-extfs`、`paragon-ntfs`、`010-editor`。它们多可以通过`brew`来安装，`IDA Pro`除外，安装好后可以试用或购买。

`Parallels Desktop`就是一年一订真的贵，看自己的需要了。

这是说一下`Office`，微软对“免费”使用目前查得并不严格，`brew`安装的版本，可以使用网上的`2021 VL license`无限制使用。使用`microsoft-auto-update`可以完美的升级更新，有条件还是建议购买正版，好用省事。

**[Office激活](https://github.com/alsyundawy/Microsoft-Office-For-MacOS)**

## 参考

https://brew.sh/

https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/

https://mirrors.tuna.tsinghua.edu.cn/help/pypi/

https://code.visualstudio.com/docs/editor/extension-marketplace

https://docs.orbstack.dev/machines/

https://www.sweetscape.com/download/010editor/

https://www.paragon-software.com/

https://developer.chrome.com/docs/extensions/how-to/distribute/install-extensions?hl=zh-cn

https://github.com/gitbito/CLI

https://github.com/pypa/pipx
