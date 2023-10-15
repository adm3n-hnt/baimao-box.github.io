# 前言
一次练习Android逆向的记录，写得很详细，有什么没有理解的地方可以私信

csdn不让我加外链，所以将链接前面的#号去掉即可

题目：
```
ht#tps://github.com/tlamb96/kgb_messenger
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c08a5bf4f9404d57ae8309ddb1db4501.png)

在这个挑战中，一共有三个flag，翻译为中文是

![在这里插入图片描述](https://img-blog.csdnimg.cn/e3a91cba340941708f96aa20e874986a.png)

# 使用到的工具
```
adb：apt install adb
安卓模拟器：ht#tps://www.yeshen.com/
Apktool：apt install apktool
JD-GUI：ht#tps://github.com/java-decompiler/jd-gui/releases/
dex2jar：apt install d2j-dex2jar
uber-apk-signer：ht#tps://github.com/patrickfav/uber-apk-signer
```

# 安装程序
我们使用安卓模拟器来安装这个apk程序，直接将apk包拖入模拟器即可安装

![在这里插入图片描述](https://img-blog.csdnimg.cn/4ceadb37c84a41bd80f5ec5a445d9e73.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/99fd29d396744fed9df0ab3cf0ef997e.png)

但是打开这个程序就会报错，提示为这个app只能在俄罗斯设备上运行


# apk文件结构介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/47261c6996d8424ba625e1d7f668a97b.png)


AndroidManifest.xml：
```
https://developer.android.com/guide/topics/manifest/manifest-intro
```
```
1.应用程序包的名称
2.应用程序的所有组件
3.此应用程序运行需要什么权限，以及其他应用程序访问此应用程序的信息所需的权限
4.兼容性功能
```
res：
```
包含资源但具有开发人员无法更改的预定义文件夹层次结构的文件夹。这些文件用于为不同的屏幕大小、操作系统版本和多语言支持提供替代方案。
```
META-INF：
```
这是一个包含验证信息的文件夹。这是在“签署”应用程序时生成的。这个文件夹APK中包含的每个文件的指纹信息。这意味着对 APK 的任何修改（甚至替换图标）都需要重新签署 APK，否则操作系统将拒绝安装
```
classes.dex：
```
Google 的 Java VM 版本的专有格式，它包含所有编译成称为Dalvik的特定字节码的 Java/Kotlin 代码
```
resources.arsc：
```
包含将代码 (classes.dex) 链接到资源 (res) 的信息的文件。例如，代码可能引用对话框的文本，而资源包含所有语言的文本。然后，Android 操作系统会根据设备的区域设置选择正确的语言。
```
# Flag1
## 静态分析
使用Apktool解码APK并将其输出到kgb文件夹中
```
apktool d kgb-messenger.apk -o kgb
```
首先查看程序的AndroidManifest.xml文件，看看程序启动时调用了什么

![在这里插入图片描述](https://img-blog.csdnimg.cn/d46be315cc3b4fa3b3c84c7f2ced2d2d.png)

启动时调用的是MainActivity函数

我们用dex2jar将apk转换为.jar 格式，因为这样我们之后就可以使用JD-GUI来查看程序的Java代码
```
d2j-dex2jar.sh kgb-messenger.apk -o kgb.jar
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9eece1036c954697913de7d5ae9a85c3.png)

然后用JD-GUI分析这个jar包

![在这里插入图片描述](https://img-blog.csdnimg.cn/59dadb3170274e9d86f21251f5d185af.png)

双击打开程序，按下ctrl+o，选择jar包


![在这里插入图片描述](https://img-blog.csdnimg.cn/ea46ca8cff3c46fcb83f241f40e4ff93.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/11a4986ff44e41a5bd4d0e2b9c4a9edc.png)

找到MainActivity.class，在下面可以看到程序启动时报错的字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/be5bf6842272457aa2554480f2db3a3e.png)

```
if (str1 == null || str1.isEmpty() || !str1.equals("Russia")) {
      a("Integrity Error", "This app can only run on Russian devices.");
      return;
} 
```
首先程序会判断str1的值是否等于Russia，不等于的话就会报错
```
if (str2 == null || str2.isEmpty() || !str2.equals(getResources().getString(2131558400))) {
      a("Integrity Error", "Must be on the user whitelist.");
      return;
}
```
然后会判断str2的值是否等于2131558400，我们需要将这个值转换为十六进制值，才能在xml文件中找到对应的位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/5f47392fd74a4c0eb52d6f98728e6471.png)

