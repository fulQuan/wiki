---
title: "Linux 基础知识"
layout: page
date: 2016-07-18
---
[TOC]
## 关于
作为一个合格的工程师，怎能不懂Linux，所以学习记录一些知识点。

## 常用命令集合
### 文件系统相关
- 重定向：`2>&1`表示将stderr重定向到stdout，`2>/dev/null`表示重定向stderr到空洞，也就是不打印出stderr。
  后面这个在运行hadoop和spark的时候有用。也可以将stdout和stderr重定向到不同文件，例如：        
```bash
some-cmd 1>stdout.txt 2>stderr.txt
```

### 权限相关
- `sudo`:      
在`sudo`出现之前，使用`su`命令提升权限，缺点是必须知道超级用户的密码。`sudo`可以将用户的名字、可以使用的特殊命令、按照那种用户组执行等信息保存在
文件中（通常是`/etc/sudoers`）
sudo [-bhHpV][-s ][-u <用户>][指令]
或
sudo [-klv]
参数[编辑]
　　-b 　在后台执行指令。
　　-h 　显示帮助。
　　-H 　将HOME环境变量设为新身份的HOME环境变量。
　　-k 　结束密码的有效期限，也就是下次再执行sudo时便需要输入密码。
　　-l 　列出目前用户可执行与无法执行的指令。
　　-p 　改变询问密码的提示符号。
　　-s 　执行指定的shell。
　　-u <用户> 　以指定的用户作为新的身份。若不加上此参数，则预设以root作为新的身份。
　　-v 　延长密码有效期限5分钟。
　　-V 　显示版本信息。
   -S   从标准输入流替代终端来获取密码
### 操作系统相关
后台可靠运行的几种方法：          
- 后台运行加一个`&`即可，例如`sleep 100 &`，只能一直保持会话，如果shell会话关了，这个子进程也会关掉。     
- nohup 方式：    
我们知道，当用户注销（logout）或者网络断开时，终端会收到 HUP（hangup）信号从而关闭其所有子进程。因此，我们的解决办法就有两种途径：要么让进程忽略 HUP 信号，要么让进程运行在新的会话里从而成为不属于此终端的子进程。
只需在要处理的命令前加上 nohup 即可，标准输出和标准错误缺省会被重定向到 nohup.out 文件中。一般我们可在结尾加上"&"来将命令同时放入后台运行，也可用">filename 2>&1"来更改缺省的重定向文件名。           
- setsid 方式：    
nohup 无疑能通过忽略 HUP 信号来使我们的进程避免中途被中断，但如果我们换个角度思考，如果我们的进程不属于接受 HUP 信号的终端的子进程，那么自然也就不会受到 HUP 信号的影响了。setsid 就能帮助我们做到这一点。               
使用的时候，也只需在要处理的命令前加上 setsid 即可。
- 当我们将"&"也放入“()”内之后，我们就会发现所提交的作业并不在作业列表中，也就是说，是无法通过jobs来查看的。让我们来看看为什么这样就能躲过 HUP 信号的影响吧。                     
```bash   
subshell 示例
[root@pvcent107 ~]# (ping www.ibm.com &)
[root@pvcent107 ~]# ps -ef |grep www.ibm.com
root     16270     1  0 14:13 pts/4    00:00:00 ping www.ibm.com
root     16278 15362  0 14:13 pts/4    00:00:00 grep www.ibm.com
[root@pvcent107 ~]#
```
从上例中可以看出，新提交的进程的父 ID（PPID）为1（init 进程的 PID），并不是当前终端的进程 ID。因此并不属于当前终端的子进程，从而也就不会受到当前终端的 HUP 信号的影响了。
- 如果已经提交了任务，但是没加nohup或者setsid，怎么补救？可以用 `disown` 命令来补救。
    - `disown -h jobspec` 是某个作业忽略HUP信号
    - `disown -ah` 是所有作业忽略HUP信号
    - `disown -rh` 是正在运行的作业忽略HUP信号
- 更多技巧，参考IBM文档
1. IBM文档 <https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/>
### 字符串工具
- grep 字符匹配
```bash
## 从log.txt中查找keyword出现的行
grep keyword log.txt
## 查询多个模式
grep -E 'keyword|otherword' log.txt
grep -e keyword -e otherword log.txt
```
### sort 命令
将文件的每一行作为一个单位，进行排序，从首字符开始，按照ASCII码进行比较（默认情况）。            
- u 参数，去重
- r 参数，降序
- o 参数，排序后写入原文件，重定向会将文件内容清空，达不到要求，可以用这个参数
- n 参数，表示不按照ASCII码排序，而是按照数值大小
- t 参数 和 k 参数，用来排序csv格式，t 指明分隔符 k 指明排序的列序号
- f 参数，忽略大小写
- c 参数，检查是否排好序，输出第一个乱序行信息，返回1；C参数，也是检查，但不输出乱序行信息
- M 参数，以月份排序，会识别月份（只对英文吧）
- b 参数，忽略每一行前面所有空白