---
title: Windows PowerShell常用命令整理
date: 2023-10-16
tags: [Windows, PowerShell]
---

在 Windows 系统管理和运维中，PowerShell 是一个强大的工具，提供了各种命令来执行任务、管理系统和执行自动化操作。在本文中，我们将介绍一些常见的 PowerShell 命令，以及如何使用它们来管理 Windows 系统。

### 模拟阻塞命令

有时候，我们需要在脚本或批处理中添加延迟。使用 `Start-Sleep` 命令，你可以模拟阻塞操作，例如等待 10 秒：

```powershell
Start-Sleep -Seconds 10
```

### 查看当前登录的用户名

要查看当前登录用户的用户名，可以使用 `whoami` 命令：

```powershell
whoami
```

### 查看当前环境变量

查看当前用户的环境变量可以使用以下命令：

```powershell
echo $env:PATH
```

### 查看命令路径

如果你想查找特定命令的路径，例如 `sqlplus`，可以使用 `get-command` 命令：

```powershell
get-command sqlplus
```

### 获取所有用户的环境变量

要获取系统中所有用户的环境变量，可以使用 `Get-WmiObject` 命令：

```powershell
Get-WmiObject -Class Win32_Environment
```

### 获取文件权限

如果你需要查看文件或目录的权限设置，可以使用 `Get-Acl` 命令。例如，要查看 `C:\` 的权限，可以运行：

```powershell
Get-Acl C:\
```

### 查询进程信息

PowerShell 允许你查询和筛选正在运行的进程。例如，要查找包含特定关键字的进程，可以使用以下命令：

```powershell
Get-Process | Where-Object { $_.ProcessName -like "*关键字*" }
```

这些是一些常见的 PowerShell 命令，可以帮助你进行 Windows 运维和管理任务。无论是自动化任务、查询系统信息还是管理权限，PowerShell 提供了丰富的功能，可用于广泛的系统管理任务。