然后在/res/values文件夹里查找0x7f0d0000的位置
```
grep -inr --color 0x7f0d0000 *.xml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f7298236f5524e5c8b6c7e95e1fb8f04.png)

可以看到name的值，接着我们搜索User

![在这里插入图片描述](https://img-blog.csdnimg.cn/84f48abf7ef049ac80d3999c15e3a975.png)

在这里可以看到一个base64编码，我们解密看看

```
echo "RkxBR3s1N0VSTDFOR180UkNIM1J9Cg==" | base64 -d
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c82083df86f460a997524bb1c891a5e.png)

成功找到第一个flag
```
FLAG{57ERL1NG_4RCH3R}
```
# Flag2
![在这里插入图片描述](https://img-blog.csdnimg.cn/9b2aceee9a904decb8fe4d15677e7f43.png)

我们需要想办法让这两个if判断成功，才能进入下一步login挑战

我们可以修改程序，将这两个if判断去掉，然后构建一个新的apk，之前我们用apktool解码过这个apk包，选择我们直接搜索即可

找到MainActivity文件
```
find /home/kali/apk/kgb -name "MainActivity*"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7497eea23a5649228bcbd5bc7c9a6824.png)

打开第一个文件，定位到0x7f0d0000的位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/b380322f9b3e446d9eb7a262b81b3b9b.png)

我们需要把第115行到175行的内容都删除，然后将186行的内容更改为return-void



![在这里插入图片描述](https://img-blog.csdnimg.cn/ddbe0530894446e28a246e34a3f9e42f.png)



保存文件

## 构建apk
然后使用Apktool构建一个新的apk
```
apktool b kgb -o new_kgb.apk
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f1f11cdbf85047068977ed2428b13f41.png)

## 给apk文件签名
构建完apk文件后，不能直接安装，我们需要先给apk文件签名
```
java -jar uber-apk-signer-1.2.1.jar -a new_kgb.apk
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d12335904fb48c7b7340bdd5b51e954.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/7f4632a3342342f7b84f670ff5b1901d.png)

然后将这个新生成的apk直接拖到安卓模拟器内并打开

![在这里插入图片描述](https://img-blog.csdnimg.cn/76aa829552a54de5a5f81f09a31209f8.png)
## 破解账户与密码

现在就不会报错了，回到JD-GUI，单击LoginActivity

![在这里插入图片描述](https://img-blog.csdnimg.cn/29ce177bce2042e68f7e22bfe738f0d2.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/895fc3b3713248d8a9961753604a8903.png)



在这里可以看到程序登录验证的代码，和我们找第一个flag的步骤相似，我们需要将2131558450转换为16进制，来查找用户名

```
grep -inr --color 0x7f0d0032 *.xml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/70aff794b5f146848b4dec4610d5014e.png)

name的值为username，现在查找username字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/2b46908750524065b5a055a478b4c43f.png)

用户名为：codenameduchess

现在我们需要知道密码是什么，回到JD-GUI

