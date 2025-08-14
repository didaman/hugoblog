+++
date = '2025-08-14T14:59:43+08:00'
draft = false
title = 'Cpptools'
tags = ['VSCode插件']
+++

# vscode-cpptools 代码目录 本地代码路径
```
Extension
├─src
  ├─Debugger
    ├─attachQuickPick.ts  主要用于实现 "附加到进程" 的功能，提供了一个交互式界面让用户选择要调试的进程。
    ├─attachToProcess.ts  主要负责实现进程附加功能，允许用户将调试器附加到正在运行的进程上。
    ├─configurationProvider.ts
    ├─configurations.ts
    ├─debugAdapterDescriptorFactory.ts  创建和配置调试适配器描述符
    ├─extension.ts  扩展注册
    ├─nativeAttach.ts  本地进程附加功能
    ├─ParsedEnvironmentFile.ts
    ├─utils.ts
```

VS Code ←→ Debug Adapter（cpptools调用 OpenDebugAD7 / vsdbg.exe）←→ **MIEngine**（Visual Studio 的 MI 引擎）←→ **gdb / lldb（以 MI 模式）**

# 调试详细流程

## 1 用户在 VS Code 按 F5 / Run

- 通过在launch.json中配置 `cppvsdbg`或`cppdbg` 来控制使用哪种DA

## 2 启动 Debug Adapter
cpptools中 src-Debugger-debugAdapterDescriptorFactory.ts中定义了不同的DA描述工厂。根据配置调用OpenDebugAD7 / WindowsDebugLauncher等
实现Debug Adapter Protocol（DAP）的一端，和 VS Code 对话

### WindowsDebugLauncher 的IO / 进程包装
负责以合适的方式启动 gdb、管理 stdin/stdout/stderr 的重定向（通常会把这些流写到临时文件名如 `Microsoft-MIEngine-Out-*.zkv` 等）
并把 pid/文件名等参数交给 MIEngine / 调试适配层


## 3 MIEngine 被初始化并成为“翻译器”
vscode-cpptools 通过调用MIEngine 实现c/c++ 调试 [MIEngine 仓库地址](https://github.com/microsoft/MIEngine)

高层调试动作转成 GDB/MI 命令、解析 MI 输出、并管理调试会话

OpenDebugAD7 会把 DAP 的请求传给 MIEngine，MIEngine 负责和真实的 debugger（gdb / lldb）通信

## 4 MIEngine 启动并控制 gdb/lldb 进程（以 MI 模式）
- MIEngine 启动一个 gdb（或 lldb-mi）子进程，带上 `--interpreter=mi` 或等价参数
- MIEngine 向 gdb 发送 MI 命令（例如 `-break-insert`、`-exec-run`），异步读取标准输出/错误里以 MI 格式返回的数据。
- 解析这些数据，映射成DAP信息

# MIEngine代码详解

