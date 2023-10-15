﻿![在这里插入图片描述](https://img-blog.csdnimg.cn/eb0e7327bd454ca7b3aeb8d855915202.png)
# 前言
内存取证在ctf比赛中也是常见的题目，内存取证是指在计算机系统的内存中进行取证分析，以获取有关计算机系统当前状态的信息。内存取证通常用于分析计算机系统上运行的进程、网络连接、文件、注册表等信息，并可以用于检测和分析恶意软件、网络攻击和其他安全事件

# 工具安装
## python与pip安装方法

首先就是安装python和pip，在kali和一些linux发行版上，python都是自带的，python和pip安装方法如下：
```
sudo apt-get update  #更新源
sudo apt-get install python2   #安装python2
sudo apt-get install python-pip2   #安装pip2
```

## 下载和安装Volatility
Volatility是一款开源的内存分析框架，主要用于从计算机内存中提取数字证据。它可以用于取证、恶意代码分析、漏洞研究、操作系统学习以及其他安全领域

Volatility项目地址：
```
https://github.com/volatilityfoundation/volatility
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a8898fcc1b14b55b7d4810c30ff5689.png)

如果你在国外的话，可以直接运行这个命令来获取Volatility项目，在国内用这个命令的话，下载非常慢
```
apt install git
git clone https://github.com/volatilityfoundation/volatility.git
```

从压缩包里提取文件
```
unzip [file_name] -d [destination]  #filename：你要提取的压缩包名称，destination：提取后的文件存放位置
```

进入文件夹，运行以下命令即可安装
```
python2 setup.py install
```

## 依赖安装
如果不安装依赖，Volatility很多功能都用不了
```
pip2 install pycryptodome -i https://pypi.tuna.tsinghua.edu.cn/simple
pip2 install yara -i https://pypi.tuna.tsinghua.edu.cn/simple
pip2 install distorm3 -i https://pypi.tuna.tsinghua.edu.cn/simple
```
如果distorm3安装失败的话，只能手动去安装了

项目地址：
```
https://github.com/vext01/distorm3
```
下载解压后进入文件夹，运行以下命令即可
```
chmod 777 setup.py
python2 setup.py install
```

mimikatz脚本文件下载地址
```
链接：https://pan.baidu.com/s/1HS65N4UXfuzChB9UU9vveA 
提取码：fywk 
```
然后将这个脚本文件移动到/volatility/plugins目录下
```
mv mimkatz.py /volatility/plugins/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6550fff74dc845a7862b057bf2d61c71.png)


然后安装construct库
```
sudo pip2 install construct==2.5.5-reupload
```

# 演示题目的下载地址
本片文章使用的案例：
```
链接：https://pan.baidu.com/s/1nD-svI98v3yQPPKT0HyEjg 
提取码：0dnx 
```
# 工具的使用方法
## 获取内存镜像详细信息
imageinfo是Volatility中用于获取内存镜像信息的命令。它可以用于确定内存镜像的操作系统类型、版本、架构等信息，以及确定应该使用哪个插件进行内存分析
```
python2 vol.py -f Challenge.raw imageinfo  #f：指定分析的内存镜像文件名
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3422e6b9f4134545b38e28f440bf4b43.png)
```
上述输出中，Suggested Profile(s) 显示了 Volatility 推荐的几个内存镜像分析配置文件，可以根据这些配置文件来选择合适的插件进行内存分析
AS Layer2 显示了使用的内存镜像文件路径
KDBG 显示了内存镜像中的 KDBG 结构地址
Number of Processors 显示了处理器数量
Image Type 显示了操作系统服务包版本
Image date and time 显示了内存镜像文件的创建日期和时间
```
## 获取正在运行的程序
这里我们用Win7SP1x64配置文件进行分析，Volatility 的 pslist 插件可以遍历内存镜像中的进程列表，显示每个进程的进程 ID、名称、父进程 ID、创建时间、退出时间和路径等信息
```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 pslist
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/45abecfd68f44f519f29117157ed8202.png)
## 提取正在运行的程序
Volatility 的 procdump 插件可以根据进程 ID 或进程名称提取进程的内存映像，并保存为一个单独的文件

