# 简介
需要此文章用到的虚拟机环境和工具可以加我qq3316735898
# 环境介绍
可以去看一下我上一篇文章
```
https://blog.csdn.net/qq_45894840/article/details/124607902?spm=1001.2014.3001.5501
```
# 开始
将题目解压到目录里
![在这里插入图片描述](https://img-blog.csdnimg.cn/6b9217f486ef4e4fb9d12ea423a80420.png)
一共有48个可执行程序
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8183d652e7c4c6c95d3c818637da723.png)
我们随意选择一个文件拖入ida分析一下
用peid分析一下文件，发现这个程序是32位的，而且没有加密和混淆
![在这里插入图片描述](https://img-blog.csdnimg.cn/277a8b5c77674e31a580f846fe6169fd.png)
将文件拖入ida
![在这里插入图片描述](https://img-blog.csdnimg.cn/4608550e916e405c9a12fab98be05bcb.png)
如果以后遇到不知道程序从哪开始的话，可以查看左边那一栏的函数，或者windows特别查找导入
![在这里插入图片描述](https://img-blog.csdnimg.cn/3452f6e55e82423bab721ed4e4f8b121.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b3edf8ba37094f5d9d169a076f454747.png)
可以看到，这个二进制文件导入了这些函数，我们可以获得很多信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/109fd16f9b864594881383126d8208c8.png)

isdebuggerpresnt这个函数是遇到错误时执行的，说明我们在此程序里需要规避一些东西
![在这里插入图片描述](https://img-blog.csdnimg.cn/c3e7d160f2104f4e8272f544863ee020.png)
这个程序还有打开，读取和写入文件的功能，获得这些信息对之后的逆向很有用，这些的外部参照，交叉引用很可能在实现一些很重要的东西
ctrl+f12，查找一下程序中存在的字符串
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac770a82683a46799d72e0120bdf3a0b.png)
看起来并没有什么有用的信息，在这种情况下，我们必须手动指定字符串的类型，打开ida的选项，选择字符串
![在这里插入图片描述](https://img-blog.csdnimg.cn/b15e6a2bf53349869a64286f60232b79.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9b39102d02842598ea0ac5b54b3b03b.png)
在linux二进制文件中看到一个字符串通常只是字节，然后以空值结尾
```
"ABCD\x00"
```
但是在windows上，它通常是16位，因此字符串始终基于字符，空字符，字符，空字符，字符，空字符这样的
```
"A\x00B\x00C\x00D\x00......."
```
如果在ida里ctrl+f12里是默认寻找linux的c字符串，所以在做windowsPE文件时，通常找不到它们
但是在ida里我们可以手动选择查找字符串的方式，这种情况下，c 16bits可以查看字符串
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9976aa659c74188a295635c2dc5ad01.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/728d756313384b679101834588a32c10.png)
现在找到了有意义的字符串，在这里，我们可以看到这个程序要求我们输入密码，双击这个字符串，去到调用的地方
![在这里插入图片描述](https://img-blog.csdnimg.cn/774266bddc8a4f869d82a215c0f61eac.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/ec92a59c93e94e2bb06b5df05896888e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ecca2b13eb7442d88029e444d3f999a3.png)
看下空格，进入可视化界面
![在这里插入图片描述](https://img-blog.csdnimg.cn/49b27ba25bf2424dbcceae43d49d31e5.png)
可以看到，这个地方就是它加载的地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/6b02ace7ac18473e9e986d9373448d1a.png)
然后调用了这个sub，这可能是printf，然后是格式化字符串%s
![在这里插入图片描述](https://img-blog.csdnimg.cn/7d229818e7c44cb7b72a80ca89326ae0.png)
我们跟踪一下这个sub，我们就会看到scanf
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2bf932d68444fcd8701aecb70150b71.png)
所以这只是围绕scanf实现的一些函数包装，我们重命名一下这些调用函数的名字，之后看起来会一目了然
![在这里插入图片描述](https://img-blog.csdnimg.cn/1d0acca0450042b9876d3e0ee025ef32.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/629894f7dc7f494fb2a5277457c3471b.png)
这个程序在这里读取了我们输入的字符后，然后做了一个if判断
![在这里插入图片描述](https://img-blog.csdnimg.cn/ec6ba411801446c09ca8ec0fb622f3df.png)
所以最后一个调用的函数是用来检查我们输入的字符串的
![在这里插入图片描述](https://img-blog.csdnimg.cn/42c54bc2dce047bfa2a71132fc0e0aef.png)
我们将它重命名为check_password
![在这里插入图片描述](https://img-blog.csdnimg.cn/ad85becde00f4abca098f4705d5407bc.png)
我们双击进入这个函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/83b1f7308e554919828ca55f453d9237.png)
发现在这个函数的开头，加载了一个字符串"IronManSucks"，在下面还有一些循环，并且比较了一些东西，所以这很有可能是密码
我们执行这个程序，然后输入IronManSucks
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d5a7538f1124e48a70df1dbe9083f4e.png)
但是并没有什么其他的东西显示出来，但我们还需要考虑一件事，那就是为什么有40多个程序，它们是如何归属在一起的，它们是相同的还是不同的？我随意复制了两个程序到linux上做对比
![在这里插入图片描述](https://img-blog.csdnimg.cn/04e37d0801e94e90855a48de8dc11305.png)
我们使用hexdump进行比较
```
cat 程序 | hexdump -C > a.out
cat 程序2 | hexdump -C > b.out
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/dfa78b3fa5294be5bc35b31a2edbec18.png)
然后使用vimdiff比较它们
```
vimdiff a.out b.out
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/be9b66c0751b4e37ae9ec28cb6a390b8.png)
我们可以看到，这两个文件非常不一样，我们去ida里看一下源代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/bcdfde9a477145e2ba8a5552650eff94.png)
emm.....看得头疼，我打算用ghidra来看一下源代码
关于安装和使用ghidar的方法可以去看我之前的文章，进入ghidar后反汇编main函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/92c7b8cd25784811b4667212cd204382.png)
看起来舒服多了
![在这里插入图片描述](https://img-blog.csdnimg.cn/8de1a039b1f6466ca34ebca43c5cb900.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/54bece32934540099045d2994832e992.png)
这个程序最后的一句话是叫我们去踩...砖？这个地方是检查密码的地方
![在这里插入图片描述](https://img-blog.csdnimg.cn/ddb0cca22dde40ddadc4de465bf48b19.png)
我以为密码是IronManSucks，但是是不对的，所以我们重新分析一下检查密码的函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/f8da8d0fd7e5425bb5b786b11218e347.png)
看得头疼，我们回到ida，进入可视化界面
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7af4fe1e36c4638867782bd93b342f1.png)
可以看到，在经过比较后，它成功打印出了Oh，hello Batban字符串，但是另一个判断里有更多的代码，我将两个程序检查密码的函数反汇编对比了一下，发现都是差不多的
![在这里插入图片描述](https://img-blog.csdnimg.cn/b18ec15d552f4f40b787dc912fb1671e.png)
字符串比较的值是当前未映射的地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/bd8d890a734a44d1820fb502454804ac.png)
这是程序在执行时来自动态内存的值，所以还有一个密码，当我们查看此地址其他位置的交叉引用时，我们会跳转到此位置
![在这里插入图片描述](https://img-blog.csdnimg.cn/fe9b0638f2174aba85bdad46c611da06.png)
memcpy，说明有一些字符串从某个地方复制到这个地址，并且复制的字符串在eax寄存器中，双击调用的函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/a6a7bbb8fe894c4388cb86b413a7a3b6.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/8babdee2532c457e86147b2e75a9f894.png)
按下f5查看源代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/52507f045b2441f69e9712dd61154657.png)
可以看到，它执行了FindResourceW后又执行了LoadResource，这个函数在可执行文件中查找资源并加载它，他加载了这些数据，然后复制到该区域
![在这里插入图片描述](https://img-blog.csdnimg.cn/2e52c2641a12404eaeaec4b5c646a781.png)
然后我们使用x32dbg来调试这个程序，直接运行到叫我们输入密码的位置
![在这里插入图片描述](https://img-blog.csdnimg.cn/967b03c70a7143f48db440bcb0bfd69e.png)
然后我在字符串周围的堆栈找到这个返回地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/a0c38bf092eb481cbfda9bb2f92766a3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/16ddbaf068ac47a9b4b80da3c167b9c1.png)
然后跳转到此地址进行分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/7af59f49cb654b82af63f298f2c057f3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5748621921f444fbaa102456846216d6.png)
右击跟踪此函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/b982104b62fc421cb967747141c5bcd8.png)
进入此函数后，在下面找到一些有趣的东西
![在这里插入图片描述](https://img-blog.csdnimg.cn/727a761ce6ba48d58627df78dff90965.png)
这个密码是被引用的，所以看起来像另一个密码，我们在此处设置一个断点，然后输入密码
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed0b718891f94dfd8a6ec9223d6042db.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b11f1a024dc149a483da93956a9cef3c.png)
按F8一步一步慢慢调试
![在这里插入图片描述](https://img-blog.csdnimg.cn/704f6ea376504138b15a73a8a58cbd46.png)
可以看到，这里一直在比较字符串的值，然后继续往下
![在这里插入图片描述](https://img-blog.csdnimg.cn/ce8b4cba927642a485f5c1d6fb528d82.png)
然后输出了正确的答案，他输入了一个图片，然后说是m
在之前分析程序函数的时候，这个程序有读取和写入文件的功能
![在这里插入图片描述](https://img-blog.csdnimg.cn/0355458f2ff14f0cb6b02b52a1c4c251.png)
我们打开图片看一下
![在这里插入图片描述](https://img-blog.csdnimg.cn/970c1c8aa82f4468a033e4b427a32c1f.png)
这个图片编号为35，现在我们只需要将字母对应的编号排列起来即可
但是文件太多了，我们不能全部都一对一调试，不然浪费的时间太多了，于是我又返回逆向分析
然后发现他加载的资源非常大，但是我们的密码很短
![在这里插入图片描述](https://img-blog.csdnimg.cn/195e55beb79747ffb1f970edabc8abdd.png)
然后我在data段里到处找，最终找到了一个其他部分引用了该区域内的数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/b5d6ed154db745998f1008a9eecaaa17.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c3f1b13ac0a46d492530e0c0c3274f1.png)
然后发现了这个xor函数，这里有一个循环和一个常数值的异或操作，在下面还能看到被压入的堆栈常量和异或的地址，这里其实是两个不同东西的异或，它们都指向BRICK加载内容，然后根据它们的地址和BRICL的起始地址，我们可以计算它们的在BRICK数据中的偏移量
![在这里插入图片描述](https://img-blog.csdnimg.cn/5fe02d8d13004483b914be1b42a77dcd.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d32823913cc84cc6b7faafaad0ea1338.png)
emm.....总结一下：
```
现在我们知道了有一个隐藏的密码是从一个名为BRICK的嵌入式资源加载的
然后执行程序可以使用这个隐藏的密码获得图像
我们还有两个隐藏的xor加密的字符串
```
我们需要写一个python程序来读取在BRICK的资源，我们需要一个可以处理PE文件的模块
然后我在github上找到了这个
```
https://github.com/erocarrera/pefile
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/34944784f9a34aaf941b470c1183574d.png)
然后我在linux机子上安装了这个模块
![在这里插入图片描述](https://img-blog.csdnimg.cn/e19dbe592db14ee3aa6a77d4cd6ab72c.png)
然后试一下能否处理PE文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/7696cd01e837468380ff794664ec32b8.png)
从第一个条目里所有的目录中，通过遍历终于找到了一些数据，我们从二进制文件内部得到的偏移量+大小，而且大小还和在ida里看到的一样
![在这里插入图片描述](https://img-blog.csdnimg.cn/940f6e7abe954484b1bea23146f738d5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/86f5ca2758ea486ebc76d030b5569438.png)
使用CFF Explorer可以确认密码位于BRICK:id_101资源数据的开头
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a1fa56be7534b5db4a55dae433054a6.png)
使用LIEF从 PE 资源部分提取密码
```
https://github.com/lief-project/LIEF
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/95efaf21b3c24f68962ab098846886a4.png)
由于我python太差，我在这里贴上其他大佬的解题脚本
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import lief

def get_code(filename):
    binary = lief.parse(filename)
    brick = binary.resources.childs.next()
    id_101 = brick.childs.next()
    data = id_101.childs.next().content

    code = ""
    has_zero = False
    for d in data:
        if d == 0:
            if has_zero == True:
                break
            else:
                has_zero = True
        else:
            has_zero = False
            code += chr(d)
    print("{} => {}".format(filename, code))

for _, _, files in os.walk("."):
    for f in files:
        if f.endswith(".exe"):
            get_code(f)
```
运行脚本后根据得到的密码来获得图片
```
from subprocess import Popen, PIPE, STDOUT
import os

passcode = dict()
passcode["1BpnGjHOT7h5vvZsV4vISSb60Xj3pX5G.exe"] = "ZImIT7DyCMOeF6"
passcode["1JpPaUMynR9GflWbxfYvZviqiCB59RcI.exe"] = "PylRCpDK"
passcode["2AljFfLleprkThTHuVvg63I7OgjG2LQT.exe"] = "UvCG4jaaIc4315"
passcode["3Jh0ELkck1MuRvzr8PLIpBNUGlspmGnu.exe"] = "uVmH96JGdPkEBfd"
passcode["4ihY3RWK4WYqI4XOXLtAH6XV5lkoIdgv.exe"] = "3nEiXqMnXG"
passcode["7mCysSKfiHJ4WqH2T8ERLE33Wrbp6Mqe.exe"] = "Q9WdIAGjUKdNxr6"
passcode["AEVYfSTJwubrlJKgxV8RAl0AdZJ5vhhy.exe"] = "UkuAJxmt8"
passcode["aSfSVMn7B8eRtxgJgwPP5Y5HiDEidvKg.exe"] = "b1VRfMTNPu"
passcode["azcyERV8HUbXmqPTEq5JFt7Ax1W5K4wl.exe"] = "qNb6tr7n"
passcode["BG3IDbHOUt9yHumPceLTVbObBHFneYEu.exe"] = "KSL8EAnlIZin1gG"
passcode["Bl0Iv5lT6wkpVCuy7jtcva7qka8WtLYY.exe"] = "uLKEIRAEn"
passcode["bmYBZTBJlaFNbbwpiOiiQVdzimx8QVTI.exe"] = "7kcuVMWeIBFGWfJ"
passcode["Bp7836noYu71VAWc27sUdfaGwieALfc2.exe"] = "NcMkqwelbRu"
passcode["cWvFLbliUfJl7KFDUYF1ABBFYFb6FJMz.exe"] = "yu7hNshnpM4Vy"
passcode["d4NlRo5umkvWhZ2FmEG32rXBNeSSLt2Q.exe"] = "5xj9HmHyhF"
passcode["dnAciAGVdlovQFSJmNiPOdHjkM3Ji18o.exe"] = "ZYNGeumv6QuI7"
passcode["dT4Xze8paLOG7srCdGLsbLE1s6m3EsfX.exe"] = "dRnTVwZPjf0U"
passcode["E36RGTbCE4LDtyLi97l9lSFoR7xVMKGN.exe"] = "dPVLAQ8LwmhH"
passcode["eEJhUoNbuc40kLHRo8GB7bwFPkuhgaVN.exe"] = "J1kj42jZsC9"
passcode["eovBHrlDb809jf08yaAcSzcX4T37F1NI.exe"] = "rXZE7pDx3"
passcode["Ew93SSPDbgiQYo4E4035A16MJUxXegDW.exe"] = "eoneTNuryZ3eF"
passcode["gFZw7lPUlbOXBvHRc31HJI5PKwy745Wv.exe"] = "jZAorSlICuQa0g8"
passcode["hajfdokqjogmoWfpyp4w0feoeyhs1QLo.exe"] = "hqpNm7VJL"
passcode["HDHugJBqTJqKKVtqi3sfR4BTq6P5XLZY.exe"] = "45psrewIRS"
passcode["iJO15JsCa1bV5anXnZ9dTC9iWbEDmdtf.exe"] = "2LUmPSYdxDcil"
passcode["IXITujCLucnD4P3YrXOud5gC7Bwcw6mr.exe"] = "aGUwVeVZ2c19mgE"
passcode["JIdE7SESzC1aS58Wwe5j3i6XbpkCa3S6.exe"] = "goTZP4go"
passcode["jJHgJjbyeWTTyQqISuJMpEGgE1aFs5ZB.exe"] = "9aIZjTerf0"
passcode["JXADoHafRHDyHmcTUjEBOvqq95spU7sj.exe"] = "jZRmFmeIchneGS"
passcode["K7HjR3Hf10SGG7rgke9WrRfxqhaGixS0.exe"] = "Z8VCO7XbKUk"
passcode["kGQY35HJ7gvXzDJLWe8mabs3oKpwCo6L.exe"] = "14bm9pHvbufOA"
passcode["lk0SOpnVIzTcC1Dcou9R7prKAC3laX0k.exe"] = "9eDMpbMSEeZ"
passcode["MrA1JmEDfPhnTi5MNMhqVS8aaTKdxbMe.exe"] = "auDB6HtMv"
passcode["NaobGsJ2w6qqblcIsj4QYNIBQhg3gmTR.exe"] = "C446Zdun"
passcode["P2PxxSJpnquBQ3xCvLoYj4pD3iyQcaKj.exe"] = "nLSGJ2BdwC"
passcode["PvlqINbYjAY1E4WFfc2N6rZ2nKVhNZTP.exe"] = "0d7qdvEhYGc"
passcode["SDIADRKhATsagJ3K8WwaNcQ52708TyRo.exe"] = "5O2godXTZePdWZd"
passcode["SeDdxvPJFHCr7uoQMjwmdRBAYEelHBZB.exe"] = "ohj5W6Goli"
passcode["u3PL12jk5jCZKiVm0omvh46yK7NDfZLT.exe"] = "4z0gAyKdk"
passcode["u8mbI3GZ8WtwruEiFkIl0UKxJS917407.exe"] = "r6ZWNWeFadW"
passcode["v6RkHsLya4wTAh71C65hMXBsTc1ZhGZT.exe"] = "dEDDxJaxc1R"
passcode["w3Y5YeglxqIWstp1PLbFoHvrQ9rN3F3x.exe"] = "HQG0By9q"
passcode["wmkeAU8MdYrC9tEUMHH2tRMgaGdiFnga.exe"] = "0rhvT5GX"
passcode["x4neMBrqkYIQxDuXpwJNQZOlfyfA0eXs.exe"] = "Fs3Ogu6W3qk59kZ"
passcode["xatgydl5cadiWFY4EXMRuoQr22ZIRC1Y.exe"] = "8V9AzigUcb2J"
passcode["xyjJcvGAgswB7Yno5e9qLF4i13L1iGoT.exe"] = "gNbeYAjn"
passcode["y77GmQGdwVL7Fc9mMdiLJMgFQ8rgeSrl.exe"] = "8Etmc0DAF8Qv"
passcode["zRx3bsMfOwG8IaayOeS8rHSSpiRfc9IB.exe"] = "XgkvZJKe"

def run_binary(filename):
    code = passcode[filename]
    print("{} => {}".format(filename, code))
    p = Popen([filename,], stdout=PIPE, stdin=PIPE, stderr=STDOUT)
    output = p.communicate(input=code)[0]
    print("output: {}".format(output))

for _, _, files in os.walk("."):
    for f in files:
        if f.endswith(".exe"):
            run_binary(f)
```
最后就能拿到flag
# 总结
这个是国外前几年的一个ctf比赛的题，看起来还不错，学到很多逆向的知识，还是收获颇多的，现在在这里记录一下做题的笔记












