![在这里插入图片描述](https://img-blog.csdnimg.cn/8f29146095ae4f0287b213a751760d52.png)
题目地址：
```
https://app.hackthebox.com/challenges/exatlon
```
# 开始
下载程序后，拖入die分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/ad1ad838581d4398816a77aa27abd75a.png)

发现用upx加了壳，我们脱壳即可
```
upx -d exatlon_v1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f90552f2d9748b39e243d719bdb941d.png)

然后用ghidra打开分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/3bfa359b21a349b39d64c2675790e88d.png)

在函数列表里找到主函数，通过伪代码窗口可以发现一些奇怪的数值

![在这里插入图片描述](https://img-blog.csdnimg.cn/5f86bd94a4be4317aff9a923769a9a8a.png)

程序在获取了我们输入后，与这些数值做了对比，可以猜测每个字母都对应着程序里的一个数

![在这里插入图片描述](https://img-blog.csdnimg.cn/14846be2f80c45ada014868b5efb30ef.png)

我们运行程序看看是什么样子的

![在这里插入图片描述](https://img-blog.csdnimg.cn/d08298fdd6ba49cdbcb8e0a466b94842.png)

在主函数上面也看到了这些输出信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/eeaa92d4af5c49808c2556276067af31.png)

接下来用gdb动态分析，看看我们的猜想对不对

```
gdb exatlon_v1
```

首先在ghidra汇编界面找到和我们输入做对比的地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/4f2a404d82334018a1b6c33aff8233b5.png)

然后去gdb里下一个断点

```
b *0x404d2d
run
```

然后我们输入这个平台flag的首字母
```
HTB{
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d5deef37359b4616a5b7e9fac549bdfa.png)

可以看到我们输入的字符都变成了数值

![在这里插入图片描述](https://img-blog.csdnimg.cn/e07eaeacdb4247c58df0e37259e43226.png)

和程序里对比的数值头部一致

![在这里插入图片描述](https://img-blog.csdnimg.cn/615dc6dec6184a83a6dd06ba48cca09a.png)

接下来我们可以一个一个输入键盘上的字母，来获取对应的数值

```
1968:{ 2000:} 1520:_ 2016:~ 1536:` 528:! 1024:@ 560:# 576:$ 592:% 1504:^ 608:& 672:* 640:( 656:) 720:- 688:+ 976:= 1456:[ 1488:] 1984:| 1472:\ 704:, 736:. 752:/ 1008:? 960:< 992:> 928:: 944:; 784:1 800:2 816:3 832:4 848:5 864:6 880:7 896:8 912:9 768:10 1552:a 1568:b 1584:c 1600:d 1616:e 1632:f 1648:g 1664:h 1680:i 1696:j 1712:k 1728:l 1744:m 1760:n 1776:o 1792:p 1808:q 1824:r 1840:s 1856:t 1872:u 1888:v 1904:w 1920:x 1936:y 1952:z 1040:A 1056:B 1072:C 1088:D 1104:E 1120:F 1136:G 1152:H 1168:I 1184:J 1200:K 1216:L 1232:M 1248:N 1264:O 1280:P 1296:Q 1312:R 1328:S 1344:T 1360:U 1376:V 1392:W 1408:X 1424:Y 1440:Z
```

然后和程序里的值对比，即可得到flag
```
HTB{l3g1c3l_sh1ft_l3ft_1nsr3ct1on!!}
```
