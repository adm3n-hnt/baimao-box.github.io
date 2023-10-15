2022picoCTF题目地址
```
https://play.picoctf.org/practice?category=3&originalEvent=70&page=1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/03fafdb678184a8d8b71492001f24c37.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
这是一道300分的题，非常简单，我们下载二进制文件，然后发现无法动态调试，我们先静态分析试试，将文件拖入IDA PRO
![在这里插入图片描述](https://img-blog.csdnimg.cn/fb84f8e0da934c53844a4777590b1b22.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
按下空格进入汇编代码审计页面，发现一大堆的运算操作，看起来有点麻烦，然后再按下f5，ida会尽量帮我们还原源代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/356e7b8c8482404a89416fe1d5df646e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
一目了然，答案已经出来了
# 源代码分析
```
__int64 __usercall main@<rax>(char **a1@<rsi>, char **a2@<rdx>, __int64 a3@<rbp>)
{
  __int64 result; // rax
  unsigned __int64 v4; // rdi
  unsigned __int64 v5; // rt1
  __int64 v6; // [rsp-48h] [rbp-48h]
  signed int v7; // [rsp-44h] [rbp-44h]
  __int64 v8; // [rsp-40h] [rbp-40h]
  signed __int64 v9; // [rsp-38h] [rbp-38h]
  signed __int64 v10; // [rsp-30h] [rbp-30h]
  signed __int64 v11; // [rsp-28h] [rbp-28h]
  signed __int64 v12; // [rsp-20h] [rbp-20h]
  unsigned __int64 v13; // [rsp-10h] [rbp-10h]
  __int64 v14; // [rsp-8h] [rbp-8h]

  __asm { endbr64 }                    
  v14 = a3;
  v13 = __readfsqword(0x28u);
  v9 = 5509350891791333953LL;
  v10 = 3486412172599641652LL;
  v11 = 7522471968904265011LL;
  v12 = 22063033952132964LL;
  sub_1120("What's my favorite number? ");                #输出字符What's my favorite number?
  v7 = 863305;                                            #赋予v7变量863305字符串
  sub_1140("%d", &v6);                                    #接受我们输入数据，变量名为v6
  v7 = 863305;
  if ( (_DWORD)v6 == 549255 )                             #进入if循环，将v6与549255字符串做对比
  {                                                       #如果v6等于549255这个字符串，则进入以下操作
    v7 = 863305;
    v8 = sub_1249((__int64)&v14, (__int64)&v9);
    sub_1130(v8, stdout);
    sub_10E0(10LL);
    sub_10D0();
  }
  else                                                    #如果v6不等于549255这个字符串，则进入以下操作
  {
    sub_10F0("Sorry, that's not it!");                    #输出字符Sorry, that's not it!，很明显我们输入错误
  }
  result = 0LL;
  v5 = __readfsqword(0x28u);
  v4 = v5 ^ v13;
  if ( v5 != v13 )
    result = sub_1110(v4);
  return result;
}
```

根据我们的分析，只需要运行程序，然后输入549255即可
# flag
![在这里插入图片描述](https://img-blog.csdnimg.cn/108c4995d0c446c8836430541594f2ca.png)

