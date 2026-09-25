---
title: mac_to_windows
date: 2026-09-17 22:47:00
tags:
---
前言
之前我在 Windows 的 PowerShell 里敲了个 touch index.html，结果直接报错：

vim : 无法将“vim”项识别为 cmdlet、函数、脚本文件或可运行程序的名称
当时就有点懵。后来才反应过来——Windows 上压根没有 touch。如果你也从 Linux 或 macOS 转到 Windows，或者反过来，这种“命令明明存在却用不了”的情况几乎每天都能碰到。这篇文章整理了一下常用的命令对照，还有一些容易踩的坑。（如果你想在windows使用vim就必须将它配置好，其实后来在vim环境变量配又遇到问题，这里推荐直接下载vim）
同理touch这个好用的命令也有办法将它移植到windows


一、常用命令对照
1. 文件与目录
功能	Linux (Bash)	Windows CMD	PowerShell
列出文件	ls	dir	Get-ChildItem / ls
列出隐藏文件	ls -a	dir /a	Get-ChildItem -Force
详细列表	ls -l	dir（默认详细）	Get-ChildItem | Format-List
创建空文件	touch file	type nul > file	New-Item file / ni file
创建目录	mkdir dir	mkdir dir / md dir	New-Item -ItemType Directory dir
删除文件	rm file	del file	Remove-Item file / rm file
删除目录	rm -r dir	rmdir /s dir	Remove-Item -Recurse dir
复制文件	cp a b	copy a b	Copy-Item a b
复制目录	cp -r src dst	xcopy src dst /s /e	Copy-Item -Recurse src dst
移动	mv a b	move a b	Move-Item a b
重命名	mv a b	ren a b	Rename-Item a b
查看文件内容	cat file	type file	Get-Content file / cat file
分页查看	less file	more file	more file
查看文件头	head -n 10 file	不支持	Get-Content file -TotalCount 10
查看文件尾	tail -n 10 file	不支持	Get-Content file -Tail 10
实时追踪文件	tail -f file	不支持	Get-Content file -Wait
查找文件	find . -name "*.txt"	dir /s /b *.txt	Get-ChildItem -Recurse -Filter *.txt
搜索文件内容	grep "foo" file	findstr "foo" file	Select-String "foo" file
递归搜索内容	grep -r "foo" .	findstr /s "foo" *	Get-ChildItem -Recurse | Select-String "foo"
查找命令位置	which vim	where vim	Get-Command vim
当前目录	pwd	cd（无参数）	Get-Location / pwd
切换目录	cd dir	cd dir / chdir	Set-Location dir / cd
符号链接	ln -s target link	mklink link target	New-Item -ItemType SymbolicLink
修改文件时间	touch -t 202609171200 file	不支持	(Get-Item file).LastWriteTime = Get-Date
几个具体的例子：

bash
# Linux：建多个空文件
touch index.html style.css app.js

# Linux：找所有 .log 文件
find . -name "*.log" -type f

# Linux：递归搜 error
grep -r "error" . --include="*.log"
cmd
:: CMD：建空文件
type nul > index.html

:: CMD：递归找 .log
dir /s /b *.log

:: CMD：搜 error
findstr /s /i "error" *.log
powershell
# PowerShell：建多个文件
"index.html", "style.css", "app.js" | ForEach-Object { New-Item $_ }

# PowerShell：递归找 .log
Get-ChildItem -Recurse -Filter *.log

# PowerShell：递归搜内容
Get-ChildItem -Recurse -Include *.log | Select-String "error"
2. 文本处理
功能	Linux	Windows CMD	PowerShell
排序	sort file	sort file	Sort-Object
去重	uniq	不支持	Get-Unique
统计行数	wc -l file	find /c /v "" file	(Get-Content file).Count
字段提取	awk '{print $1}'	不支持	ForEach-Object { $_.Split()[0] }
流编辑	sed 's/old/new/g'	不支持	-replace 'old','new'
比较文件	diff a b	fc a b	Compare-Object (Get-Content a) (Get-Content b)
正则匹配	grep -E "pattern"	findstr /r "pattern"	Select-String -Pattern
例子：

