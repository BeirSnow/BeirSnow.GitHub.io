---
layout: post
title:  "EDKII Windows 开发环境搭建"
date:   2022-05-24 09:15:00 +0800
categories: BIOS
tags: bios
---

### 一、获取 EDKII 源码
#### 1. 获取源码：
[github 源码链接](https://github.com/tianocore/edk2)  
推荐使用 `git clone --recursive https://github.com/tianocore/edk2.git` 直接把所有依赖全 clone 下来。不然可能会因 get submodules 不完整，遇到本文 (三 - 5) 中的问题：
```bash
BrotliCompress.c(20): fatal error C1083: ޷򿪰ļ: ./brotli/c/common/constants.h: No such file or directory
```
#### 2. 获取最新的 Submodules：
**第 1 步中使用 `git clone --recursive` 命令 clone 仓库的忽略以下步骤**
>To get a full, buildable EDK II repository, use following steps of git command
```bash
git clone https://github.com/tianocore/edk2.git
cd edk2
git submodule update --init
cd ..
```
>If there's update for submodules, use following git commands to get the latest submodules code.
```bash
cd edk2
git pull
git submodule update
```
>Note: When cloning submodule repos, '--recursive' option is not recommended. EDK II itself will not use any code/feature from submodules in above submodules. So using '--recursive' adds a dependency on being able to reach servers we do not actually want any code from, as well as needlessly downloading code we will not use.

### 二、阅读 ReadMe
#### 1. 根据 ReadMe 的指引，找到以下文档：
[Getting Started with EDK II](https://github.com/tianocore/tianocore.github.io/wiki/Getting-Started-with-EDK-II)

### 三、搭建 Windows 开发环境
#### 1. 安装 Visual Studio 2019
- 安装时勾选 C++ 桌面开发组件
![VS2019 Install](/assets/images/BIOS/EDKII-build-development-environment/VS2019_install.png)

#### 2. 安装 Python
>Install Python37 or late version (https://www.python.org/) to run python tool from source

#### 3. 安装 NASM
- 下载链接： [NASM](http://www.nasm.us/)（**下载 Win32/Win64 的绿色包即可**）
- 解压 "NASM" 文件夹至 C 盘根目录（最好是 C 盘根目录，免得出奇怪的问题）
- 新建环境变量：
![NASM env](/assets/images/BIOS/EDKII-build-development-environment/NASM_env.png)


#### 4. 安装 ASL
- 下载链接：[ASL](https://acpica.org/downloads/binary-tools)
- 将压缩包中的 "iasl.exe" 解压至 "C:\ASL", 没有这个路径的话要先创建 "C:\ASL"
- 添加环境变量至 path：
![ASL path](/assets/images/BIOS/EDKII-build-development-environment/ASL_path.png)

#### 5. 编译 BaseTools
根据 [Getting Started with EDK II](https://github.com/tianocore/tianocore.github.io/wiki/Getting-Started-with-EDK-II) 的指引，使用以下命令编译 BaseTools
```bash
$ D:\edk2> set PYTHON_HOME=C:\Users\beirs\AppData\Local\Programs\Python\Python310
$ D:\edk2> edksetup.bat Rebuild
```
然后就是喜闻乐见的世事无常大肠包小肠：
```bash
BrotliCompress.c(20): fatal error C1083: ޷򿪰ļ: ./brotli/c/common/constants.h: No such file or directory
```
此问题是 submodules 不完整导致，**建议回到本文开头，按照建议重新 clone 仓库**

#### 6. 编译 EmulatorPkg （UEFI 仿真）
- 运行 edksetup 脚本，建立命令行 build 命令环境（**如果 powershell 下 build 命令不可用，可尝试 cmd 执行**）
```bash
$ D:\edk2> .\edksetup.bat
```

- 阅读 "EmulatorPkg\Readme.md", 根据文档提示执行 build 命令，编译 UEFI 仿真环境
```bash
build -p EmulatorPkg\EmulatorPkg.dsc -t VS2019 -a X64
```
编译完成如下：
![ASL path](/assets/images/BIOS/EDKII-build-development-environment/build_emu.png)

- 阅读 "EmulatorPkg\Readme.md", 根据文档提示执行 UEFI 仿真器
```bash
cd Build\EmulatorX64\DEBUG_VS2017\X64\ && WinHost.exe
```
然后，ohohoh，真棒啊！可以开心耍了
![ASL path](/assets/images/BIOS/EDKII-build-development-environment/emu0.png)
![ASL path](/assets/images/BIOS/EDKII-build-development-environment/emu1.png)

### 四、EDKII Samples
推荐看看源码中以下模块的实现，都是比较实用的示例：
- ShellPkg\Application\
- MdeModulePkg\Application\HelloWorld
- MdeModulePkg\Universal\DriverSampleDxe

EDKII 官方文档链接：
[https://github.com/tianocore/tianocore.github.io/wiki/EDK-II-Documents](https://github.com/tianocore/tianocore.github.io/wiki/EDK-II-Documents)