比如这里我要提取iexplore.exe这个程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/91b4b7d94a0941bb847d2ca1dd718e20.png)

他的进程pid号为2728

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 procdump -p 2728 -D ./
p：pid进程号
D：提取程序后保存的地址，./指的是当前shell正在运行的文件夹地址，输入pwd命令可以查看shell当前的地址，简单来说就是保存到当前文件夹
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/44983433c85949ea973d4a8847f83c80.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/7cc5cb6ea5894a01af7d2f43b43c4e12.png)

成功导出，导出后文件名为executable.2728.exe

![在这里插入图片描述](https://img-blog.csdnimg.cn/75235b0e2ab8468ba0d9bf89c226c2ec.png)

## 查看在终端里执行过的命令
Volatility 的 cmdscan 插件可以扫描内存镜像中的进程对象，提取已执行的 cmd 命令，并将其显示在终端中

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 cmdscan
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a294e3353534d53acfbc4d681935203.png)

他移动到了Documents目录下，echo了一次字符串，然后创建了一个名为hint.txt的文件

## 查看进程在终端里运行的命令
Volatility中的cmdline插件可以用于提取进程执行的命令行参数和参数值
```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 cmdline
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/63133cffd6c6438d8dac28a7fdec6a33.png)




## 查找内存中的文件
Volatility 的 filescan插件可以在内存中搜索已经打开的文件句柄，从而获取文件名、路径、文件大小等信息

我想找到hint.txt文件，可以使用以下命令

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 filescan | grep hint.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ae18212c27504c8886f86feb340466cc.png)

grep是Linux下常用的命令之一，它用于在文件中查找指定的字符串，并将包含该字符串的行输出

如果只使用filescan而不配合grep的话，Volatility就会输出系统上的全部文件，例如：



![在这里插入图片描述](https://img-blog.csdnimg.cn/6d3fc9ae14074b72b91684b01cbd91a2.png)

## 提取内存中的文件
Volatility的dumpfiles插件可以用来提取系统内存中的文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/a780dec2b2984df5b323dee8912e744f.png)


这里我要提取hint.txt文件，hint.txt的内存位置为0x000000011fd0ca70，这两个由于位置都一样，随便提取哪个都行

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000011fd0ca70 -D ./
Q：内存位置
D：提取程序后保存的地址，./指的是当前shell正在运行的文件夹地址，输入pwd命令可以查看shell当前的地址，简单来说就是保存到当前文件夹
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/535675245dd846e9a588e036ab6c4428.png)

提取出来的文件名是包含内存地址的，更改一下后缀名即可运行

## 查看浏览器历史记录
Volatility中的iehistory插件可以用于提取Internet Explorer浏览器历史记录
```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 iehistory
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/a232c42576c74dfca9c3879eddd1ba5b.png)

## 提取用户密码hash值并爆破
Volatility中的Hashdump插件可以用于提取系统内存中的密码哈希值
```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 hashdump
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7936d30ef7bd47739846e8c717da9af5.png)

这里提取了四个用户的密码hash值，我们将这些字符串复制一下，粘贴到本地本文里

![在这里插入图片描述](https://img-blog.csdnimg.cn/a1de0d61b1c5411fb5307512a86d050d.png)

我们可以使用这个在线网站：
```
https://crackstation.net/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2f67c4a94399411c8ebae5afe0ed8cef.png)


将hash值粘贴上去

![在这里插入图片描述](https://img-blog.csdnimg.cn/a2a920a2960b445d9bd1aed994ab0903.png)

就可以得到用户密码明文

## 使用mimikatz提取密码
mimikatz是一个开源的用于从Windows操作系统中提取明文密码，哈希值以及其他安全凭据的工具

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 mimikatz
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/efc11fc02f9b4ef5990e72c614ce0e03.png)

成功提取到TroubleMaker用户的密码

## 查看剪切板里的内容
Volatility中的clipboard插件可以用于从内存转储中提取剪贴板数据

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 clipboard
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/29ae5ed5721449be99d7bd02a63b30e8.png)

