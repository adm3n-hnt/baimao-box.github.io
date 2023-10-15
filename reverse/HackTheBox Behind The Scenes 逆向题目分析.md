![在这里插入图片描述](https://img-blog.csdnimg.cn/f711959ca7ac44b6852b5a5f3211198d.png)
题目地址：
```
https://app.hackthebox.com/challenges/301
```
题目简介的意思是这个程序使用了加密来保护密码
# 开始
下载完程序后，我尝试使用ida来静态分析，可是ida无法打开程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/c83d570cb0454f33a10f91c51e3ec35f.png)

把程序拖入die分析，也没看到什么加密方式

![在这里插入图片描述](https://img-blog.csdnimg.cn/cbc42081df5c43af84c6fd2080078348.png)

然后我用strings查看一下程序内部的字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/01f44088e45244cf96ae9e2156a26a8e.png)

没能找到什么有用的字符串

然后我使用r2来动态分析程序，发现了一个可疑的地方

![在这里插入图片描述](https://img-blog.csdnimg.cn/601414b8150a4aaab66ce772fdab6f38.png)

在main函数的末尾是用ud2指令结束的，正常程序的结尾一般都是pop和ret，弹出和返回，可这里不一样，我查阅了x86手册后发现

![在这里插入图片描述](https://img-blog.csdnimg.cn/e83af29f8d8f4ae0b09936512db70b9d.png)

此指令用于软件测试以显式生成无效操作码。该指令的操作码是为此目的而保留的。

除了引发无效操作码异常外，该指令与 NOP 指令相同。

知道了问题所在，现在就该解决问题了，我们可以使用NOP指令来替换 UD2 指令
NOP的指令为：0x90
UD2的指令为：0x0f 0x0b
我们可以使用bbe工具来替换指令
```
bbe -e 's/\x0f\x0b/\x90\x90/g' behindthescenes > new
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2399b1094ed47a2ba5a355bcd1d497b.png)

然后用r2来动态调试这个新生成的程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/d9f4e602e3204febb079ebf98ea994af.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/de2ac0a5b08e44978eef3c3a7629d595.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fb69a35873f542c98ca48ccd6c9cfb02.png)

可以看到，这里main函数多了很多的指令，也在字符串列表看到了程序的密码

![在这里插入图片描述](https://img-blog.csdnimg.cn/8f11224922ee4427b58e15bee4e9f140.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4b56fd8d3a614c309f73588c4bfebb48.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d80f7365eb046a1892514ed85481841.png)

密码为：Itz_0nLy_UD2

![在这里插入图片描述](https://img-blog.csdnimg.cn/124ffa2828ea47669a288188d15f3609.png)

