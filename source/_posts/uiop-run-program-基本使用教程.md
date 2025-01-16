---
title: uiop-run-program 基本使用教程
date: 2025-01-16 18:28:41
updated:
categories: Lisp
tags: [Common Lisp,uiop:run-program]
---
uiop:run-program 是 Common Lisp 中 UIOP (Utilities for Implementation- and OS- Portability) 库的一部分，用于在 Lisp 环境中执行外部程序和命令。这个函数提供了一个强大的接口来启动外部进程，并可以处理进程的输入、输出和退出状态。以下是一个关于如何使用 uiop:run-program 的入门教程：
<!-- more -->
### 安装和加载 UIOP

UIOP 是 ASDF (Another System Definition Facility) 的一部分，通常已经包含在现代 Common Lisp 实现中。你可以通过 Quicklisp 来确保 UIOP 可用：
```
(ql:quickload "uiop")
```
### 基本用法
### 1. **执行简单命令**

执行 Windows 命令行中的 `dir` 命令（类似于 Unix 的 `ls` 命令），列出当前目录内容：

```
(uiop:run-program "cmd /c dir")
```
### 2. **捕获命令输出**

获取 `dir` 命令的输出并保存在 Lisp 变量中：
```
(let ((output (uiop:run-program "cmd /c dir" :output :string)))
  (format t "~A~%" output))
```

### 3. **执行外部程序**

运行外部的可执行文件（比如 `notepad.exe`）：

```
(uiop:run-program "notepad.exe")
```
执行上述命令会打开记事本

### 4. **传递参数给外部程序**

执行外部程序时传递参数，例如用 `echo` 命令输出一段文本：
```
(uiop:run-program "cmd /c echo Hello, World!")
```

### 5. **指定输入数据**

将一段输入传递给命令行程序。例如，通过 `findstr` 命令在输入数据中查找特定字符串：
```
(let ((output (uiop:run-program "cmd /c findstr foo"
                               :input "foo\nbar\nfoobar\n"
                               :output :string)))
  (format t "Matched Output:~%~A~%" output))

```

### 6. **捕获错误输出**

捕获外部程序的标准错误输出。例如，试图访问不存在的文件，输出错误消息：
```
(let ((error-output (uiop:run-program "cmd /c type nonexistent-file.txt"
                                     :output :string
                                     :error :string
                                     :ignore-error-status t)))
  (format t "Error Output:~%~A~%" error-output))

```

### 7. **执行批处理脚本**

执行一个 `.bat` 文件。例如，假设你有一个 `test.bat` 文件：
```
(uiop:run-program "cmd /c test.bat")
```
如果 `test.bat` 内容如下：
```
@echo off
echo Hello from the batch script!
```
输出将会是：
```
Hello from the batch script!
```

### 8. **执行 PowerShell 命令**

执行 Windows PowerShell 命令：
```
(uiop:run-program "powershell -Command \"Write-Output 'Hello, PowerShell!'\"")
```
### 9. **将命令输出保存到文件**

将 `dir` 命令的输出保存到文件 `output.txt`：
```
(uiop:run-program "cmd /c dir" :output "output.txt")
```

### 10. **忽略非零退出状态**

执行一个会失败的命令（如 `exit 1`），忽略错误状态，防止 Lisp 抛出异常：
```
(uiop:run-program "cmd /c exit 1" :ignore-error-status t)
```

### 11. **重定向输入和输出**

从文件 `input.txt` 读取内容，通过外部命令处理并将输出写入 `output.txt`：
```
(uiop:run-program "cmd /c sort" :input "input.txt" :output "output.txt")
```

### 12. **使用参数列表**

将命令及其参数分开作为列表传递，而不是字符串：
```
(uiop:run-program '("cmd" "/c" "echo" "Hello, World!"))
```

### 13. **执行 Windows 系统工具**

运行 Windows 自带的工具，例如 `systeminfo` 查看系统信息：
```
(uiop:run-program "cmd /c systeminfo")
```

### 14. **执行可执行文件并等待退出**

运行外部程序并等待其退出，例如运行 `calc.exe` 打开计算器：
```
(uiop:run-program "calc.exe")
```
### 15. **执行带路径的程序**

如果外部程序在指定路径下，可以直接传入完整路径：
```
(uiop:run-program "C:\\Windows\\System32\\notepad.exe")
```
### 16. **执行并行命令**

在 `cmd` 中使用 `&&` 执行多个命令，例如先执行 `echo` 然后执行 `dir`：
```
(uiop:run-program "cmd /c echo Start && dir")
```
### 17. **捕获命令退出状态**

你可以捕获命令的退出状态（例如 0 表示成功，非零表示错误）：
```
(let ((status (uiop:run-program "cmd /c exit 2" :ignore-error-status t)))
  (format t "Exit Status: ~A~%" status))
```
### 总结

通过以上示例，你可以看到 **`uiop:run-program`** 在 Windows 下的多种用法，包括：

1. 执行系统命令（如 `cmd /c dir`）。
2. 调用外部程序（如 `notepad.exe`、`calc.exe`）。
3. 传递输入输出数据。
4. 捕获标准输出和错误输出。
5. 处理批处理脚本和 PowerShell 命令。
6. 将输出重定向到文件。