bash
# Linux：统计 error 出现次数
grep -c "error" app.log

# Linux：提取日志里的 IP，排序去重计数
awk '{print $1}' access.log | sort | uniq -c | sort -rn

# Linux：全局替换
sed 's/foo/bar/g' file.txt
cmd
:: CMD：统计行数
find /c /v "" file.txt

:: CMD：比较文件
fc file1.txt file2.txt
powershell
# PowerShell：统计 error 次数
(Select-String "error" app.log).Count

# PowerShell：对象管道，直接按属性筛
Get-Process | Where-Object { $_.CPU -gt 100 } | Sort-Object CPU -Descending | Select-Object -First 5

# PowerShell：替换文本
(Get-Content file.txt) -replace 'foo','bar' | Set-Content file.txt
3. 系统与进程
功能	Linux	Windows CMD	PowerShell
查看环境变量	env / printenv	set	Get-ChildItem Env:
设置环境变量	export VAR=val	set VAR=val	$env:VAR = "val"
查看单个变量	echo $VAR	echo %VAR%	$env:VAR
进程列表	ps aux	tasklist	Get-Process
结束进程	kill PID	taskkill /PID PID	Stop-Process -Id PID
按名称结束	pkill name	taskkill /IM name	Stop-Process -Name name
查看磁盘	df -h	wmic logicaldisk get size,freespace,caption	Get-PSDrive / Get-Volume
查看内存	free -h	wmic OS get TotalVisibleMemorySize,FreePhysicalMemory	Get-CimInstance Win32_OperatingSystem
查看系统信息	uname -a	systeminfo	Get-ComputerInfo
重启	reboot / shutdown -r now	shutdown /r /t 0	Restart-Computer
关机	shutdown -h now	shutdown /s /t 0	Stop-Computer
清屏	clear	cls	Clear-Host / cls
历史	history	doskey /history	Get-History
当前用户	whoami	whoami	whoami / $env:USERNAME
例子：

bash
# Linux：CPU 占用前5
ps aux --sort=-%cpu | head -6

# Linux：磁盘
df -h

# Linux：内存
free -h
cmd
:: CMD：磁盘
wmic logicaldisk get size,freespace,caption

:: CMD：结束记事本
taskkill /IM notepad.exe
powershell
# PowerShell：CPU 占用前5
Get-Process | Where-Object { $_.CPU -gt 100 } | Sort-Object CPU -Descending | Select-Object -First 5

# PowerShell：磁盘
Get-Volume

# PowerShell：临时环境变量
$env:MY_VAR = "hello"
4. 网络
功能	Linux	Windows CMD	PowerShell
查看 IP	ip addr show / ifconfig	ipconfig	Get-NetIPAddress
查看所有 IP	ip addr	ipconfig /all	Get-NetIPAddress
测试连通	ping host	ping host	Test-Connection host
路由表	ip route show / route -n	route print	Get-NetRoute
跟踪路由	traceroute host	tracert host	Test-NetConnection -TraceRoute
ARP 表	arp -n	arp -a	Get-NetNeighbor
端口监听	ss -tlnp / netstat -tlnp	netstat -an	Get-NetTCPConnection -State Listen
下载	wget url / curl -O url	curl url（Win10+）	Invoke-WebRequest url -OutFile file
SSH	ssh user@host	ssh user@host（需装 OpenSSH）	ssh user@host
例子：

bash
# Linux：接口
ip addr show

# Linux：下载
wget https://example.com/file.zip
# 或
curl -O https://example.com/file.zip
cmd
:: CMD：完整网络配置
ipconfig /all

:: CMD：端口监听
netstat -an
powershell
# PowerShell：网络配置
Get-NetIPAddress

# PowerShell：下载
Invoke-WebRequest -Uri "https://example.com/file.zip" -OutFile "file.zip"

# PowerShell：测试连通
Test-Connection google.com

