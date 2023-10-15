# 题目信息
这是一道非常简单的题目

![在这里插入图片描述](https://img-blog.csdnimg.cn/28c9be9fc196400fac674c7b7eefe692.png)


意思是在构建docker时，有些东西泄露了，需要我们在里面找到flag

# 开始

我们下载附件，然后解压压缩包
```
tar -xvf dockREleakage.tar.gz 
```
```
x：提取
v：显示所有过程
f：指定文件
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f27cadd644aa433198a660784c5a917a.png)

这里有一些json文件，是docker构建时的核心文件，我们打开看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/29d30ea9d5df4565be6d9069fc55fcda.png)

都是一行，读起来很不方便，我们格式化一下json
在线网站：
```
https://www.bejson.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/47a685211f484993bec69049bbff365b.png)

这些都是docker的基本信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/6a841c1c4601429bb81b684d612f919c.png)

现在读起来方便一些了，在下面可以看到一些执行过的操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/0ec92f7fa8104362b373a66be9d0bbbd.png)

在第59行，有一个被base64加密过的密文，我们解密看看

```
echo ZmxhZ3tuM3Yzcl9sMzR2M181M241MTcxdjNfMW5mMHJtNDcxMG5fdW5wcjA= | base64 -d
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7b394dc4538a4a67a6473674bd78b081.png)

只得到一半的flag，这个文件已经没有其他有用的信息了，我们进入第一个文件夹看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/52bd16a4e39646b8933f43f7c54a23b1.png)

我们解压压缩包
```
tar -xvf layer.tar
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/de4e08fdb41d4edca991b4dfbd551b26.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0e8dc3d1bf194d099f015d7e2d1441dd.png)

这个是docker完整的文件系统，我找了一会，并没有flag的信息，我们进入第二个文件夹解压压缩包

![在这里插入图片描述](https://img-blog.csdnimg.cn/d042166cd052402cbd74f78d04c0c55f.png)

第二个文件夹也没有flag文件，然后进入第三个文件夹

![在这里插入图片描述](https://img-blog.csdnimg.cn/901b31ad12234c579f3dad8a32e9af54.png)

在这里找到了flag文件，我们打开看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/a94ce6baed0c4ae5a2252c7092afe2d4.png)

成功得到后半部分flag，我们将flag拼接起来即可得到完整的flag

flag：
```
flag{n3v3r_l34v3_53n5171v3_1nf0rm4710n_unpr073c73d_w17h1n_7h3_d0ck3rf1l3}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d5a38ad00e074e388fc80394d8be4484.png)

