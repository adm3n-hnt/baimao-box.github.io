![在这里插入图片描述](https://img-blog.csdnimg.cn/f9bd103f9c434ee1a66dde68d076060b.png)

# 什么是环境变量
关于什么是环境变量，我这篇文章介绍的很清楚
```
https://blog.csdn.net/qq_45894840/article/details/128622314?spm=1001.2014.3001.5502
```
这里在扩展一点
## env
env是英文单词environment的缩写，其功能是用于显示和定义环境变量，我们可以通过查看env来获取本机全部的环境变量配置
```
ls env:\
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a6aac69a4baf44519ddede01ebda88c3.png)




在这里可以看到很多的环境变量配置，如果是cmd那就执行以下命令查看全部的环境变量

```
set
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b33e69215d524ce4b5ccaaca4da5ca24.png)

# 绕过Windows Defender和隐藏混淆行为
我们还可以通过env来查看指定的环境变量
```
echo $env:SYSTEMROOT
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/145c5cd99f5f4ff98c29a631346c4492.png)

然后我将T更换为?

![在这里插入图片描述](https://img-blog.csdnimg.cn/a866c51caa7147fba5f78e2683e26e4c.png)


还是能正常显示环境变量，因为env会根据这个环境变量表来查找，类似于find -name "baimao*.exe"，这个命令会查找当前目录所以包含baimao这几个字符的exe文件


我们还可以进一步来用?替代字符


![在这里插入图片描述](https://img-blog.csdnimg.cn/638517de233143498a7526ecc1b5fd05.png)


再多就会因为解析多个环境变量而报错

```
echo $env:S?????????
```
在后面加上其他的文件夹名，输出的内容也会改变

![在这里插入图片描述](https://img-blog.csdnimg.cn/ca7a1ea6b9c04a4f868e1daeca3fe512.png)




我们还可以使用dir或者ls或者Get-ChildItem来查看这个文件夹
```
Get-ChildItem $env:S?????????\System32
ls $env:S?????????\System32
dir $env:S?????????\System32
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d29e08a885da4a269d215e32675b95d0.png)


在后面指定的文件夹，也可以用?来代替


```
Get-ChildItem $env:S?????????\Sys?????
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a6a5e38fff434897b29f7e90e9752611.png)


为了缩小范围，我们最好使用*来代替字符，关于*的用法，上面已经介绍过了

```
Get-ChildItem $env:S?????????\S*2
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/fc87b25a62fb42c0ab93244bf7cb7f4c.png)


现在我们要弹出计算器，可以使用以下命令
```
start $env:S?????????\S*2\c*lc.*
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e9f173fdf1b74ec3ac6eb1734ef07ce6.png)


如果我们要调用schtasks.exe，可以用以下命令
```
start $env:???t??r???\*2\??h???k?*
```
这些方法是我在看一篇apt恶意软件分析的报告学到的

```
https://www.securonix.com/blog/detecting-steepmaverick-new-covert-attack-campaign-targeting-military-contractors/
```
攻击者通过这种方法来隐藏混淆行为和绕过 Windows Defender


