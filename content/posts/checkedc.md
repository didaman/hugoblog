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

# vscode调试
准备环境
- vscode
- clang 编译器
- C/C++ 插件

配置launch.json 用cppvsdbg模式。仅用于Windows系统。虽然checkedc-clang包含clang编译器和lldb调试器，但是lldb-mi已经废弃不再支持。所以windows平台用cppvsdbg，调试生成PDB/CodeView 信息。详细调试逻辑参考[[VScode调试架构]]

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug checkedc-clang Program",
      "type": "cppvsdbg",
      "request": "launch",
      "program": "${workspaceFolder}\\build\\${fileBasenameNoExtension}.exe",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [],
      "console": true,
      "preLaunchTask": "build-standard-C",
    }
  ]
}
```

## 调试显示信息
在调试过程中，调试器不显示 `_Ptr<int>` 类型，本质是因为 Checked C 是 C 语言的非标准扩展，其类型系统未被调试器原生支持，且编译器在生成调试信息时会优先使用标准类型表示以确保兼容性。

一些对指针的约束操作，仅在编译期告诉编译器指针的约束。编译后的二进制文件中，被修饰的指针，底层存储的依然是普通的内存地址。


编译器在生成调试符号（用于调试器解析类型的信息）时，通常会 “剥离” `_Nonnull` 和 `_Nullable` 这类 Checked C 特有的修饰符，仅保留标准 C 兼容的类型信息。 例如checkedc代码
```c
int main() {
    int x = 0;
    _Ptr<int> xp = &x;
    // xp = xp + 1;  // compile-time error: pointer arithmetic not allowed on _Ptr
    printf("%d\n", *xp);
}
```
变量xp是一个Ptr类型，一个安全的指针。编译后，用命令查看符号文件
```
llvm-pdbutil dump -symbols build\checked_pointer.pdb > checked_ptr_pdb.txt
```
得到结果
```txt
     156 | S_LOCAL [size = 16] `xp`
           type=0x0674 (int*), flags = none
     172 | S_DEFRANGE_FRAMEPOINTER_REL [size = 16]
           offset = 40, range = [0001:29844,+66)
           gaps = 2
     188 | S_END [size = 4]
```

可以看到，xp变量，定义的类型是`int*`, 和标准c语言定义的指针是一样的。