## 查看正在运行的服务
svcscan是Volatility中的一个插件，用于扫描进程中所有的服务

```
svcscan
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/402e4071e224454ba767e7f648382c3b.png)

执行了svcscan之后，每列代表服务的一些信息，包括服务名、PID、服务状态、服务类型、路径等等
## 查看网络连接状态
Volatility中的netscan插件可以在内存转储中查找打开的网络连接和套接字，该命令将显示所有当前打开的网络连接和套接字。输出包括本地和远程地址、端口、进程ID和进程名称等信息

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 netscan
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/902b4ff077424d839920315f41516242.png)

## 查看注册表信息

printkey是Volatility工具中用于查看注册表的插件之一。它可以帮助分析人员查看和解析注册表中的键值，并提供有关键值的详细信息，如名称、数据类型、大小和值等

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 printkey
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b78477c28c3d4013a6339b5c7440c90e.png)


然后使用hivelist插件来确定注册表的地址

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 hivelist
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/dee0917b8cd04757b7574753e848d77b.png)

查看注册表software项

hivedump是一个Volatility插件，用于从内存中提取Windows注册表的内容，这里我们选择第一个来演示

```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 hivedump -o 0xfffff8a00127d010
o：hivelist列出的Virtual值
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/471fb2976a1e4fdcbb239c4d528dde9b.png)