5. 权限与用户
功能	Linux	Windows CMD	PowerShell
查看文件权限	ls -l file	icacls file	Get-Acl file
修改权限	chmod 755 file	icacls file /grant User:F	Set-Acl（需要构造 ACL）
修改所有者	chown user file	icacls file /setowner User	Set-Acl
添加用户	useradd username	net user username password /add	New-LocalUser username
删除用户	userdel username	net user username /delete	Remove-LocalUser username
修改密码	passwd username	net user username newpass	Set-LocalUser -Password
用户列表	cat /etc/passwd	net user	Get-LocalUser
切换用户	su - username	runas /user:username cmd	Start-Process -Credential
管理员运行	sudo command	右键 → 以管理员身份运行	Start-Process -Verb RunAs
例子：

bash
# Linux：看权限
ls -l script.sh
# 输出：-rwxr-xr-x 1 user group 1234 ... script.sh

# Linux：加可执行
chmod +x script.sh

# Linux：设 755
chmod 755 script.sh

# Linux：加用户
useradd -m -s /bin/bash newuser

# Linux：sudo
sudo apt update
cmd
:: CMD：看 ACL
icacls myfile.txt

:: CMD：给 Alice 读权限
icacls myfile.txt /grant Alice:R

:: CMD：加用户
net user newuser password123 /add
powershell
# PowerShell：看 ACL
Get-Acl myfile.txt

# PowerShell：管理员运行
Start-Process powershell -Verb RunAs

# PowerShell：加本地用户
New-LocalUser -Name "newuser" -Password (ConvertTo-SecureString "pass123" -AsPlainText -Force)
6. 服务
功能	Linux	Windows CMD	PowerShell
查看状态	systemctl status name	sc query name	Get-Service name
启动	systemctl start name	net start name / sc start name	Start-Service name
停止	systemctl stop name	net stop name / sc stop name	Stop-Service name
重启	systemctl restart name	sc stop name && net start name	Restart-Service name
开机自启	systemctl enable name	sc config name start=auto	Set-Service -StartupType Automatic
列出所有	systemctl list-units --type=service	sc query type= service	Get-Service
例子：

bash
# Linux：看 nginx
systemctl status nginx

# Linux：启动
sudo systemctl start nginx

# Linux：自启
sudo systemctl enable nginx
cmd
:: CMD：看 Spooler
sc query Spooler

:: CMD：启动
net start Spooler

:: CMD：自启
sc config Spooler start= auto
powershell
# PowerShell：看服务
Get-Service nginx

# PowerShell：启动
Start-Service nginx

# PowerShell：自启
Set-Service -Name nginx -StartupType Automatic
7. 压缩归档
功能	Linux	Windows CMD	PowerShell
压缩目录	tar -czf archive.tar.gz dir/	tar -czf archive.tar.gz dir/（Win10+）	Compress-Archive dir archive.zip
解压	tar -xzf archive.tar.gz	tar -xzf archive.tar.gz	Expand-Archive archive.zip dest
ZIP 压缩	zip -r archive.zip dir/	不支持（需第三方）	Compress-Archive dir archive.zip
查看内容	tar -tzf archive.tar.gz	tar -tzf archive.tar.gz	Get-ChildItem archive.zip
例子：

bash
# Linux：压缩
tar -czf backup.tar.gz /home/user/data/

# Linux：解压
tar -xzf backup.tar.gz
powershell
# PowerShell：压缩
Compress-Archive -Path .\data -DestinationPath .\backup.zip

# PowerShell：解压
Expand-Archive -Path .\backup.zip -DestinationPath .\data
8. 软件包管理
功能	Linux (Debian/Ubuntu)	Linux (RHEL/CentOS)	Windows	PowerShell
安装	apt install pkg	yum install pkg	下载 .exe	winget install pkg
卸载	apt remove pkg	yum remove pkg	控制面板	winget uninstall pkg
更新	apt upgrade	yum update	Windows Update	winget upgrade
搜索	apt search pkg	yum search pkg	不支持	winget search pkg
已安装	dpkg -l	rpm -qa	wmic product get name	winget list
例子：

bash
# Linux：装 nginx
sudo apt update && sudo apt install nginx

# Linux：搜软件
apt search "text editor"
powershell
# PowerShell：装 Vim
winget install --id vim.vim -e

# PowerShell：搜软件
winget search "text editor"

# PowerShell：已安装
winget list
二、为什么命令名不一样？
Linux的思路是每个命令只干一件事，靠管道拼起来。比如 cat、grep、awk、sed 各管一摊，组合起来就能做很复杂的事：

