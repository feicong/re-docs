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

### 逆向分析工具

- **IDA Pro**（反汇编器）
- **Ghidra**（开源反编译工具）
- **Apktool**（安卓反编译工具）

### 动态调试工具

- **Frida**（动态插桩框架）

### 网络安全工具

- **Wireshark**（网络抓包工具）
- **Fiddler**（HTTP 调试代理）
- **Burp Suite**（Web 渗透测试工具）

## 开发环境配置

### 编程语言环境

### 安装 Python

```powershell
winget install Python.Python.3
```

安装常见的安全研究库：

```powershell
pip install pwntools frida requests scapy capstone unicorn
```

### 安装 Visual Studio

- 选择 **Desktop development with C++** 组件。
- 安装 **Windows SDK** 和 **调试工具**。

### 安装虚拟机环境

- **VMware Workstation** 或 **VirtualBox**。
- 创建 **Windows 沙盒环境** 进行动态分析。

### 启用 Windows Subsystem for Linux（WSL）

```powershell
wsl --install -d Ubuntu
```

安装常用工具：

```bash
sudo apt update && sudo apt install gdb ltrace strace binwalk
```