根据名称查看具体子项的内容，这里以SAM\Domains\Account\Users\Names做演示，这个是Windows系统中存储本地用户账户信息的注册表路径，它包含了每个本地用户账户的名称和对应的SID信息
```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 printkey -K "SAM\Domains\Account\Users\Names"
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/abf787f806634700b013b8583bfaab5b.png)

如果要提取全部的注册表，可以用这个命令
```
python2 vol.py -f Challenge.raw --profile=Win7SP1x64 dumpregistry -D ./
```


## 全部插件
```
amcache        	查看AmCache应用程序痕迹信息
apihooks       	检测内核及进程的内存空间中的API hook
atoms          	列出会话及窗口站atom表
atomscan       	Atom表的池扫描(Pool scanner)
auditpol       	列出注册表HKLMSECURITYPolicyPolAdtEv的审计策略信息
bigpools       	使用BigPagePoolScanner转储大分页池(big page pools)
bioskbd        	从实时模式内存中读取键盘缓冲数据(早期电脑可以读取出BIOS开机密码)
cachedump      	获取内存中缓存的域帐号的密码哈希
callbacks      	打印全系统通知例程
clipboard      	提取Windows剪贴板中的内容
cmdline        	显示进程命令行参数
cmdscan        	提取执行的命令行历史记录（扫描_COMMAND_HISTORY信息）
connections    	打印系统打开的网络连接(仅支持Windows XP 和2003)
connscan       	打印TCP连接信息
consoles       	提取执行的命令行历史记录（扫描_CONSOLE_INFORMATION信息）
crashinfo      	提取崩溃转储信息
deskscan       	tagDESKTOP池扫描(Poolscaner)
devicetree     	显示设备树信息
dlldump        	从进程地址空间转储动态链接库
dlllist        	打印每个进程加载的动态链接库列表
driverirp      	IRP hook驱动检测
drivermodule   	关联驱动对象至内核模块
driverscan     	驱动对象池扫描
dumpcerts      	提取RAS私钥及SSL公钥
dumpfiles      	提取内存中映射或缓存的文件
dumpregistry   	转储内存中注册表信息至磁盘
editbox        	查看Edit编辑控件信息 (Listbox正在实验中)
envars         	显示进程的环境变量
eventhooks     	打印Windows事件hook详细信息
evtlogs        	提取Windows事件日志（仅支持XP/2003)
filescan       	提取文件对象（file objects）池信息
gahti          	转储用户句柄（handle）类型信息
gditimers      	打印已安装的GDI计时器(timers)及回调(callbacks)
gdt            	显示全局描述符表(Global Deor Table)
getservicesids 	获取注册表中的服务名称并返回SID信息
getsids        	打印每个进程的SID信息
handles        	打印每个进程打开的句柄的列表
hashdump       	转储内存中的Windows帐户密码哈希(LM/NTLM)
hibinfo        	转储休眠文件信息
hivedump       	打印注册表配置单元信息
hivelist       	打印注册表配置单元列表
hivescan       	注册表配置单元池扫描
hpakextract    	从HPAK文件（Fast Dump格式）提取物理内存数据
hpakinfo       	查看HPAK文件属性及相关信息
idt            	显示中断描述符表(Interrupt Deor Table)
iehistory      	重建IE缓存及访问历史记录
imagecopy      	将物理地址空间导出原生DD镜像文件
imageinfo      	查看/识别镜像信息
impscan        	扫描对导入函数的调用
joblinks       	打印进程任务链接信息
kdbgscan       	搜索和转储潜在KDBG值
kpcrscan       	搜索和转储潜在KPCR值
ldrmodules     	检测未链接的动态链接DLL
lsadump        	从注册表中提取LSA密钥信息（已解密）
machoinfo      	转储Mach-O 文件格式信息
malfind        	查找隐藏的和插入的代码
mbrparser      	扫描并解析潜在的主引导记录(MBR)
memdump        	转储进程的可寻址内存
memmap         	打印内存映射
messagehooks   	桌面和窗口消息钩子的线程列表
mftparser      	扫描并解析潜在的MFT条目
moddump        	转储内核驱动程序到可执行文件的示例
modscan        	内核模块池扫描
modules        	打印加载模块的列表
multiscan      	批量扫描各种对象
mutantscan     	对互斥对象池扫描
notepad        	查看记事本当前显示的文本
objtypescan    	扫描窗口对象类型对象
patcher        	基于页面扫描的补丁程序内存
poolpeek       	可配置的池扫描器插件
printkey       	打印注册表项及其子项和值
privs          	显示进程权限
procdump       	进程转储到一个可执行文件示例
pslist         	按照EPROCESS列表打印所有正在运行的进程
psscan         	进程对象池扫描
pstree         	以树型方式打印进程列表
psxview        	查找带有隐藏进程的所有进程列表
qemuinfo       	转储 Qemu 信息
raw2dmp        	将物理内存原生数据转换为windbg崩溃转储格式
screenshot     	基于GDI Windows的虚拟屏幕截图保存
servicediff    	Windows服务列表(ala Plugx)
sessions       	_MM_SESSION_SPACE的详细信息列表(用户登录会话)
shellbags      	打印Shellbags信息
shimcache      	解析应用程序兼容性Shim缓存注册表项
shutdowntime   	从内存中的注册表信息获取机器关机时间
sockets        	打印已打开套接字列表
sockscan       	TCP套接字对象池扫描
ssdt           	显示SSDT条目
strings        	物理到虚拟地址的偏移匹配(需要一些时间，带详细信息)
svcscan        	Windows服务列表扫描
symlinkscan    	符号链接对象池扫描
thrdscan       	线程对象池扫描
threads        	调查_ETHREAD 和_KTHREADs
timeliner      	创建内存中的各种痕迹信息的时间线
timers         	打印内核计时器及关联模块的DPC
truecryptmaster	Recover 	恢复TrueCrypt 7.1a主密钥
truecryptpassphrase		查找并提取TrueCrypt密码
truecryptsummary	TrueCrypt摘要信息
unloadedmodules	打印卸载的模块信息列表
userassist     	打印注册表中UserAssist相关信息
userhandles    	转储用户句柄表
vaddump        	转储VAD数据为文件
vadinfo        	转储VAD信息
vadtree        	以树形方式显示VAD树信息
vadwalk        	显示遍历VAD树
vboxinfo       	转储Virtualbox信息（虚拟机）
verinfo        	打印PE镜像中的版本信息
vmwareinfo     	转储VMware VMSS/VMSN 信息
volshell       	内存镜像中的shell
windows        	打印桌面窗口(详细信息)
wintree        	Z顺序打印桌面窗口树
wndscan        	池扫描窗口站
yarascan       	以Yara签名扫描进程或内核内存
```
# 总结
本篇文章演示的插件已经可以做绝大部分题目了，之后就多在buuctf或者ctfshow等线上ctf平台刷题，积累经验

