![在这里插入图片描述](https://img-blog.csdnimg.cn/3d0476ba41df41cb9da4525dd2f426b3.png)
# 简介
在编写程序时，有很多人调用第三方程序使用的是相对路径，而不是绝对路径，如下：
```
#!/bin/python3

import os

os.system("whoami") #调用whoami程序，查看当前用户名
```


```
#!/bin/bash

find / -name "helloworld.c"  #调用find程序，搜索根目录里名为helloworld.c的文件
```


```
#include <stdio.h>
#include <stdlib.h>

int main(void){
        system("ping -c 1 127.0.0.1");  #ping一次127.0.0.1地址
}
```
……
```
tpmtool.exe drivertracing stop
```
这个工具是windows自带的一个工具，它的官方解释是：

受信任的平台模块 (TPM) 技术旨在提供基于硬件的安全相关功能。 TPM芯片是一种安全的加密处理器，旨在执行加密操作。 该芯片包含多个物理安全机制以使其防篡改，并且恶意软件无法篡改TPM的安全功能

这条命令的意思是停止收集TPM驱动的程序日志

这条命令是没什么问题，但是运行这个程序时，程序本身启动了一个新的cmd进程，然后调用了另一个程序，这就造成了一些问题，下面讲实例的时候会详细说
# 关于什么是绝对路径和相对路径
我举一个例子，这是绝对路径
```
/usr/bin/whoami
C:\Windows\System32\calc.exe
```
而这是相对路径
```
whoami
calc.exe
```
一个是指定路径下的，一个是根据当前环境变量调用的，那么，环境变量是什么
# 什么是环境变量
![在这里插入图片描述](https://img-blog.csdnimg.cn/65402828a2544cce86abd8ffc51f2690.png)

任何计算机编程语言的两个基本组成部分，变量和常量。就像数学方程式中的自变量一样。变量和常量都代表唯一的内存位置，其中包含程序在其计算中使用的数据。两者的区别在于，变量在执行过程中可能会发生变化，而常量不能重新赋值

这里只举几个常见的环境变量
# $PATH
包含了一些目录列表，作用是终端会在这些目录中搜索要执行的程序
查看$PATH环境变量
```
echo $PATH
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2dbe298e262444b0ab238c6b645259d6.png)

假如我要执行whoami程序，那么终端会在这个环境变量里搜索名为whoami程序

搜索的目录如下
```
/usr/local/sbin
/usr/local/bin
/usr/sbin
/usr/bin
/sbin
/bin
/usr/local/games
/usr/games
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/37b867aaa8ed48f593bebc35ec89229b.png)

而whoami程序在/usr/bin目录下，终端会执行这个目录下的whoami程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/0abd0a154bc94c8b8ce588a9ab70195a.png)

而windows的PATH环境变量在这可以看到

![在这里插入图片描述](https://img-blog.csdnimg.cn/c6cb510cb52241e8b3a8881fdfca5b5e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/fed50f07a53341d6bfd0687cd81e3d19.png)

# $HOME
包含了当前用户的主目录

```
echo $HOME
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d5844b513da4a0d893278415a02c29c.png)
# $PWD
包含了当前用户目前所在的目录位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/fdb7ecf67bcf49b993596d9be27c84c2.png)

关于环境变量的更多信息：

```
https://en.wikipedia.org/wiki/Environment_variable
```
# 编写程序时调用第三方程序时使用的是相对路径而不是绝对路径会造成什么严重后果
## Windows
在介绍了什么是绝对路径和相对路径，以及什么是环境变量后，现在开始进入主题了，我用上面介绍的windows自带的工具tpmtool.exe来演示
```
tpmtool.exe drivertracing stop
```
上面说过，这条命令的作用是停止收集TPM驱动的程序日志的，我们在cmd里运行这条命令

![在这里插入图片描述](https://img-blog.csdnimg.cn/65e94d35ee13430daf9ff954cb64c3bf.png)

工具弹出错误了，不过并不重要，我们只是拿这个工具做演示，能运行即可
接下来我们要用Procmon工具来监视tpmtool.exe这个工具，Procmon工具下载地址：
```
ht去掉字符tps://learn.microsoft.com/en-us/sysinternals/downloads/procmon
```
双击启动这个工具，点击筛选

![在这里插入图片描述](https://img-blog.csdnimg.cn/0c60c5189ee74828998177430a5fb6f2.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4a4867d5c72c47eab41c5b45eb3c5a7a.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/61b6d53139b84f4ba29613a94eb1e72a.png)

设置完成之后，删除当前监听的内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/dfd34b270afb4cf990c660ea18cbcd77.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/105d06026c1c4ef9b900f9868b1bdbf5.png)



然后回到cmd，重新运行命令

![在这里插入图片描述](https://img-blog.csdnimg.cn/bafb42762a344f0f9221942f17f5872f.png)

然后返回Procmon，在下面可以看到tpmtool工具的一些奇怪的调用

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf62f0795d2d4daba90719b1b847fc67.png)

这个工具创建了一个子进程，调用了cmd程序，为了清楚它做了什么，我们将cmd也加入过滤

![在这里插入图片描述](https://img-blog.csdnimg.cn/ee21ebc5fae5487e994699ad53f18abb.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/30be0a06af0746e8a9234ddc61aa89c4.png)


添加完之后在下面可以看到调用

![在这里插入图片描述](https://img-blog.csdnimg.cn/60783eb1fa6e4653b160839089fb850d.png)


tpmtool创建了一个子进程cmd，然后这个cmd执行了一些命令，但是并没有路径显示，我们双击查看详细信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/c97a0720dd7b435a9b22ef5f4531a1e5.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f22f90cd741b4ffb9d33cebae7a063fa.png)

tpmtool工具创建了一个子进程调用cmd，并执行了logman.exe stop TPMTRACE -ets这条命令
但是，问题就出在这里，它执行这条命令时并没有指定logman.exe的绝对目录，只是声明了logman.exe

假如我有一个名叫ogman.exe的恶意软件会怎么样

这里我将计算器来当作恶意软件做演示，现在将计算器移动到当前文件夹下
```
copy C:\Windows\System32\calc.exe .
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d9e4be7fdd7e4448aa367a468f30717b.png)


然后将计算器更名为logman.exe，这里我们并不需要改变当前的环境变量，只需要保证logman.exe程序在当前运行目录下即可

```
move calc.exe logman.exe
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a63fc37dc0fa4f1ea6682f4a2e202e22.png)

然后运行一开始的命令
```
tpmtool.exe drivertracing stop
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/11b245829d7540bcb9429b78f9d09dc5.png)

可以看到，成功的弹出了计算器程序

## Linux
在这个目录下有一个c语言的程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/3f388ddd5f844e3ea98299050b28b14b.png)

查看源码

![在这里插入图片描述](https://img-blog.csdnimg.cn/e9767386f95d4aa8a227fe51ba06a218.png)

这个程序只是执行了一个系统命令curl，这个工具的作用是获取指定网站的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/0efa7156f26140a0b4f8fa2a63850cf3.png)


但是和上面windows分析的那个程序一样，没有指定工具的绝对路径

我们在当前文件夹里创建一个名为curl的文件

```
echo "pwd" > curl
chmod +x curl
```
然后改变当前终端的环境变量

```
PATH=$PWD:$PATH
```
$PATH环境变量=当前文件夹

然后执行程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/7eebd1a360b64ecab0d5cb63464bd511.png)

成功执行了pwd程序

# 总结
很多小细节的错误堆在一起可能会造成一个大的错误，在做hackthebox和tryhackme的机子时，经常会利用这种方式来对机子进行提权
