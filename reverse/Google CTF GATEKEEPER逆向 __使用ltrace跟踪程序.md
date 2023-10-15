# 简介
需要题目和逆向工具的都可以加我qq：3316735898，有什么不懂的也可以来问我
# 开始
直接运行程序后，它会提醒你输入用户名和密码

![在这里插入图片描述](https://img-blog.csdnimg.cn/77a88aeda8964ecfb94b633dd8cea858.png)

我们随意输入几个字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/334654daf69048cfb8480df3ded131c9.png)

程序提示用户名输入错误
这里我不打算用常规的方法来逆向这个程序，这题我打算使用ltrace工具来跟踪程序的运行
# ltrace
这个工具是用来跟踪进程调用库函数的情况
```
ltrace ./gatekeeper baimao passwd
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0e71cc9d94b45158dd6cc9108f3b097.png)
出现了一大堆无用的代码，我们需要筛选一下有用的函数
```
ltrace ./gatekeeper baimao passwd 2>&1 | grep "strcmp"
```
我们通过重定向到标准错误流到标准输出，然后通过grep筛选出strcmp函数
这个函数会将我们输入的值与一个程序里的值做对比，最后输出的是布尔值，如果对比的值一样，则输出0，如果不一样，则输出1

![在这里插入图片描述](https://img-blog.csdnimg.cn/4cb62de51c41481498728754ecc3f898.png)
可以看到，这里一个字符串和我们输入的值做了对比，这应该就是用户名了，我们输入看看
```
ltrace ./gatekeeper 0n3_W4rM passwd 2>&1 | grep "strcmp"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/84ef9ff93bc34759b409b28944683723.png)
这里可以看到，我们用户名输入正确后，密码也进行了对比，但是我们输入的密码在这里是倒着的，所有被对比的值也是倒着的，我们从后往前输入就好了
```
 ./gatekeeper 0n3_W4rM I_g0T_m4d_sk1lLz
```
我们输入正确的账号和密码后就能显示出flag
![在这里插入图片描述](https://img-blog.csdnimg.cn/580ad3a316e645ac88da714df575ec8f.png)

```
CTF{I_g0T_m4d_sk1lLz}
```
# 查看伪代码
用ghidra打开程序，然后进入main函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/438735b543be46498180af233a1244c7.png)

我们来看看程序的伪代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/7459713ddcde4e6ca68e690b11c653f4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d933bb729534e68b0638eff9bc3ebf5.png)

分析伪代码：
```
  text_animation(
                "/===========================================================================\\\n|                Gatekeeper - Access your PC from everywhere!                |\n+======= ====================================================================+\n"
                );  //花里胡哨的欢迎图，无用
  if (param_1 == 3) {   //检查我们是否输入了程序需要的两个字符串，还有一个是程序本身，如果没有，则退出，显示提示
    text_animation(" ~> Verifying.");   //输出字符串~> Verifying
    verify_animation(3);  //我跟进进函数看了，只是花里胡哨的加载字符，无用
    iVar1 = strcmp((char *)param_2[1],"0n3_W4rM");  //将我们输入的第二个值，也就是用户名与字符串做对比，如果正确则进入循环
    if (iVar1 == 0) {   
      sVar3 = strlen((char *)param_2[2]);
      local_18 = (char *)malloc(sVar3 + 1);
      strcpy(local_18,(char *)param_2[2]);
      local_10 = 0;
      //对我们输入的用户名进行一大堆移动操作
      while( true ) {   //进入while循环
        sVar3 = strlen(local_18);
        if (sVar3 >> 1 <= local_10) break;
        local_19 = local_18[local_10];
        sVar3 = strlen(local_18);
        local_18[local_10] = local_18[(sVar3 - local_10) + -1];
        sVar3 = strlen(local_18);
        local_18[(sVar3 - local_10) + -1] = local_19;
        local_10 = local_10 + 1;
      }  //又对我们输入的字符做一大堆叠加操作
      verify_animation(3);    //我跟进进函数看了，只是花里胡哨的加载字符，无用
      iVar1 = strcmp(local_18,"zLl1ks_d4m_T0g_I");   //将我们的最后一个值，也就是密码与字符串做对比，正确则进入if循环
      if (iVar1 == 0) {
        text_animation("Correct!\n");   //输出字符Correct!
        text_animation("Welcome back!\n");  //输出字符Welcome back!
        snprintf(local_a8,0x80,"CTF{%s}\n",param_2[2]);    //输出flag，也就是我们输入的密码
        text_animation(local_a8);
      }
      else {    //否则输出密码错误
        text_animation("ACCESS DENIED\n");
        text_animation(" ~> Incorrect password\n");
      }
      uVar2 = 0;
    }
    else {  //否则输出用户名错误
      putchar(10);
      text_animation("ACCESS DENIED\n");
      text_animation(" ~> Incorrect username\n");
      uVar2 = 1;
    }
  }
  else {   //如果我们没有输入值就运行程序，就会提醒你
    puts("[ERROR] Login information missing");
    printf("Usage: %s <username> <password>\n",*param_2);
    uVar2 = 1;
  }
  return uVar2;
}

```
由此可见，flag也是我们输入的密码
这里记录一下使用ltrace跟踪程序来逆向的笔记
