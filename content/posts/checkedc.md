+++
date = '2025-08-18T14:30:49+08:00'
draft = true
title = 'Checkedc'
tags = ['c语言']
+++

# 介绍
Checked C是微软开源的一个C 语言的扩展和 C++ 语言的子集，为这些语言增加边界检查。

# 安装环境
首先下载Checked C 标准库。从[checked](git clone https://github.com/microsoft/checkedc.git) 下载到本地。之后可以在编译时指定库位置 
```
clang -fcheckedc-extension -I ~/checkedc/include helloworld.c
```

或者加入到环境变量中, 这样 `clang` 会自动在这个目录找头文件。(windows平台cmd)
```shell
set CPATH=C:\Users\zbz\Documents\AWork\Projects\checkedc-main\include;%CPATH%
```
之后编译
```shell
clang -fcheckedc-extension -g -O0 -fno-omit-frame-pointer helloworld.c
```

其中 
- -fcheckedc-extension 参数是打开Checked C 扩展
- -g -O0 -fno-omit-frame-pointer  保留调试信息与栈


# checkedc-clang
官方提供的Checked C编译器。可以直接下载对应平台的编译器[release](https://github.com/checkedc/checkedc-clang/releases)  下载后安装。

安装后如果没有添加到系统的PATH环境变量，可以临时指定
```
set PATH=C:\Program Files\CheckedC-Clang\bin;%PATH%
```

# 从源码buildcheckedc-clang 
平台windows， 准备以下环境

- Visual Studio 2022, Python (version 3.7~3.11 用于执行测试), and versions of UNIX command-line tools.
- 通过 Cygwin 安装 UNIX 命令行工具。 安装 `Base` 包。`Cygwin` 安装目录的`bin` 目录，添加到系统路径 。
- 安装Ninja （用于构建checkedc-clang)

根据 [github](https://github.com/checkedc/checkedc-clang/blob/main/clang/docs/checkedc/Setup-and-Build.md)提示 设置

1. 选择一个目录，设为工作目录 `<WORK_DIR>`
2. 克隆 `checkedc-clang` 存储库  `git clone -c core.autocrlf=false https://github.com/checkedc/checkedc-clang src`  注意将仓库命名为src
3. 在项目内`src\llvm\projects\checkedc-wrapper` 目录内，克隆 Checked C仓库`git clone https://github.com/checkedc/checkedc`
4. 在工作目录 `<WORK_DIR>` 下，创建build目录并进入
5. 执行以下cmake命令

```
cmake -G Ninja -DLLVM_ENABLE_PROJECTS=clang -DCMAKE_INSTALL_PREFIX=C:\Users\zbz\Documents\AWork\Projects\checked-c\install -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON  -DLLVM_INSTALL_TOOLCHAIN_ONLY=ON  -DLLVM_TARGETS_TO_BUILD="X86" C:\Users\zbz\Documents\AWork\Projects\checked-c\src\llvm
```

6.  执行cmake后用ninja构建编译器
```
ninja            // This command will build the compiler and all other supporting tools.
OR
ninja clang      // This command will build only the compiler.

ninja clean      // This command cleans the build directory.
```

构建之后的可执行文件在`bin`目录下， 可以直接使用
```
build\bin\clang.exe --version

# output
clang version 12.0.0 
Target: x86_64-pc-windows-msvc
Thread model: posix
InstalledDir: C:\Users\zbz\Documents\AWork\Projects\checked-c\build\bin
```


# 编译c程序
用命令编译
```
clang -g -fcheckedc-extension -fsanitize=bounds -o demo.exe demo.c
```