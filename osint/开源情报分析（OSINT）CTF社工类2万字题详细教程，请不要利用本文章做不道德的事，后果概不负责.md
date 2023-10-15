# 简介
现在国内外最新的ctf比赛都有这个项目了，列如给你一个照片找地址或者人名，给你一个名字找他的社交账号什么的，考验选手的信息收集与社工能力，这篇文章对这类题型做一个基础的总结，以后遇到这种题型就知道该怎么做了
本文章会教你们关于查找有关个人信息以及其他危险的技术，请不要做不道德的事，在任何时候都不应该将此技术用于未授权的事
# 开源情报分析简介
在企业和社交媒体上都有公开信息，这些是对我们开源的，所以叫做开源情报分析
然后是情报搜集步骤
![在这里插入图片描述](https://img-blog.csdnimg.cn/6cb48a2338fb4ee28ae1b122badd1f3c.png)
情报搜集步骤由五个部分组成
```
规划和指导
搜集
处理和利用
分析报告和产品
传播与整合
```
假如你获得公司授权后开始准备收集此公司的信息，就需要开始计划以下的事项
```
我们需要针对什么
我们的目标是什么
我们什么时候去收集
```
这些都要在规划和指导的阶段完成，一旦确定了目标，并且已经完成了规划，就可以进入下一个收集阶段了
此文章大部分都是关于如何收集信息的，列如隐藏自己的信息，反向图像搜索，图像EXIF信息，物理位置，识别地理位置，电子邮件地址，寻找用户名和帐户，寻找人，电话号码，生日，破解账户密码，简历，社交媒体账号信息收集等，再提醒一次，请不要做不道德的事
# 马甲
你可以把这个认为是你的第二个虚假身份，比如虚假账户，有社交媒体账户，邮箱地址，看起来和普通账户一样，ip伪装等，拥有一个好的马甲可以使自己在调查的时候不会暴露真实身份，我们不能让目标知道我们正在寻找他的信息，在进行在线调查时，你可以做无数的事情来保持匿名
```
仅用于调查的专用计算机
加密电子邮件 – 使用 Proton Mail
在线平台的电话号码
目标最活跃的社交媒体资料
几个不同的虚拟机
博客或网站（您可以使用 WordPress、Blogger 或 Medium 等免费博客）
一个VPN
```
### 专用电脑
拥有一台专用计算机是绝对必须的。您不希望您在头像下所做的任何事情以某种方式与您的个人真实帐户相关联。这不仅会显示您的马甲确实是马甲，而且可能会将您的真实身份与它联系起来。这台计算机不必很贵，您可以使用像 Raspberry Pi 或便宜的笔记本电脑，使用我将在下面讨论的其他工具，您的专用计算机应该无法链接到网络上的另一台计算机。
### 加密电子邮件

这通常是开源情报分析（OSINT） 和信息安全社区的最佳实践。尽管由于 Gmail 提供了大量免费工具和无缝集成，但不要这样做。谷歌正在跟踪您。即使您提供虚假信息，他们最终仍然会知道是您。  Proton Mail是加密电子邮件行业的知名品牌。还有其他选择，但如果您以前没有尝试过它们，我会选择 Proton Mail。用户界面易于理解，不需要任何高级设置。
 Proton Mail
 ```
 https://protonmail.com/
 ```
### 电话号码

如果可以，请尝试获得一个非常便宜的专用于您头像的电话计划。在线在线接收短信验证码和谷歌语音是一个不错的选择。请记住，许多此类网站在注册时都会要求您提供主要电话号码 (Google)。如果您非常关心隐私，请找一个不关心隐私的人。
### VPN
在线进行开源情报分析（OSINT） 研究时，掩盖您的 IP 很重要。最好的方法是使用 VPN，这个比较敏感，我无法多说些什么
### 社交媒体资料
现在您有了专用的计算机、加密的电子邮件、电话号码和 VPN，我们可以进入有趣的部分。您可以使用您的所有信息（电子邮件、电话号码）来创建您选择的社交媒体资料。由于您是从头开始，因此以有机的方式开始互动很重要。这可能包括关注人、发布链接、进行状态更新、与目标相同的人进行互动等，这个过程将需要很长时间，我建议使用多个电子邮件和电话号码创建多个头像，以降低风险并以不同方式部署它们
### 虚拟机
虚拟机是创建额外隐私层的好方法。您还可以将它们用于 OSINT 调查中的特定工具。我建议从Buscador开始，因为它提供了各种各样的 OSINT 工具。您还可以尝试使用 Windows VM 来访问FOCA等工具和其他 Windows 特定工具。试用 Android 模拟器以利用移动应用程序。  Nox是一款出色的模拟器，可帮助您入门
Buscador
```
https://inteltechniques.com/buscador/
```
FOCA
```
https://www.elevenpaths.com/labstools/foca/index.html
```
Nox
```
https://www.bignox.com/
```
### 博客 
如果您想深入创造马甲，请在 WordPress、Medium 或 Blogger 上创建一个免费博客，并将其链接到您的社交媒体资料。在社交和您的博客上生成内容以提高可信度。经过一段时间的发展，您将拥有一个可信且有价值的复杂角色
### Chrome 扩展程序
在网络上保持匿名的部分原因是阻止所有形式的跟踪。我推荐的两个扩展是AdBlock和Disconnect Me。这些将阻止广告跟踪您以及来自社交媒体网站的所有拉取请求。结合 VPN，您应该拥有安全搜索所需的一切。
AdBlock
```
https://chrome.google.com/webstore/detail/adblock-—-best-ad-blocker/gighmmpiobklfepjocnamgkkbiglidom?hl=en-US
```
Disconnect Me
```
https://chrome.google.com/webstore/category/extensions?hl=en
```
### 其他资料
袜子的艺术，一个关于OSINT的详细介绍
```
https://www.secjuice.com/the-art-of-the-sock-osint-humint/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e262b1307d141578f76aebae2e2d4d8.png)

我设置匿名 Sock Puppet 帐户的流程（reddit），一个制作马甲的详细介绍
```
https://www.reddit.com/r/OSINT/comments/dp70jr/my_process_for_setting_up_anonymous_sockpuppet/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/85eee7e88da14782822334a18b8539e7.png)

假名生成器，一个最近生成假身份的网站
```
https://www.fakenamegenerator.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d6294f9350264de282215332d9f13c27.png)

此人不存在，AI随机生成一个不存在的人的网站
```
https://www.thispersondoesnotexist.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4896a03d83194d0ebeb5be3f441bf62b.png)

虚拟支付卡，一个创建虚拟支付卡而不是使用您的常规借记卡或信用卡程序
```
https://privacy.com/join/LADFC
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2980d43592d4edd8e187ffc6f2c6815.png)

关于制作一个马甲我就不演示了，大家可以根据我上面谈到的东西做一个属于自己的马甲
# 搜索引擎运营商
### google
不用多说，大家都知道google是什么
```
www.google.com
```
现在我在google搜索我的名字，出现了很多信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/de042263963a4243a72759bbbb5df679.png)
google高级搜索
```
https://www.google.com/advanced_search
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f6924d60e22248539b4710221f36849f.png)
### duckduckgo
duckduckgo更像是一个基于隐私的搜索引擎
```
www.duckduckgo.com
```
我现在搜索我的名字，可以看到，有关我的名字的信息没有Google的多
![在这里插入图片描述](https://img-blog.csdnimg.cn/276170ec05104b3abb828ee9d973e807.png)
DuckDuckGo 搜索指南 
```
https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b6b256d91cde4f1f89a878e098030814.png)
### bing
```
https://www.bing.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a04deec7665f41ee8e3cb6ab35ffd61d.png)
搜索指南
```
https://www.bruceclay.com/blog/bing-google-advanced-search-operators/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/71d9a016b7154686b573bf8c4100e3b3.png)
### Yandex 
一个俄罗斯的搜索引擎
```
https://yandex.com/
```
### 百度
```
www.baidu.com
```
合理的使用搜索引擎以及语法，可以获得很多需要的信息，在写文章的时候，用了这个技术刷了很多洞

# 反向图片搜索
通过图片，查找网上关于这张图片留下的痕迹
### google图片搜索
```
https://images.google.com
```
### Yandex
```
https://yandex.com
```
### 天眼
```
https://tineye.com
```
# 查看exif数据
exif是可交换图像文件格式，是专门为数码相机的照片设定的，可以记录数码照片的属性信息和拍摄数据。
exif数据可以提供很多信息，当你拍照时，可能会留下一些数据，比如手机系统信息，经纬度信息，拍摄的时间等
### 在线网站exif图片信息查询
```
exif.tuchong.com
```
```
https://exifdata.com/
```
我随意拍摄了一张图片，然后查询exif数据，可以看到我们获得了很多敏感信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/0844282988224a899f1e440368892f60.png)
拍摄的器材，时间，以及地理位置的经纬度
![在这里插入图片描述](https://img-blog.csdnimg.cn/7549e0c9f891455ab0df85785649a238.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/b121417d659147dfa119eae3de65a935.png)
# 物理位置分析
我们将要通过一张图片，来找到图片拍摄的位置
### google地图
```
https://www.google.com/maps/
```
点击左下方的卫星卫星，我们可以看到现实里的情况
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ca055dc89e041ebbc4b96d9661ad937.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2af2636ae5b4bb8823549ce6b8ad21d.png)
我们搜索大熊猫基地看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/176f9b8cbc1b4ee5b32d00f90cdb1420.png)
我们能获取很多物理位置的信息，比如大楼，马路，树林，停车场等，哪里可能人多，哪里人少，哪里可能有警卫，需要避免的地方，能为之后现场分析或者社会工程学做铺垫
我们可以将右下角的小人拖到我们想要查看的地方，进行街景查看
![在这里插入图片描述](https://img-blog.csdnimg.cn/f0aeee9f20c64122a40201fb7c630427.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a324446690c4484f8199e8b814a8f48d.png)

我们可以点击箭头来进行前进，现在我们进入园区看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/eb4e94bbb97749e9a5321895bcadde09.png)
我们可以在街景分析里找到很多有用的信息，列如摄像头，可以进入的门，读卡器，人们正在做什么等来找到一个适合的目标位置，可以为之后的社会工程学做铺垫
# 识别地理位置
假设你有一张图片，但是不知道图片在哪里拍摄的，没有exif数据，没有有用的信息，我们如何找到图片拍摄的位置呢
![在这里插入图片描述](https://img-blog.csdnimg.cn/f96c7a0376bf4a86804d9242ab6ceb05.png)

这是一张图片，难度为简单，假设我们要调查这张图片，我们能从中找到什么信息呢
其实这张图里有很多信息
### 汽车
```
这是一辆凯迪拉克汽车，我们可以寻找出售凯迪拉克汽车的地方来缩小寻找的范围
我们还可以看到，这辆凯迪拉克停在路的右侧，方向盘在车辆的左侧，所以可以确定这个国家的汽车方向盘是左边的
我们还可以看这个车牌，可以说明这辆车所在的国家
```
### 背景
```
我们可以看见周围有很多雪，说明这张照片不是在南半球或者赤道附近拍的
我们可以根据十字架来判断这是一个教堂
还有一座桥，路灯，路牌，河流
```
现在我们从这张照片获取了很多信息，首先我们进行google图片搜索
![在这里插入图片描述](https://img-blog.csdnimg.cn/f5ab6ff8148c4f5a8cda95de27d35ee3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4286c36ee06349db8afece8bc4c22202.png)
这可能是在瑞典哥德堡境内拍摄的图像，根据我们前面的环境分析，来进一步确定
然后进入Google地图，我们的关键词为
```
瑞典哥德堡 教堂
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f2751072e3a34c00869ce972e5aa0d87.png)