bash
cat access.log | grep "404" | awk '{print $1}' | sort | uniq -c | sort -rn
CMD 就有点像 DOS 时代留下来的东西。命令本身功能多，但不好组合，管道传的是文本，处理能力弱。

PowerShell 完全不一样。它传的是 .NET 对象，不是文本。你可以直接访问属性：

powershell
Get-Process | Where-Object { $_.CPU -gt 100 } | Sort-Object CPU -Descending
在 Linux 里你还需要 awk 去切文本，在 PowerShell 里直接 .CPU 就完事了。这个差别挺大的。

三、容易踩的坑
1. 路径分隔符
Linux 用 /，Windows 用 \。PowerShell 里两者都能用，但 CMD 里必须用 \。写脚本的时候注意一下。

2. 大小写
Linux 区分大小写，File.txt 和 file.txt 是两个文件。Windows 不区分，是同一个。在 Linux 上跑得好好的脚本，到 Windows 上可能因为大小写出问题。

3. 换行符
Linux 是 LF（\n），Windows 是 CRLF（\r\n）。Git 跨平台协作时如果没配 core.autocrlf，会看到“整个文件都被改了”的假象。我之前在 WSL 里跑脚本，报错 set: invalid option，查了半天才发现是 CRLF 搞的鬼。

4. 环境变量
Linux Bash：$VAR 或 ${VAR}

CMD：%VAR%

PowerShell：$env:VAR

在 PowerShell 里写 %PATH% 是没用的，必须用 $env:Path。

5. 可执行权限
Linux 靠 rwx 权限位，脚本第一行 #!/bin/bash 指定解释器，得 chmod +x 才能跑。Windows 靠扩展名（.exe、.bat、.ps1），没有 chmod 这种东西，权限是 ACL 管的。

6. 通配符
Linux 里 *.txt 是 shell 先展开，再传给命令。CMD 里是命令自己处理，比如 dir *.txt 是 dir 在解析。

7. ls 和 dir 的输出
ls 默认紧凑，要 -l 才详细，默认不显示隐藏文件（. 开头的），要 -a。dir 默认就详细，显示大小和日期，默认也显示隐藏文件。

四、跨平台怎么办？
用跨平台工具：Git Bash 在 Windows 上模拟 Bash，touch、ls、grep 都能用。WSL 更彻底，直接跑真正的 Linux，apt、systemd 都有。MSYS2 / Cygwin 也类似。

Git Bash 和 WSL 有个区别：Git Bash 跑的是 Windows 程序或者移植过来的 Unix 工具，而 WSL 跑的是真正的 Linux 二进制。在 WSL 里不能直接调 Windows 的 .exe，得用完整路径。

PowerShell 别名：PowerShell 内置了 ls、cat、rm、pwd 这些别名，但只是别名，参数和行为不一定和 Linux 一致。比如 ls -la 就不一定能用。

直接学 PowerShell：如果你长期在 Windows 上干活，花时间学 PowerShell 的对象管道，比强行套 Linux 命令效率高得多。

微软的 Coreutils：微软在 Build 2026 上推出了 Windows Coreutils，把 cat、ls、grep、head 等 75+ 个 Unix 命令移植到了 Windows CMD 和 PowerShell 上，体积才 4.6MB。

五、一点建议
别死记命令名，理解自己要做什么，然后查对应平台的写法。

能用 PowerShell 就用 PowerShell，CMD 已经是历史了。

跨平台项目用 Git Bash 或 WSL，能避开路径和换行符的坑。

写脚本前先确认目标平台，别指望一套命令走天下。

PowerShell 的对象管道值得花时间学，习惯了之后处理结构化数据比 Linux 的文本管道舒服。

结语
Linux 和 Windows 终端命令的差异，表面是名字不一样，实际上是设计思路不同。Linux 是小工具加管道，CMD 是 DOS 遗产，PowerShell 是对象加管道。想明白这点，就不会纠结“为什么 Windows 没有 touch”，而是会去查“Windows 里创建空文件该用什么”。
这才是跨平台该有的心态。