![在这里插入图片描述](https://img-blog.csdnimg.cn/b591017cb7ea4f63a63ee908081c0b1d.png)

双击这个函数，跳转到密码验证的地方

![在这里插入图片描述](https://img-blog.csdnimg.cn/5988b55a004a4280951d09d5518f11d8.png)

我们输入的密码和2131558446进行了比较，将其转换为十六进制后继续寻找

![在这里插入图片描述](https://img-blog.csdnimg.cn/2d7fb4c18b3f400ea4b7d352b4dacd36.png)

name的值为password，继续寻找

![在这里插入图片描述](https://img-blog.csdnimg.cn/2553426257474180b4b034a2088e7f4b.png)

在这里有一串md5值，但是无法爆破出来，根据题目简介

![在这里插入图片描述](https://img-blog.csdnimg.cn/c14f6a0cc9de413c92734be11aa20ebc.png)

this is a "recon" challenge，难道意思是需要社会工程吗

用Google搜索这个用户名

![在这里插入图片描述](https://img-blog.csdnimg.cn/ff1ff24924a94c28b65a8803aa5a3da3.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/5401195b339044ff93a957f6e3e8ad48.png)


一个动画人物，现在直接搜索密码试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/f5ebbbc2f3a14da596917acef7a7334e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4e1a63c3ecf94c71bc69ed7d932032bf.png)

password为：Guest，题目说字符均为小写，所以是：guest

现在登录程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/bd1747da4b194169bc9c56148f871f2b.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/00d49c46f5254e1b9e475827839ca6b1.png)

登录成功，获得flag
```
flag{G00G1_PRO}
```
我对guest字符串进行了加密
![在这里插入图片描述](https://img-blog.csdnimg.cn/129d63283dca4dcd8829bb2368a18c95.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/8eb85f218f88481b879dba831a3bae76.png)

发现是这个md5值不完整，开头少了一个0，不然是能被爆破出来的
# Flag3
回到JD-GUI，单击MessageActivity

![在这里插入图片描述](https://img-blog.csdnimg.cn/1ade368d8a524509b11a8a5c86749982.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c98b418a8a94567a38016a7f9075513.png)

在这下面可以看到获取最后的flag代码，每当我们发送消息时，都会调用onSendMessage函数，

```
EditText editText = (EditText)findViewById(2131165225);
String str = editText.getText().toString();
```
我们输入的文本会转换成String str
```
if (a(str.toString()).equals(this.p)) {
        Log.d("MessengerActivity", "Successfully asked Boris for the password.");
        this.q = str.toString();
        this.o.add(new a(2131558434, "Only if you ask nicely", j(), true));
        this.n.c();
      } 
```
然后str传参到a函数里，并且在这里调用了equals函数来检查它是否等于p

![在这里插入图片描述](https://img-blog.csdnimg.cn/0bdd6633e2e14ff1996bc504062a8ce1.png)

在上面可以看到p的值，下面可以看到a的函数代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/9c91838458d44c4cb4257823e5b1a269.png)

这个a函数对我们输入的字符串进行一些异或操作，我们需要写一个脚本来还原字符串
```
p = "V@]EAASB\022WZF\022e,a$7(&am2(3.\003"
p = list(str(p))

for i in range(len(p) // 2):
	p[i] = chr(ord(p[i]) ^ 0x32)
	p[len(p) // 2 + 1 + i] = chr(ord(p[len(p) // 2 + 1 + i]) ^ 0x41)

p.reverse()
print("".join(p))
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/323f7a2d687b43d2a5a54a5dea8b4631.png)

运行脚本，获得字符串，回到模拟器，输入字符串并发送

![在这里插入图片描述](https://img-blog.csdnimg.cn/3d00c80a1a074e2dbb8221aebf639874.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/17985def3e014022a005b5a8ab31f73f.png)

r的值为

![在这里插入图片描述](https://img-blog.csdnimg.cn/1405e0a497aa4ed087cb534ee7d87816.png)

b函数进行的操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/a4c3910b83b44c19b0375a2cc0bb4998.png)

我们继续写一个脚本来还原字符串

```
import string

r = "\000dslp}oQ\000 dks$|M\000h +AYQg\000P*!M$gQ\000"
r = list(str(r))
r.reverse()

for i in range(len(r)):
	if i % 8 == 0:
		print("_", end="")
		continue 
	for ch in string.printable:
		final_ch = chr((ord(ch) >> (i % 8)) ^ ord(ch))
		if final_ch == r[i]:
			print(ch, end="")
print("")
```
运行脚本，获得字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/3ea8d38c69534bc196f8af0a7625d8e2.png)

还原一下，字符串为
```
May I *PLEASE* have the password
```
回到模拟器，发送消息
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f0b9d8129f24a228a7e87b9c50dd5e3.png)

获得最后的flag
```
FLAG{p455w0rd_P134SE}
```