这个地方很符号我们的判断
```
教堂，桥，分岔口，路牌，河流
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3738c02480654306b2a75ae049d9bf6d.png)

我们把小人拖到这里看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/cd9047adb3204723990098d377e94a34.png)
成功通过图片找到地址

这里有一个网站，他提供图片，然后玩家根据图片找到拍摄地址，可以得到分数之类的，有兴趣的朋友可以来挑战一下
```
https://www.geoguessr.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/671ca69b0672425daa5afb8bdcee5bc3.png)
强烈推荐大家看看这个博客，关于通过照片寻找拍摄地址，这篇博客是写得最详细的
```
https://somerandomstuff1.wordpress.com/2019/02/08/geoguessr-the-top-tips-tricks-and-techniques
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9f96250fceb4106b8a43de6b605d9ff.png)
# 发现电子邮件地址
获取目标的电子邮件地址，可以发送各种木马来进一步达成我们的目标，也可以用电子邮件地址来获取目标的社交媒体账号，以及一些网站的账号
### Hunter.io
```
https://hunter.io/
```
在这个网站输入一家公司的名字，就可以获取到公司使用的电子邮件地址
我们输入bilibili看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e74ccfe34bc437baaec075dbef17ab4.png)
我们找到了5个电子邮件地址，右边是电子邮件地址的来源
### Phonebook.cz
```
https://phonebook.cz/
```
这个网站能获取公司很多员工的电子邮件地址，也是很厉害的一个网站，我们输入bilibili看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ee691030f1940dea7be7b7d38e4581f.png)
找到了80多条电子邮件地址，我们还可以利用这些来做凭证，之后会讲到
### VoilaNorbert
```
https://www.voilanorbert.com/
```
这个网站可以找到任何人的电子邮件地址，不过我没怎么用过，我经常用的是下面这个软件
![在这里插入图片描述](https://img-blog.csdnimg.cn/5e4c71865b3f4681a2a4c40f83bbef28.png)
### Clearbit Connect
这个工具是google邮箱插件，下载地址
```
https://chrome.google.com/webstore/detail/clearbit-connect-supercha/pmnhcgfcafcnkbengdcanjablaabjplo/related?hl=zh-CN
```
这个插件可以查找任何公司的员工电子邮件地址
下载完成后我们登录一下
![在这里插入图片描述](https://img-blog.csdnimg.cn/63b1728a2b8a42bdbdbb0fb47c67a60e.png)
我们可以在这里查找
![在这里插入图片描述](https://img-blog.csdnimg.cn/a403b206768448188add58b606fb4d5b.png)
输入bilibili后，我们找到了一些用户
![在这里插入图片描述](https://img-blog.csdnimg.cn/4396960a4c10497ebb1191bf7cf97a0d.png)
### Email Hippo
```
https://tools.verifyemailaddress.io/
```
这个网站可以检查电子邮件地址是否存在
![在这里插入图片描述](https://img-blog.csdnimg.cn/363d8dccaf9548a39ec5162e21d1df4e.png)
就不多介绍了
### Email Checker
```
https://email-checker.net/validate
```
这个网站和上面那个网站一样，是检查电子邮件地址是否存在的简单工具
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8852f53a42043828834b2edaafe04c8.png)
就不多介绍了
### github电子邮件发现
工具地址
```
https://github.com/paulirish/github-email
```
clone下来后进入目录，运行工具
```
./github-email.sh 用户名
```
这里我输入我的github用户名
![在这里插入图片描述](https://img-blog.csdnimg.cn/f21c17a51c144c6fad4b458ea732e4ea.png)
这个工具用不了的话可以加我qq，在文章底部，它要先在本地变量里获取github的令牌，工具下面有详细的教程，如果不会，我可以详细教师傅使用
# 密码情报分析
这是最重要的一个技术，我们可以通过各种方式找到目标的密码，下面是详细的教学
### Dehashed
```
https://dehashed.com/
```
一个非常强大的搜索网站
![在这里插入图片描述](https://img-blog.csdnimg.cn/f0185fff0cfb4eacbfcbf781875020f7.png)

我们可以搜索ip，电子邮件，用户名，泄露的密码，地址，车牌号，域名，知识产权等，都是可以关联的
缺点是需要付费，这里我没钱，就无法演示了，之后找机会专门介绍一下这个网站吧

### WeLeakInfo
```
https://weleakinfo.to/v2/
```
这个网站和上一个网站差不多可以搜索ip，电子邮件，用户名，泄露的密码，可惜也要钱
![在这里插入图片描述](https://img-blog.csdnimg.cn/48719fa5c5fa4d01bcf1c6d7d0cfcd79.png)
### LeakCheck
```
https://leakcheck.io/
```
这个网站可以搜索电子邮件和用户名关联的一些东西，也要钱，只不过比上面网站便宜一点
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7574912a6d740a28afe1a4669be5e14.png)
### HaveIBeenPwned
```
https://haveibeenpwned.com/
```
最后这个网站是免费的，可以免费查找泄露的电子邮件地址和电话号
![在这里插入图片描述](https://img-blog.csdnimg.cn/f3dd2284f3d443bdac0f37fbaa1cd332.png)
测试
![在这里插入图片描述](https://img-blog.csdnimg.cn/5c3bcc351dcc4341af755d927c800cf7.png)
这些是检测信息泄露的网站，对情报分析工作有很大的帮助
# 寻找用户名和账号
我们可以用一些专门网站，来找到用户名等需要的信息
### NameChk
```
https://namechk.com/
```
这个网站可以查找用户名注册过的一些东西
进入网站搜索，我使用我的名字
![在这里插入图片描述](https://img-blog.csdnimg.cn/74fd6635a14a48cbb39727fe38b5bbc9.png)
可以看到，很多平台上都有我注册过的账号

### WhatsMyName
```
https://whatsmyname.app/
```
这个网站可以枚举多个网站的用户名，但是没有上面那个网站准确，不怎么推荐用
同样，这里搜索我的名字，可以看到，搜索到的结果没有上面那个网站多
![在这里插入图片描述](https://img-blog.csdnimg.cn/69c93f74ea1a4e55a7985dca2a3dbe92.png)
### NameCheckup
```
https://namecheckup.com/
```
这个网站也很好用，同样，这里搜索我的名字
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b412f0298b54ffd941ef38777de7d50.png)
# 找人
我们可以通过各种方式，来找到目标的身份信息，可以通过google关键词搜索来找到一些信息，这里我搜索我的信息
```
Ba1_Ma0 "qq"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/538755461fc9459f80ce0e62ee2a7a39.png)
可以看到，发现了我的qq号，通过各种关键词，来进一步对目标的身份信息搜集，这里我就不多说了
### WhitePages
```
https://www.whitepages.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/66296775239c43acbcc6fcd772c4fced.png)
这个网站可以免费查找手机号码，背景调查，犯罪记录，地址，亲戚，座机号码，年龄，交通记录，诈骗/欺诈评级，财务记录，业务详情，留置权记录，专业执照，娘家姓，物业详情，搜索统计，承运人信息等，缺点是只能输入英文
![在这里插入图片描述](https://img-blog.csdnimg.cn/a427207291b847f08718033c5ea92e51.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/a37d6ea303a64c68bcf7dc0702f78f6c.png)
各种用法大家可以自己去试试，这里就不过多演示了
### TruePeopleSearch
```
https://www.truepeoplesearch.com/
```
这个网站也是免费的，可以搜索姓名，电话和地址，缺点是只能输入英文
![在这里插入图片描述](https://img-blog.csdnimg.cn/2a8af50e5d4c4ac5ac747a26f2b7dab8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/79b0797984574cfeb66b20357f8b7262.png)
### FastPeopleSearch
```
https://www.fastpeoplesearch.com/
```
这个和上面那两个网站一样，都是免费的
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e70c93c86e0407195cd9e96a587a036.png)
### FastBackgroundCheck
```
https://www.fastbackgroundcheck.com/
```
和上面一样
![在这里插入图片描述](https://img-blog.csdnimg.cn/94a826e6fe724f489a779077db52e9b2.png)
### WebMii
```
https://webmii.com/
```
这个网站可以搜索名字在互联网上留下的痕迹
![在这里插入图片描述](https://img-blog.csdnimg.cn/7cd6e3d8eb2140b2baa28aa44d112bf0.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/7cca19385a3248349b67c80e64f96326.png)
### PeekYou
```
https://peekyou.com/
```
和上面那个网站一样，查找名字在互联网上留下的痕迹
![在这里插入图片描述](https://img-blog.csdnimg.cn/ce586d15205642189eb3281a826421e0.png)
### Spokeo
```
https://www.spokeo.com/
```
这个网站可以查找电子邮件关联的信息，我搜索了一下我的电子邮件地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab13863c7b154d469a6f713a4ad8bbd8.png)
点击家庭背景调查，有点哈人，缺点是需要给钱
![在这里插入图片描述](https://img-blog.csdnimg.cn/d844487400c74171bbbc2de12fc5cba4.png)
### That's Them
```
https://thatsthem.com/
```
一个免费的搜索网站，缺点也是只能输入英文
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a413082fd0f4921ba892a68dc12bdf7.png)

# 美国选民记录查询
### Voter Records
```
https://voterrecords.com/
```
美国免费查找选民记录的网站
![在这里插入图片描述](https://img-blog.csdnimg.cn/37d14043ab074d00a9d68f5cdcea2632.png)
# 电话搜索
上面一些网站也能搜索电话信息，这里介绍几个专门查找电话信息的网站，当你拿到一个电话号码时，不知道这个号码是哪个地区的，可以先使用google来缩小范围，我搜索我的电话号
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2d3ff90d6114263816619c18bba4749.png)

### TrueCaller
```
https://www.truecaller.com/
```
一个电话号码查询网站，这里我查询我的电话号码
![在这里插入图片描述](https://img-blog.csdnimg.cn/73f4e8e0ab0c495bb13a07f05758059a.png)
### CallerID Test
```
https://calleridtest.com/
```
一个美国电话号码查询网站
![在这里插入图片描述](https://img-blog.csdnimg.cn/a5803ac27ec34025a0d3fda3ec6aff91.png)
### Infobel
```
https://infobel.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d16f0e70bedb40438cb1b40331ded082.png)
# 社交媒体情报分析
ctf社工类题目最喜欢考这个，下面我详细介绍一下各社交平台的搜索方法
## Twitter情报分析
假设题目给了一个姓名，我们除了用上面介绍过的那些网站，还可以在社交平台上直接搜索
![在这里插入图片描述](https://img-blog.csdnimg.cn/39031a9ae1504bd3aeb708ff235ab66f.png)
在这里，我们可以直接搜索人名，我搜索我自己的用户名
![在这里插入图片描述](https://img-blog.csdnimg.cn/67d5c81770844bbd8c140836244eee23.png)
还有一个Twitter高级搜索

Twitter Advanced Search
```
https://twitter.com/search-advanced
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6e4e63deff90458ebbc77578b0cce04b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c4b9fd9863c4559ba0d0bc05416aad2.png)
可以更详细的锁定目标账号
### Social Bearing
```
https://socialbearing.com/
```
一个强大的推特内容搜索平台，可以免费Twitter 分析和搜索推文、时间线和推特地图，按参与度、影响力、位置、情绪等查找、过滤和排序推文或人员
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e51030144a943b090d27d7f7f1e6dbe.png)

### Twitonomy
```
https://www.twitonomy.com/
```
和上面那个网站差不多
![在这里插入图片描述](https://img-blog.csdnimg.cn/e148f3e3764846bd8836618a9c96a0a8.png)
### Sleeping Time
```
http://sleepingtime.org/
```
这个网站可以更根据Twitter 用户的最后 1000 条推文，然后根据他在 Twitter 上最不活跃的时间确定大致的睡眠时间表
![在这里插入图片描述](https://img-blog.csdnimg.cn/d1d33c5f87454057bd53764804cfd341.png)
### Mentionmapp
```
https://mentionmapp.com/
```
这个网站可以根据大数据分析，来找到目标的社交范围
![在这里插入图片描述](https://img-blog.csdnimg.cn/58580647667948f7a1f5d791fa6b0645.png)
### Tweetbeaver
```
https://tweetbeaver.com/
```
可以对目标推特账号进行一些数据分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/079109d200ae4fb593645d3cb5d61981.png)
### Spoonbill.io
```
http://spoonbill.io/
```
这个网站可以查看目标账号之前的签名
![在这里插入图片描述](https://img-blog.csdnimg.cn/4662bc3ef14f4bd5a583544f8c2dc3f6.png)
### Tinfoleak
```
https://tinfoleak.com/
```
这个网站可以查找目标账号的一些敏感信息，如地理位置，身份信息等
![在这里插入图片描述](https://img-blog.csdnimg.cn/0968df52132745b490b8a002c74a13d7.png)
### TweetDeck
```
https://tweetdeck.com/
```
TweetDeck 让您在一个简单的界面中查看多个时间线，从而提供更方便的 Twitter 体验。它包括许多高级功能，可帮助您充分利用 Twitter：管理多个 Twitter 帐户、安排未来发布的推文、构建推文集合等
![在这里插入图片描述](https://img-blog.csdnimg.cn/427fef01b087406480602109007d83c5.png)
## Facebook情报分析
### Sowdust Github
```
https://sowdust.github.io/fb-search/
```
这个网站可以免费搜索Facebook目标用户的一些信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/774891f3d5604e87907bab1347af6424.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7f627d21407f42358032c87d65a08fbc.png)
### IntelligenceX Facebook Search
```
https://intelx.io/tools?tab=facebook
```
这个网站和上面那个差不多，都是对Facebook用户的一些信息收集
![在这里插入图片描述](https://img-blog.csdnimg.cn/02deb467a0d24633894f55623e092ab4.png)
## Instagram情报分析
### Code of a Ninja
```
https://tools.codeofaninja.com/find-instagram-user-id
```
这个网站可以查询Instagram的用户id，可惜需要付费
![在这里插入图片描述](https://img-blog.csdnimg.cn/5d6d984469e64bcea9330301565623a1.png)
### InstaDP
```
https://www.instadp.com/
```
这个网站可以下载Instagram的用户的资料
![在这里插入图片描述](https://img-blog.csdnimg.cn/0718d4ef7dd5482b944259b2fd819ae3.png)
### ImgInn
```
https://imginn.com/
```
这个网站可以匿名浏览 Instagram 快拍，直接搜索用户名即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed8e5a7f466740beb7f279223ed167d7.png)
## Snapchat情报分析
### Snapchat Maps
```
https://map.snapchat.com/
```
这个网站可以可以轻松在线查看来自世界各地的Snapchat
![在这里插入图片描述](https://img-blog.csdnimg.cn/83296e84fa9d4997aaacc134149c92f0.png)
# 总结
```
https://github.com/TCM-Course-Resources/Open-Source-Intellingence-Resources
```
在这里还有更多的情报分析网站总结
![在这里插入图片描述](https://img-blog.csdnimg.cn/b48f6a2ab471415da28bbfa990d2b10e.png)
本文章也是我记录学习开源情报分析的笔记，之后有机会实战也会详细的写成文章，欢迎大家来关注我，有什么问题可以加我qq：3316735898
 
